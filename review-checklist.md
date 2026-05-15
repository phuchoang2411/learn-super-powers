# Review Checklist — Brownfield Project theo Superpowers + AgentSkills

## Bối cảnh

Dùng khi kêu Claude Code review một brownfield project đã có CLAUDE.md, Claude setup, và bộ skill.
Đối chiếu với 2 tiêu chuẩn độc lập, bổ sung nhau:
- **agentskills** — kiểm tra kỹ thuật cấu trúc file
- **obra/superpowers** — kiểm tra độ phủ workflow

---

## Tiêu chuẩn 1 — agentskills (format/structure)

Validator fail nếu vi phạm các rule sau:

| Vấn đề | Chi tiết |
|---|---|
| Thiếu YAML frontmatter trong SKILL.md | Bắt buộc có `name:` và `description:` |
| `name` không khớp tên thư mục | `skills/my-skill/SKILL.md` phải có `name: my-skill` |
| `name` sai format | Lowercase, kebab-case, max 64 ký tự |
| `description` quá dài | Max 1,024 ký tự |
| Có frontmatter field không hợp lệ | Chỉ cho phép: `name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility` |

**Cách fix:** Đi qua từng SKILL.md, thêm/sửa frontmatter cho đúng. Đây là việc máy móc, dễ fix.

Ví dụ frontmatter hợp lệ:
```yaml
---
name: systematic-debugging
description: Four-phase methodology for root cause investigation before attempting fixes.
---
```

---

## Tiêu chuẩn 2 — obra/superpowers (workflow coverage)

Kiểm tra độ phủ của workflow. Các skill thường thiếu:

```
brainstorming                   ← nhiều project bỏ qua design phase
writing-plans                   ← đặc biệt quan trọng với brownfield
test-driven-development         ← có thể có nhưng không đúng chuẩn rigid
systematic-debugging            ← thường quá chung chung
verification-before-completion  ← rất hay thiếu
using-superpowers               ← meta-skill định nghĩa khi nào invoke skill khác
finishing-a-development-branch  ← thường thiếu
```

**Cách fix:** Bổ sung skill còn thiếu, hoặc refactor skill hiện có theo đúng pattern:
- **Rigid skill** (TDD, debugging): agent phải làm đúng 100%, không tự ý thích nghi
- **Flexible skill** (patterns): áp dụng tinh thần, không cứng nhắc

---

## Tiêu chuẩn 3 — CLAUDE.md completeness

CLAUDE.md brownfield project thường thiếu 3 thứ:

**1. Priority hierarchy:**
```
User instructions > Superpowers skills > Default behavior
```

**2. Invocation rule:**
```
Invoke relevant skills BEFORE any response or action,
even before asking clarifying questions.
```

**3. Bootstrap skill** — chỉ định skill nào tự động load khi session bắt đầu
(thường là `brainstorming` hoặc `using-superpowers`)

---

## Thứ tự ưu tiên khi review

```
1. [agentskills] Fix SKILL.md frontmatter cho đúng format      ← nhanh, rõ ràng
2. [superpowers] Thêm meta-skill using-superpowers nếu thiếu   ← quan trọng nhất
3. [superpowers] Bổ sung skill thiếu trong workflow            ← tùy gap thực tế
4. [superpowers] Cập nhật CLAUDE.md với priority + invocation rules
5. [superpowers] Thêm verification-before-completion nếu thiếu ← hay bị bỏ qua
```

---

## Những gì KHÔNG nên đề xuất (sẽ bị reject)

Theo tiêu chuẩn của obra/superpowers:
- Refactor skill hiện có mà không có before/after evaluation
- Thêm skill domain-specific vào core (nên là plugin riêng)
- Đề xuất thay đổi speculative, không có bằng chứng user gặp vấn đề thực tế
- Bundle nhiều thay đổi vào một lần

---

## Prompt mẫu để kêu Claude Code review

```
Đọc 2 repo sau để hiểu tiêu chuẩn:
- https://github.com/obra/superpowers/tree/main
- https://github.com/agentskills/agentskills

Sau đó review project này theo 3 hạng mục:
1. agentskills: kiểm tra SKILL.md frontmatter format
2. superpowers: kiểm tra workflow coverage (skill nào còn thiếu)
3. CLAUDE.md: kiểm tra priority hierarchy và invocation rules

Với mỗi vấn đề tìm thấy: chỉ rõ file, dòng cụ thể, và cách fix.
Không đề xuất thay đổi speculative — chỉ report vấn đề có bằng chứng thực tế.
```
