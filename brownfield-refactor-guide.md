# Áp dụng Superpowers cho Brownfield Project — Refactor an toàn

## Vấn đề khi refactor codebase có sẵn

Refactor mà không có quy trình → dễ break thứ đang chạy, khó biết khi nào "xong", không có bằng chứng là an toàn.

Superpowers giải quyết chính xác vấn đề này bằng 3 nguyên tắc cốt lõi.

---

## 3 nguyên tắc cốt lõi cho brownfield refactor

### 1. Evidence over claims
> "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"

Trước khi nói "refactor xong" hay "không break gì":
1. Xác định lệnh nào chứng minh được điều đó
2. Chạy lệnh đó ngay trong session hiện tại
3. Đọc toàn bộ output, kiểm tra exit code
4. Chỉ khi output xác nhận → mới được tuyên bố

Tránh dùng từ: *"should work"*, *"probably fine"*, *"seems okay"*.

### 2. TDD làm lưới an toàn
> "Write the test first. Watch it fail. Write minimal code to pass."

Với brownfield, TDD có vai trò **kép**:
- **Characterization tests**: viết test mô tả behavior *hiện tại* trước khi refactor → đây là lưới an toàn
- **Refactor tests**: sau khi refactor, toàn bộ test cũ phải vẫn xanh

Quy trình:
```
1. Viết test mô tả behavior hiện tại → chạy → phải XANH
2. Refactor code
3. Chạy lại test → phải vẫn XANH
4. Nếu có test đỏ → refactor đã break gì đó → revert và điều tra
```

### 3. Tôn trọng codebase hiện có
> "In existing codebases, follow established patterns. Don't unilaterally restructure."

- Giữ nguyên conventions, naming style, structure của codebase
- Chỉ refactor file đang chạm vào, không refactor lan sang chỗ khác
- Nếu file quá lớn và cần split để làm việc → ghi rõ vào plan, không làm ngầm

---

## Quy trình refactor an toàn step-by-step

### Bước 1 — Brainstorm trước khi làm
Kích hoạt `brainstorming` skill. Hỏi bản thân:
- Mục tiêu thực sự của refactor này là gì?
- Scope: refactor đến đâu thì dừng?
- Rủi ro: đoạn code này được dùng ở đâu?

Tạo file spec: `docs/superpowers/specs/YYYY-MM-DD-refactor-<tên>.md`

### Bước 2 — Viết characterization tests
Trước khi đụng vào code, viết test mô tả behavior hiện tại:
```
- Test input/output hiện tại (kể cả behavior xấu cần giữ lại)
- Test edge cases
- Chạy toàn bộ → phải xanh hết
```
Đây là "snapshot" trạng thái hiện tại. Nếu refactor break test này → có vấn đề.

### Bước 3 — Lập kế hoạch (writing-plans)
Chia nhỏ thành task 2-5 phút, mỗi task phải:
- Chỉ rõ file nào thay đổi
- Chỉ rõ cách verify sau khi xong
- Không có placeholder hay "TODO"

Ví dụ task tốt:
```
Task 3: Extract validateEmail() từ UserService.signup()
- File: src/services/UserService.ts (dòng 45-52)
- Tạo: src/utils/validators.ts
- Verify: chạy `npm test -- --testPathPattern=UserService` → tất cả xanh
```

### Bước 4 — Thực thi từng task
Với mỗi task:
1. Chạy test suite → baseline xanh
2. Làm thay đổi
3. Chạy test suite → phải vẫn xanh
4. Commit ngay

Commit nhỏ, thường xuyên → dễ revert nếu có vấn đề.

### Bước 5 — Verification trước khi báo xong
Chạy toàn bộ test suite lần cuối. Đọc output. Chỉ khi xanh hết mới tuyên bố xong.

---

## Khi nào dừng và điều tra

Nếu đã thử fix 3+ lần mà vẫn fail → **đừng tiếp tục patch**. Đây là dấu hiệu vấn đề kiến trúc, không phải bug nhỏ. Dùng `systematic-debugging`:

```
1. Root cause investigation — đọc error, trace data flow
2. Pattern analysis — tìm code tương tự đang chạy đúng
3. Hypothesis — đặt giả thuyết cụ thể, test từng cái
4. Implementation — fix một chỗ, verify không regression
```

---

## Red flags cần tránh

| Dấu hiệu | Ý nghĩa |
|---|---|
| "Refactor xong, chắc không sao" | Chưa chạy test → vi phạm verification |
| Sửa luôn khi thấy code xấu | Scope creep → tăng rủi ro |
| Không viết test trước | Không có lưới an toàn |
| Commit lớn cuối buổi | Khó revert khi có vấn đề |
| "Quick fix trước, test sau" | Sunk cost trap |

---

## Checklist trước khi merge

- [ ] Characterization tests đã viết và xanh trước khi refactor
- [ ] Toàn bộ test vẫn xanh sau refactor
- [ ] Đã chạy test suite trong session này (không dựa vào CI)
- [ ] Không có refactor lan sang code không liên quan
- [ ] Mỗi bước đã commit riêng
