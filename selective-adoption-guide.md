# Selective Adoption — Tham khảo Superpowers + AgentSkills mà không áp dụng hoàn toàn

## Bối cảnh

Project brownfield đã có CLAUDE.md và bộ skill riêng.
Muốn bổ sung ý tưởng từ 2 repo nhưng không muốn restructure hay thay thế những gì đang chạy.

**Nguyên tắc cốt lõi: thêm vào, không thay thế.**

---

## Phần 1 — Chuẩn hóa skill format (agentskills)

Không đổi nội dung skill, chỉ thêm YAML frontmatter vào đầu mỗi SKILL.md:

```yaml
---
name: tên-skill-hiện-tại        # phải khớp tên thư mục, kebab-case
description: Mô tả ngắn gọn mục đích của skill này, khi nào dùng.
---

(nội dung skill hiện tại giữ nguyên)
```

**Lợi ích:** skill dùng được với `skills-ref` validator và hệ sinh thái agentskills sau này — không mất gì hiện tại.

**Rules của agentskills:**
- `name`: lowercase, kebab-case, max 64 ký tự, phải khớp tên thư mục
- `description`: max 1,024 ký tự
- Chỉ cho phép các field: `name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`

---

## Phần 2 — Cải thiện CLAUDE.md (superpowers)

Không viết lại CLAUDE.md. Chỉ thêm một block nhỏ vào đầu file:

```markdown
## Skill Invocation Rules

Priority: User instructions > Project skills > Default behavior

Before any response or action — including before asking questions
or exploring the codebase — check if a skill applies.
If probability ≥ 1%, invoke it first.

Skill types in this project:
- Rigid: [tên skill] — follow exactly, no adaptation
- Flexible: [tên skill] — apply principles contextually
```

Phần còn lại của CLAUDE.md giữ nguyên. Chỉ cần thêm invocation logic này là agent sẽ dùng skill đúng thời điểm hơn.

---

## Phần 3 — Bổ sung skill còn thiếu (superpowers)

Chỉ lấy skill giải quyết pain point thực tế — không copy toàn bộ bộ skill superpowers:

| Nếu project hay gặp... | Lấy skill này |
|---|---|
| Agent code xong mà không biết có đúng không | `verification-before-completion` |
| Fix bug mà không tìm root cause | `systematic-debugging` |
| Không có test, sợ break khi sửa | `test-driven-development` |
| Agent làm xong mà chưa dọn branch | `finishing-a-development-branch` |
| Yêu cầu mơ hồ, agent hiểu sai | `brainstorming` |

**Cách lấy:** copy SKILL.md từ superpowers → thêm frontmatter agentskills → đặt vào thư mục skill của project.

---

## Phần 4 — Cải thiện workflow (chọn 1 vòng lặp)

Không áp dụng cả 7 bước superpowers cùng lúc. Chọn một vòng lặp phù hợp nhất rồi làm quen trước:

```
Option A — Tối giản (dễ nhất để bắt đầu):
  brainstorming → verification-before-completion

Option B — Feature workflow:
  brainstorming → writing-plans → TDD → verification-before-completion

Option C — Debug workflow:
  systematic-debugging → TDD → verification-before-completion
```

Khi team đã quen một vòng lặp → mới thêm bước tiếp theo.

---

## Prompt mẫu để dùng ngay

```
Project này có sẵn CLAUDE.md và bộ skill ở [đường dẫn].

Tham khảo ý tưởng từ:
- https://github.com/obra/superpowers (workflow methodology)
- https://github.com/agentskills/agentskills (skill format spec)

Yêu cầu:
1. Thêm YAML frontmatter vào các SKILL.md hiện có (agentskills format)
   — không thay đổi nội dung skill
2. Thêm skill invocation block vào đầu CLAUDE.md
   — không viết lại phần còn lại
3. Đề xuất tối đa 2 skill còn thiếu dựa trên gap thực tế
   — chỉ lấy từ superpowers, không thêm skill mới tự nghĩ ra
4. Không restructure project, không đổi tên file hiện có
```
