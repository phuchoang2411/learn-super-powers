# Deep Dive: Systematic-Debugging Skill

Nguồn: https://github.com/obra/superpowers/tree/main/skills/systematic-debugging

---

## Cấu trúc thư mục

```
skills/systematic-debugging/
├── SKILL.md                           ← quy trình 4 phase
├── root-cause-tracing.md              ← kỹ thuật trace call chain
├── defense-in-depth.md                ← validate nhiều layer
├── condition-based-waiting.md         ← fix flaky tests do timing
├── condition-based-waiting-example.ts
├── test-pressure-1/2/3.md             ← case studies dưới áp lực
└── find-polluter.sh                   ← script tìm test làm nhiễm test khác
```

Skill này có **nhiều tài liệu phụ nhất** trong toàn bộ superpowers — vì debugging có nhiều dạng thất bại.

---

## 4 phase bắt buộc

```
Phase 1: Root Cause Investigation
  → Đọc kỹ error message và stack trace
  → Reproduce lại được một cách nhất quán
  → Review recent changes (git log)
  → Thêm diagnostic logging tại ranh giới giữa các component
  → KHÔNG đề xuất fix ở phase này

Phase 2: Pattern Analysis
  → Tìm code tương tự đang chạy đúng
  → Đọc TỪNG DÒNG của reference implementation
  → Document mọi điểm khác biệt
  → Hiểu dependencies và assumptions

Phase 3: Hypothesis & Testing
  → Đặt một giả thuyết cụ thể về root cause
  → Test bằng thay đổi nhỏ nhất có thể
  → Một biến / một lần — không bundle nhiều fix
  → Bỏ giả thuyết nếu evidence mâu thuẫn

Phase 4: Implementation
  → Viết failing test trước (TDD vẫn áp dụng)
  → Implement một fix duy nhất vào đúng root cause
  → Verify không regression
  → NẾU đã thử 3+ lần mà vẫn fail → đặt câu hỏi về architecture
```

---

## 3 kỹ thuật phụ quan trọng

### Root-cause tracing — trace ngược call chain

```
Symptom (error xuất hiện ở đây)
  ↑ tìm immediate cause
  ↑ ai gọi hàm này?
  ↑ data invalid từ đâu đến?
  ↑ tìm original trigger ← fix ở đây, không phải ở symptom
```

Dùng `console.error()` trong test (không dùng logger — có thể không hiển thị).

### Defense-in-depth — validate nhiều layer

Khi fix bug, không chỉ fix một chỗ. Thêm validation tại mọi layer data đi qua:

- **Layer 1: Entry point** — invalid input bị chặn sớm nhất
- **Layer 2: Business logic** — data có ý nghĩa đúng không
- **Layer 3: Environment guard** — bảo vệ theo context (ví dụ: test env)
- **Layer 4: Debug instrumentation** — log để forensic sau này

Mục tiêu: *"make the bug structurally impossible"*, không phải chỉ patch một điểm.

### Condition-based waiting — fix flaky tests do timing

Thay `setTimeout` bằng polling thực sự:

```ts
// Không dùng:
await new Promise(r => setTimeout(r, 50))
expect(result).toBe(...)

// Dùng thay thế:
await waitFor(() => result !== undefined, { timeout: 1000, interval: 10 })
expect(result).toBe(...)
```

Kết quả thực tế từ tài liệu: pass rate từ 60% → 100%, execution time giảm 40%.

---

## Vấn đề thực sự: Workflow consistency khi bug xuất hiện giữa chừng

Tình huống hay gặp:

```
brainstorming → writing-plans → executing-plans
                                      ↓
                               test fail (bug)
                                      ↓
                              fix bug ad-hoc
                                      ↓
                         plan/spec không còn đúng
                                      ↓
                          phải sync docs thủ công
```

Cái "sai sai" này là có thật — không phải do làm sai, mà do thiếu một bước:
superpowers không nói rõ *phải làm gì với plan khi bug được tìm thấy*.

---

## Workflow đúng khi bug xuất hiện giữa chừng

```
executing-plans → test fail
                      ↓
              DỪNG executing-plans
                      ↓
         invoke systematic-debugging
                      ↓
              tìm root cause
                      ↓
        phân loại bug thuộc loại nào?
        ┌─────────────────┬──────────────────────┐
        │ Bug trong code  │ Gap trong plan/spec  │
        │ (implementation │ (assumption sai,     │
        │ sai so với plan)│ requirement thiếu)   │
        └────────┬────────┴──────────┬───────────┘
                 ↓                   ↓
         Fix code, thêm task   Quay lại writing-plans
         fix vào plan          → update plan → tiếp tục
         (KHÔNG fix im lặng)   (KHÔNG fix ad-hoc)
                 ↓                   ↓
         verify → commit       verify → commit
                      ↓
              tiếp tục executing-plans
```

**Nguyên tắc:** Plan là source of truth. Nếu reality lệch với plan → plan phải được update trước khi tiếp tục, không phải sau.

---

## Phân loại bug để biết update gì

| Loại bug | Ví dụ | Cần update |
|---|---|---|
| Implementation error | Code sai logic so với plan | Thêm task fix vào plan, không đổi spec |
| Missing assumption | Plan giả định X nhưng thực tế không phải X | Update cả plan lẫn spec |
| Scope gap | Requirement chưa cover edge case này | Update spec → re-run writing-plans cho phần đó |
| Architecture issue | 3+ fix attempts vẫn fail | Brainstorming lại phần đó |

---

## Quy tắc tránh "sync docs thủ công"

1. **Không fix bug ngoài plan** — mọi fix đều là task trong plan
2. **Commit plan update trước khi commit fix** — plan và code luôn đồng bộ
3. **Nếu fix thay đổi architecture** → update spec trước, plan sau, code cuối

Format thêm task fix vào plan:

```markdown
## Task N+1: Fix — <mô tả root cause>
Discovered during Task N execution.
Root cause: <một câu>
File: <file cụ thể>

1. Test (đã có — test đang fail từ Task N)
2. Fix: <code cụ thể>
3. Verify: <lệnh chạy>
4. Commit: "fix: <mô tả>"
```
