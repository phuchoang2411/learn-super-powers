# Superpowers cho Brownfield Project không có TDD

## Vấn đề

Superpowers được thiết kế với giả định codebase đã có test coverage.
Với brownfield không có TDD từ đầu, khi plan execution gặp fail, không biết fail đó do:
- Code mới vừa viết sai?
- Code cũ đã sai từ trước?
- Interaction giữa code cũ và code mới?

→ Fix ad-hoc thì plan/spec bị lệch. Fix đúng thì không biết đúng theo baseline nào.

**Superpowers chỉ nói:** *"Existing code has no tests → You're improving it. Add tests for existing code."*
Nhưng không hướng dẫn cụ thể cách làm. Đây là gap cần lấp.

---

## Giải pháp: Characterization Tasks

Thêm một loại task đặc biệt vào plan trước khi execute — **Characterization Task**.

Characterization test không kiểm tra behavior đúng hay sai.
Nó chỉ đóng băng *"hệ thống hiện tại đang làm gì"* — dù đúng hay sai.
Đây là baseline. Khi có baseline mới biết change của bạn có break gì không.

---

## Workflow đầy đủ cho brownfield

```
brainstorming
    ↓
writing-plans
    ↓ (thêm characterization tasks vào đầu plan)
Characterization Phase
    ↓ (baseline xanh hết)
executing-plans (TDD bình thường)
    ↓
test fail?
    ├── fail ở characterization test → code mới break code cũ → systematic-debugging
    └── fail ở test mới → code mới sai → TDD cycle tiếp tục
```

---

## Characterization Phase — làm gì cụ thể

### Bước 1: Xác định scope

Trước khi viết bất kỳ test nào, liệt kê những file/function sẽ bị chạm vào trong plan.
Chỉ viết characterization test cho những thứ đó — không test toàn bộ codebase.

```markdown
## Characterization Scope
Files plan sẽ chạm vào:
- src/services/UserService.ts (hàm signup, validateEmail)
- src/utils/validators.ts (nếu có)
- src/models/User.ts
```

### Bước 2: Viết characterization tests

Với mỗi function sẽ bị chạm vào:

```ts
describe('CharacterizationTest: UserService.signup (existing behavior)', () => {
  it('hiện tại accept email rỗng', async () => {
    // Gọi function với input hiện tại
    const result = await userService.signup({ email: '', password: '123' })
    // Snapshot lại output — dù đúng hay sai
    expect(result).toMatchSnapshot()
    // hoặc nếu biết cụ thể:
    expect(result.error).toBeUndefined() // hệ thống hiện tại không validate
  })

  it('hiện tại return user object khi thành công', async () => {
    const result = await userService.signup({ email: 'a@b.com', password: '123' })
    expect(result.id).toBeDefined()
    expect(result.email).toBe('a@b.com')
  })
})
```

**Quy tắc viết characterization test:**
- Đặt prefix `CharacterizationTest:` trong describe để phân biệt
- Không assert behavior "đúng" — assert behavior "hiện tại"
- Nếu không chắc output, dùng `toMatchSnapshot()` lần đầu
- Chạy → xanh → commit → đây là baseline

### Bước 3: Baseline phải xanh trước khi execute plan

```
Chạy toàn bộ characterization tests → phải XANH HẾT
Nếu có test đỏ ở bước này → code cũ đã bị broken từ trước
→ ghi nhận lại, KHÔNG fix ngay, tiếp tục (đây là known issue)
```

---

## Thêm Characterization Tasks vào plan

Trong writing-plans, thêm các task này vào **đầu plan**, trước mọi task implementation:

```markdown
## Task 0: Characterization — UserService.signup
File: src/services/UserService.ts

Mục đích: đóng băng behavior hiện tại trước khi thay đổi.

1. Viết characterization tests cho signup(), validateEmail()
   (xem format trong brownfield-no-tdd-guide.md)
2. Chạy: `npm test -- --testPathPattern=UserService.char`
3. Tất cả phải XANH (nếu đỏ: ghi chú là known issue, không fix)
4. Commit: "test: characterization tests for UserService.signup"
```

---

## Khi test fail trong executing-plans

Với brownfield, khi gặp test fail, bước đầu tiên là **xác định loại test đang fail**:

```
Test fail
    ↓
Loại test nào?
    ├── Characterization test đỏ
    │       → code mới break behavior cũ
    │       → invoke systematic-debugging
    │       → tìm root cause trong code mới
    │
    └── Test mới (viết trong task hiện tại) đỏ
            ├── Đỏ vì feature chưa implement → bình thường, tiếp tục viết code
            └── Đỏ vì lý do khác → investigate trước khi tiếp tục
```

---

## Khi systematic-debugging gặp code cũ không có test

Phase 1 của systematic-debugging yêu cầu reproduce lại issue.
Với brownfield, thêm sub-step:

```
Phase 1 (brownfield version):
  → Đọc error message và stack trace
  → Reproduce lại được nhất quán
  → Kiểm tra: vấn đề này có TỒN TẠI TRƯỚC khi bạn chạm vào không?
      git stash → chạy test → nếu vẫn fail = pre-existing bug
      git stash pop → tiếp tục
  → Review git diff để biết chính xác mình đã thay đổi gì
  → Thêm diagnostic logging
```

Nếu là pre-existing bug:
- Ghi vào plan: `## Known Issue: <mô tả>` — không fix trong task hiện tại
- Tạo task riêng sau để fix nếu cần
- Tiếp tục task hiện tại

---

## Update plan sau khi fix bug — brownfield version

Khi bug được tìm ra và fix, cập nhật plan theo format:

```markdown
## Task N+1: Fix — <mô tả root cause>
Type: [implementation-error | pre-existing-bug | missing-assumption]
Discovered during: Task N
Root cause: <một câu>
Pre-existing: [yes/no] — nếu yes: tồn tại trước khi plan bắt đầu

File: <file cụ thể>
1. Characterization test (nếu chưa có cho code bị ảnh hưởng)
2. Failing test mô tả behavior đúng cần có
3. Fix: <code cụ thể>
4. Verify characterization tests vẫn xanh
5. Commit: "fix: <mô tả>"
```

---

## Checklist trước khi bắt đầu plan execution (brownfield)

- [ ] Đã xác định scope — file/function nào sẽ bị chạm vào
- [ ] Characterization tasks đã được thêm vào đầu plan
- [ ] Characterization tests đã chạy và xanh (hoặc known issues đã ghi nhận)
- [ ] Biết phân biệt characterization test vs test mới khi debug
- [ ] Có plan rõ ràng khi gặp pre-existing bug
