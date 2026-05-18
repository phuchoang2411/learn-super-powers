# Deep Dive: Writing-Plans Skill

Nguồn: https://github.com/obra/superpowers/tree/main/skills/writing-plans

---

## Cấu trúc thư mục

```
skills/writing-plans/
├── SKILL.md                         ← quy trình viết plan
└── plan-document-reviewer-prompt.md ← template subagent review plan
```

Tương tự brainstorming — có reviewer subagent riêng để tránh bias.

---

## Vị trí trong workflow

```
brainstorming → [spec approved] → writing-plans → [plan saved] → executing-plans
```

writing-plans nhận **spec đã được approve** từ brainstorming làm input, output là **plan file** cho executing-plans chạy. Không làm gì khác.

---

## Cấu trúc một plan file chuẩn

**Header bắt buộc:**
```markdown
# Feature: <tên>
Goal: <một câu mô tả kết quả>
Architecture: <2-3 câu giải thích cách hệ thống hoạt động>
Stack: <tech stack>
Execution: xem skills/executing-plans hoặc subagent-driven-development
```

**Mỗi task theo TDD cycle:**
```markdown
## Task N: <tên cụ thể>
File: src/path/to/file.ts

1. Viết test:
   ```ts
   // code test đầy đủ, không placeholder
   ```
2. Chạy: `npm test -- --testPathPattern=<file>` → expect FAIL
3. Implement:
   ```ts
   // code implement đầy đủ
   ```
4. Chạy lại → expect PASS
5. Commit: `git commit -m "feat: <mô tả>"`
```

---

## Rule quan trọng nhất: Zero placeholders

Đây là điểm phân biệt plan tốt vs plan vô dụng.

| Không chấp nhận | Phải là |
|---|---|
| `"Add appropriate validation"` | Code validation thực tế |
| `"Handle edge cases"` | List edge case cụ thể + code xử lý |
| `"Similar to Task 3"` | Lặp lại code đầy đủ |
| `"TBD"`, `"TODO"` | Nội dung thực tế |
| `"Implement the function"` | Function signature + body đầy đủ |

**Lý do:** Plan được giao cho subagent không có context của conversation. Nếu plan mơ hồ → subagent đoán mò → kết quả sai.

---

## Consistency xuyên suốt plan

Nếu Task 3 định nghĩa `clearLayers()`, Task 7 phải dùng đúng tên đó — không được tự ý đổi thành `resetLayers()` hay `removeLayers()`. Type signatures, tên biến, tên hàm phải nhất quán từ đầu đến cuối.

**Self-review 3 câu hỏi trước khi handoff:**
1. Mọi requirement trong spec đều có task tương ứng chưa?
2. Có placeholder hay vague language nào không?
3. Type signatures và tên có nhất quán xuyên suốt không?

---

## Plan reviewer — khác gì spec reviewer?

| | Spec reviewer (brainstorming) | Plan reviewer (writing-plans) |
|---|---|---|
| Input | Spec document | Plan document |
| Kiểm tra | Completeness, consistency, clarity, scope, YAGNI | Completeness, spec alignment, task decomposition, buildability |
| Câu hỏi cốt lõi | "Spec này đủ để planning không?" | "Engineer có thể follow plan này không cần hỏi thêm?" |
| Block nếu | Gap ảnh hưởng đến planning | Placeholder, missing requirement, vague task |

---

## Hai chế độ thực thi sau khi plan xong

Sau khi lưu plan vào `docs/superpowers/plans/`, offer user chọn:

**Option A — Subagent-driven (khuyến nghị):**
Mỗi task dispatch một agent mới, không có context cũ. Agent chỉ nhận task đó + plan. Sau mỗi task có review checkpoint.

**Option B — Inline execution:**
Agent hiện tại chạy từng task tuần tự, dừng ở checkpoint để confirm.

Subagent-driven tốt hơn vì: agent fresh không bị bias bởi context cũ, lỗi một task không ảnh hưởng task sau, dễ retry từng task độc lập.

---

## So sánh brainstorming vs writing-plans

| | brainstorming | writing-plans |
|---|---|---|
| Input | Yêu cầu từ user | Spec đã approved |
| Output | Spec document | Plan document |
| Đơn vị làm việc | Section của design | Task 2-5 phút |
| Reviewer | Spec reviewer subagent | Plan reviewer subagent |
| Lưu vào | `docs/superpowers/specs/` | `docs/superpowers/plans/` |
| Transition sang | writing-plans | executing-plans |
| Có hỏi user không | Có (từng câu) | Không — chỉ viết |
