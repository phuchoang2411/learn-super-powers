# Lộ trình học obra/superpowers

Repo gốc: https://github.com/obra/superpowers

## obra/superpowers là gì?

Một framework phương pháp phát triển phần mềm dành cho AI coding agent (Claude, Gemini, Copilot, Cursor...). Cung cấp bộ "skills" giúp agent làm việc có hệ thống thay vì code bừa.

---

## Quy trình 7 bước chính

1. **Brainstorming / Design** — Hỏi rõ mục tiêu trước khi code, trình bày thiết kế để con người duyệt
2. **Setup môi trường** — Tạo workspace cô lập bằng git worktrees
3. **Lập kế hoạch (Planning)** — Chia nhỏ thành các task 2-5 phút, rõ file path và cách verify
4. **Thực thi (Executing)** — Dùng subagent để làm từng task, có 2 vòng review
5. **Test-Driven Development** — Bắt buộc viết test trước rồi mới code (red-green-refactor)
6. **Code Review** — Review có hệ thống theo spec
7. **Finalize branch** — Dọn dẹp và hoàn thiện branch

---

## Thứ tự đọc tối ưu

```
1. README.md                                  ← bức tranh toàn cảnh
2. skills/using-superpowers/SKILL.md          ← "luật chơi" căn bản nhất
3. skills/brainstorming/SKILL.md              ← skill đầu tiên luôn được kích hoạt
4. skills/writing-plans/SKILL.md              ← tiếp theo sau brainstorming
5. skills/test-driven-development/SKILL.md   ← core của mọi implementation
6. skills/systematic-debugging/SKILL.md      ← khi có bug
7. Các skill còn lại theo nhu cầu thực tế
```

---

## 3 điều cần hiểu ngay từ đầu

**1. Skills có thứ tự ưu tiên cứng:**
> User instructions > Superpowers skills > Default system prompt

Nghĩa là skills này *ghi đè* hành vi mặc định của AI.

**2. Invoke skills TRƯỚC MỌI hành động** — kể cả trước khi hỏi clarifying question. Nếu thấy mình nghĩ:
- *"Cái này đơn giản, không cần skill"*
- *"Để tôi xem code trước đã"*

→ Đó là dấu hiệu đang tránh né skill, hãy dừng lại.

**3. Hai loại skill:**
- **Rigid** (TDD, debugging): làm đúng 100%, không tự ý thích nghi
- **Flexible** (patterns): áp dụng tinh thần, không cứng nhắc

---

## Danh sách skills

| Skill | Mô tả |
|---|---|
| `using-superpowers` | Luật dùng toàn bộ framework |
| `brainstorming` | Động não, thiết kế trước khi code |
| `writing-plans` | Viết kế hoạch phát triển |
| `executing-plans` | Thực thi kế hoạch |
| `test-driven-development` | Quy trình TDD |
| `systematic-debugging` | Debug có phương pháp |
| `requesting-code-review` | Yêu cầu review code |
| `receiving-code-review` | Xử lý feedback review |
| `using-git-worktrees` | Quản lý git worktrees |
| `dispatching-parallel-agents` | Chạy nhiều agent song song |
| `subagent-driven-development` | Phát triển bằng subagent |
| `verification-before-completion` | Kiểm tra trước khi báo xong |
| `finishing-a-development-branch` | Hoàn thiện branch |
| `writing-skills` | Viết skill mới cho framework |
| `writing-systems` | Xây dựng hệ thống |

---

## Triết lý cốt lõi

- **"Step back and ask what you're really trying to do"** — Đừng code ngay, hỏi rõ mục tiêu trước
- **Evidence over claims** — Phải verify, không tự tuyên bố xong
- **Systematic over ad-hoc** — Có quy trình, không làm bừa
- **TDD bắt buộc** — Test trước, code sau

---

## Cách học thực chiến

Đừng chỉ đọc — hãy thử ngay:

1. Nghĩ một tính năng nhỏ muốn build (ví dụ: todo list CLI)
2. Kích hoạt `brainstorming` skill → làm đúng 9 bước của nó
3. Sau khi có spec → chuyển sang `writing-plans`
4. Implement theo `test-driven-development`
