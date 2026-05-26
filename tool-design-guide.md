# Tool Design trong AI Agents

Nguồn cảm hứng: bài viết của Hoang Dung Le về Agentic Software Team và stress test pipeline SDLC.

---

## Tool design là gì?

Tool design không phải "tool calling" (cơ chế API function calling).

Tool design là **cách bạn thiết kế những công cụ mà agent được phép dùng** — mỗi tool trả về gì, bao nhiêu thông tin, ở dạng nào, và khi nào dừng lại chờ human.

> *"Tại sao tool design lại là yếu tố sống còn"* — Hoang Dung Le

Vì agent chỉ giỏi bằng những tool nó được trao:
- Tool trả về quá nhiều → context window đầy → agent bị overwhelm → stuck
- Tool trả về quá ít → agent đoán mò → kết quả sai
- Tool scope sai → agent làm đúng lệnh nhưng sai mục đích

---

## Ví dụ thực tế từ Agentic Software Team (30 slash commands)

| Tool | Tool design kém | Tool design tốt |
|---|---|---|
| `/discover-codebase` | Dump toàn bộ file tree + nội dung | Trả về dependency graph có scope, chỉ phần liên quan đến task |
| `/write-unit-tests` | Đưa cả codebase vào context | Đưa đúng file + interface của dependencies |
| `/check-coverage` | Trả về full coverage report | Trả về danh sách file chưa đủ coverage + số dòng cần cover |

Triệu chứng tool design kém: agent bị stuck vì context window không đủ để nhìn toàn bộ dependency graph — không phải agent kém, mà tool đưa vào quá nhiều thứ không cần thiết.

---

## Tool design trong obra/superpowers

Superpowers đã áp dụng tool design ngay trong cấu trúc skill:

**Progressive disclosure = quyết định tool design:**
- SKILL.md < 200 dòng không phải tùy ý — đây là giới hạn tool design
- Agent đọc entry point → quyết định reference nào cần → chỉ nạp reference đó
- Nếu SKILL.md 1,000 dòng → context bị chiếm → agent không còn bandwidth để làm việc thực sự

**`find-polluter.sh` = well-designed tool:**
```bash
# Không dump toàn bộ test logs
# Chạy từng test một, dừng khi tìm ra thủ phạm
# Output: đúng một file + gợi ý điều tra tiếp
```
Agent nhận được đúng thứ cần — không hơn, không kém.

**`spec-document-reviewer-prompt.md` = well-designed tool:**
Subagent reviewer chỉ nhận spec file + checklist 5 điều cần kiểm tra.
Không nhận toàn bộ conversation history → không bị bias → review độc lập.

---

## HITL — tool design quan trọng nhất

HITL (Human-in-the-loop) là quyết định thiết kế về **khi nào trao quyền cho agent, khi nào giữ lại cho người**.

```
# Tool design kém — agent tự quyết toàn bộ
/implement-task → /run-tests → /merge-pr   (không có checkpoint)

# Tool design tốt — human ở đúng điểm quyết định
/implement-task → /run-tests → /create-pr → [human review] → /merge-pr
```

Dung Le nhắc HITL "không phải là điểm yếu của hệ thống mà là điểm neo quan trọng nhất" — vì nó là nơi human inject judgment vào đúng chỗ cần nhất.

Trong superpowers: bước 8 của brainstorming (`User reviews written spec`) là HITL được thiết kế có chủ đích — agent không được tự chuyển sang writing-plans mà phải đợi approval.

---

## Nguyên tắc tool design tốt

**1. Scope rõ ràng**
Mỗi tool làm đúng một việc, trả về đúng thứ cần cho bước tiếp theo.
Không trả về "mọi thứ liên quan" — chỉ trả về "thứ agent cần để quyết định tiếp theo".

**2. Output có cấu trúc**
Agent parse output dễ hơn khi có format nhất quán.
`find-polluter.sh` trả về filename + ls -la + suggestion — không phải raw log.

**3. Fail rõ ràng**
Tool nên báo lỗi cụ thể, không im lặng.
`/check-coverage` block merge khi < 80% và nói rõ file nào thiếu — không chỉ "coverage insufficient".

**4. HITL ở đúng điểm**
Không phải mọi bước đều cần human. Nhưng những bước có rủi ro cao (merge, deploy, xóa dữ liệu) thì cần.

---

## Kết nối với quá trình học superpowers

Tool design là lý do các quy tắc sau tồn tại:

| Quy tắc superpowers | Lý do tool design |
|---|---|
| SKILL.md < 200 dòng | Giữ context cho agent đủ bandwidth xử lý |
| References nạp theo yêu cầu | Chỉ đưa thông tin khi agent thực sự cần |
| Spec reviewer là subagent độc lập | Tránh context bias từ conversation dài |
| Brainstorming chờ approval ở bước 8 | HITL ở điểm quyết định quan trọng |
| Plan không có placeholder | Tool (plan file) phải đủ rõ để subagent dùng không cần hỏi thêm |

Khi đọc log thấy agent stuck → đặt câu hỏi: **tool đang trả về gì, và agent thực sự cần gì?**
Câu trả lời thường chỉ ra chỗ cần cải thiện tool design, không phải cải thiện agent.
