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

---

## Xử lý small fixes sau khi plan đã execute xong

Tình huống: implement theo plan tương đối tốt, nhưng vẫn còn sai/thiếu một vài chỗ nhỏ.
Vấn đề không phải thiếu baseline — mà là làm sao fix mà không làm lệch plan/spec.

### Bước 1: Liệt kê hết trước, đừng fix ngay

Chạy full test suite, đọc hết output, ghi lại tất cả issue một lần:

```
Issues found:
- [ ] validateEmail() không reject email thiếu domain
- [ ] signup() trả về 500 thay vì 400 khi email trùng
- [ ] thiếu index trên bảng users.email
```

Lý do: nếu fix từng cái ngay lập tức, dễ fix cái này tạo ra cái khác và mất dấu.

### Bước 2: Phân loại từng issue

| Issue | Loại | Xử lý |
|---|---|---|
| Code sai so với plan | Implementation error | Thêm fix task vào plan hiện tại |
| Plan đúng nhưng spec thiếu edge case | Spec gap | Update spec → thêm task vào plan |
| Requirement mới phát sinh khi làm | Scope change | Brainstorming lại phần đó |

### Bước 3: Thêm fix tasks vào cuối plan

Không tạo plan mới. Mở `docs/superpowers/plans/<file-hiện-tại>.md`, thêm vào cuối:

```markdown
---
## Post-Execution Fixes

### Task N+1: Fix — validateEmail không reject thiếu domain
Type: implementation-error
Root cause: regex pattern thiếu TLD check

File: src/utils/validators.ts

1. Test (đang fail): `npm test -- --testPathPattern=validators` → confirm đỏ
2. Fix regex: `^[^\s@]+@[^\s@]+\.[^\s@]+$`
3. Verify: chạy lại → XANH
4. Chạy full suite → không có regression
5. Commit: "fix: validateEmail reject missing TLD"

### Task N+2: Fix — signup trả 500 thay vì 400 khi email trùng
Type: implementation-error
...
```

### Bước 4: Execute fix tasks theo TDD bình thường

Mỗi fix task là một vòng TDD nhỏ:
- Test đang đỏ → confirm đỏ → fix → xanh → commit

Sau mỗi fix task: chạy full test suite để đảm bảo không có regression mới.

**Kết quả:** Plan vẫn là source of truth. Mọi fix được ghi lại. Không có gì bị fix im lặng.
