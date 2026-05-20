# learn-super-powers

Tài liệu học tiếng Việt về **obra/superpowers** — framework phương pháp phát triển phần mềm AI-first.

> Repo gốc: [github.com/obra/superpowers](https://github.com/obra/superpowers)

---

## obra/superpowers là gì?

Một bộ "skills" dành cho AI coding agent (Claude, Gemini, Cursor, Copilot...) giúp agent làm việc có hệ thống thay vì code bừa. Framework định nghĩa workflow 7 bước từ brainstorming đến merge, với từng bước được encode thành skill file mà agent phải invoke trước khi hành động.

Triết lý cốt lõi:

- **Evidence over claims** — không tuyên bố "xong" khi chưa chạy verify
- **Systematic over ad-hoc** — có quy trình cho mọi tình huống
- **TDD bắt buộc** — test trước, code sau, không có ngoại lệ
- **Plan là source of truth** — plan và code phải luôn đồng bộ

---

## Nội dung repo này

| File | Nội dung |
|---|---|
| `learning-path.md` | Lộ trình đọc và 3 điều cần hiểu ngay từ đầu |
| `tdd-mindset-guide.md` | Học TDD đúng mental model — tại sao phải thấy test đỏ trước |
| `brownfield-refactor-guide.md` | Refactor codebase có sẵn an toàn theo superpowers |
| `brownfield-no-tdd-guide.md` | Dùng superpowers khi project chưa có test coverage |
| `selective-adoption-guide.md` | Tham khảo superpowers mà không áp dụng hoàn toàn |
| `review-checklist.md` | Checklist review project theo tiêu chuẩn agentskills + superpowers |
| `skills/brainstorming-deep-dive.md` | Phân tích chi tiết brainstorming skill — 9 bước, server mockup, spec reviewer |
| `skills/writing-plans-deep-dive.md` | Phân tích writing-plans skill — zero placeholders, plan reviewer |
| `skills/systematic-debugging-deep-dive.md` | Phân tích debugging skill — 4 phase, root-cause tracing, flaky tests |

---

## Workflow 7 bước của superpowers

```
1. Brainstorming / Design    → hỏi rõ mục tiêu, viết spec, subagent review spec
2. Setup môi trường          → git worktree để cô lập
3. Writing Plans             → chia nhỏ thành task 2-5 phút, zero placeholders
4. Executing Plans           → subagent làm từng task, có checkpoint review
5. Test-Driven Development   → red → green → refactor, bắt buộc không có ngoại lệ
6. Code Review               → review có hệ thống theo spec
7. Finalize Branch           → dọn dẹp, verify lần cuối, merge
```

`brainstorming` là **cửa vào duy nhất**. Không bỏ qua dù task nhỏ.

---

## Bắt đầu từ đâu

### Nếu chưa biết gì về superpowers

```
1. Đọc learning-path.md                  ← bức tranh toàn cảnh + thứ tự ưu tiên skill
2. Đọc tdd-mindset-guide.md             ← cần hiểu mental model này trước khi dùng framework
3. Thử FizzBuzz theo TDD (tự làm)       ← không đọc solution, mục tiêu là quen vòng đỏ-xanh-refactor
4. Đọc skills/brainstorming-deep-dive.md ← skill quan trọng nhất, được kích hoạt đầu tiên
```

### Nếu có project brownfield cần áp dụng

```
1. Đọc brownfield-no-tdd-guide.md       ← nếu chưa có test coverage
2. Đọc brownfield-refactor-guide.md     ← nếu cần refactor codebase có sẵn
3. Đọc selective-adoption-guide.md      ← nếu không muốn áp dụng toàn bộ framework
```

### Nếu muốn review project đang có

```
Đọc review-checklist.md
→ kiểm tra 3 hạng mục: agentskills format, superpowers workflow coverage, CLAUDE.md completeness
```

---

## 3 điều phân biệt superpowers với hướng dẫn thông thường

**1. Skills có thứ tự ưu tiên cứng:**

```
User instructions > Superpowers skills > Default system prompt
```

Skills ghi đè hành vi mặc định của AI — không phải gợi ý, là rule.

**2. Invoke skill TRƯỚC MỌI hành động** — kể cả trước khi hỏi clarifying question. Nếu thấy mình nghĩ "cái này đơn giản không cần skill" → đó là dấu hiệu đang tránh né skill.

**3. Hai loại skill:**
- **Rigid** (TDD, debugging): làm đúng 100%, không tự ý thích nghi
- **Flexible** (patterns): áp dụng tinh thần, không cứng nhắc theo từng chữ

---

## Danh sách skills trong repo gốc

| Skill | Khi nào dùng |
|---|---|
| `using-superpowers` | Meta-skill — định nghĩa khi nào invoke skill khác |
| `brainstorming` | Trước mọi feature, refactor — cửa vào duy nhất |
| `writing-plans` | Sau khi spec được approve |
| `executing-plans` | Thực thi từng task trong plan |
| `test-driven-development` | Mọi implementation — rigid, không ngoại lệ |
| `systematic-debugging` | Khi có bug — 4 phase bắt buộc |
| `requesting-code-review` | Yêu cầu review code |
| `receiving-code-review` | Xử lý feedback review |
| `using-git-worktrees` | Setup môi trường cô lập |
| `dispatching-parallel-agents` | Chạy nhiều agent song song |
| `subagent-driven-development` | Phát triển dùng subagent |
| `verification-before-completion` | Bắt buộc trước khi báo "xong" |
| `finishing-a-development-branch` | Dọn dẹp và hoàn thiện branch |
