# Deep Dive: Brainstorming Skill

Nguồn: https://github.com/obra/superpowers/tree/main/skills/brainstorming

---

## Cấu trúc thực tế của skill

```
skills/brainstorming/
├── SKILL.md                         ← quy trình 9 bước chính
├── spec-document-reviewer-prompt.md ← template cho subagent review spec
├── visual-companion.md              ← hướng dẫn dùng browser tool cho mockup
└── scripts/
    ├── start-server.sh              ← khởi động server mockup
    ├── stop-server.sh
    ├── server.cjs                   ← Node.js server phục vụ HTML mockup
    ├── frame-template.html          ← wrapper tự động cho mockup content
    └── helper.js
```

Đây không chỉ là file text hướng dẫn — có cả **server chạy được** để render mockup trực tiếp trong browser khi brainstorm.

---

## 9 bước chi tiết — điều thực sự xảy ra

```
Bước 1: Explore context
  → Đọc file hiện có, docs, git log gần đây
  → KHÔNG hỏi gì trước khi làm bước này

Bước 2: Offer visual companion
  → Nếu topic cần visual: hỏi user có muốn xem mockup không
  → Offer RIÊNG, không kèm nội dung khác trong cùng message
  → Nếu user từ chối: tiếp tục bình thường

Bước 3: Clarifying questions
  → Một câu hỏi / một message
  → Ưu tiên multiple-choice hơn open-ended
  → Hỏi về mục đích, constraints — không hỏi về implementation

Bước 4: Propose 2-3 approaches
  → Mỗi approach: trade-offs rõ ràng
  → Có recommendation cụ thể

Bước 5: Present design
  → Chia theo mức độ phức tạp
  → Xin approval từng section, không dump hết một lúc

Bước 6: Write design doc
  → Lưu vào: docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md
  → Phải là file thực, không phải chỉ nói miệng

Bước 7: Spec self-review (subagent)
  → Dispatch subagent riêng chạy spec-document-reviewer-prompt.md
  → Kiểm tra 5 chiều: completeness, consistency, clarity, scope, YAGNI
  → Chỉ flag issue nếu sẽ gây vấn đề thực sự khi planning

Bước 8: User reviews spec
  → Đợi approval — không tự chuyển sang bước tiếp

Bước 9: Transition
  → Invoke DUY NHẤT writing-plans skill
  → Không làm gì khác
```

---

## 3 điểm tinh tế hay bị bỏ qua

### 1. Hard gate tuyệt đối

> *"Simple projects are where unexamined assumptions cause the most wasted work."*

Dù task nhỏ đến đâu — một cái button, một API endpoint — cũng phải qua brainstorming trước khi code. Không có ngoại lệ.

### 2. Visual companion có logic riêng

Không phải cứ UI là dùng visual. Rule là: *chỉ khi câu hỏi cần visual để trả lời*.

| Câu hỏi | Dùng gì |
|---|---|
| "Nên dùng approach nào?" | Text đủ |
| "Layout này trông ổn không?" | Visual |
| "Architecture có hợp lý không?" | Diagram → Visual |

**Cách hoạt động của server mockup:**
1. Agent viết HTML fragment (chỉ inner content, không cần `<!DOCTYPE>`)
2. Server tự wrap với CSS/JS
3. User thấy trong browser, click chọn option
4. Click event được log ra JSON → agent đọc để hiểu preference

### 3. Spec reviewer là subagent độc lập

Bước 7 không phải self-check. Nó dispatch một **agent mới**, không có context của conversation, để review spec.

Mục đích: tránh bias — agent viết spec thường không thấy lỗ hổng trong spec của chính mình.

**5 chiều reviewer kiểm tra:**
- **Completeness** — có section nào bỏ dở, TODO không?
- **Consistency** — có mâu thuẫn nội bộ không?
- **Clarity** — yêu cầu nào mơ hồ, dễ hiểu sai?
- **Scope** — có đủ hẹp cho một planning cycle không?
- **YAGNI** — có feature nào chưa được yêu cầu nhưng tự thêm vào không?

Chỉ block nếu issue sẽ gây vấn đề thực sự khi planning — không block vì wording chưa hoàn hảo.

---

## Khi nào skill này được kích hoạt?

| Tình huống | Dùng skill gì |
|---|---|
| "Làm cho tôi một..." | brainstorming |
| "Thêm feature..." | brainstorming |
| "Refactor cái này..." | brainstorming (cần biết scope trước) |
| "Fix bug này..." | systematic-debugging (không dùng brainstorming) |
| "Giải thích code này..." | Không cần skill |

---

## Spec document format chuẩn

File lưu tại: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

Nội dung mỗi section: *"a few sentences if straightforward, up to 200-300 words if nuanced"*

Mỗi component trong design phải trả lời được 3 câu hỏi:
1. Nó làm gì?
2. Được dùng như thế nào?
3. Phụ thuộc vào gì?

---

## Liên kết với skill khác

```
brainstorming
    └─→ writing-plans          (bước 9, transition bắt buộc)
            └─→ executing-plans / subagent-driven-development
                    └─→ test-driven-development
                            └─→ verification-before-completion
                                    └─→ finishing-a-development-branch
```

brainstorming là **cửa vào duy nhất** của toàn bộ workflow.
