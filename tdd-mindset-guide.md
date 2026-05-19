# Học TDD để dùng Superpowers hiệu quả

## Vấn đề thực sự khi bắt đầu TDD

TDD không khó về kỹ thuật. Khó ở **mental model** — phải học cách nghĩ ngược lại so với thói quen:

| Thói quen bình thường | TDD |
|---|---|
| Nghĩ → Code → Test (để verify) | Nghĩ → Test (để specify) → Code |
| Test là để kiểm tra code đúng không | Test là để mô tả code phải làm gì |
| Viết test sau khi biết kết quả | Viết test khi chưa có code nào |

Nếu không thay đổi mental model này → sẽ vẫn viết "test sau" dù dùng superpowers.

---

## 3 điều cần hiểu trước khi viết dòng test đầu tiên

### 1. RED không phải thất bại — đó là mục tiêu

Khi test đỏ, nghĩa là bạn đã mô tả được điều mình muốn build.
Chưa có test đỏ = chưa biết mình đang build gì.

Cảm giác khó chịu khi thấy test đỏ là bình thường — đó là thói quen cũ.
Thực ra đỏ đúng chỗ = bạn đang kiểm soát tình huống.

### 2. GREEN nghĩa là "đủ để pass" — không phải "code tốt"

Ở bước Green, mục tiêu duy nhất là làm test xanh bằng cách đơn giản nhất có thể.
Kể cả hardcode giá trị cũng được — sẽ refactor sau.

Sai lầm hay gặp: cố viết code "tốt" ngay ở bước Green → mất tập trung, dễ over-engineer.

### 3. REFACTOR chỉ được làm khi đang XANH

Refactor = cải thiện code mà không thay đổi behavior.
Test xanh là lưới an toàn. Nếu refactor làm test đỏ → bạn đã thay đổi behavior.

Không được refactor khi đang đỏ — đó là implement, không phải refactor.

---

## Vòng lặp cần thuộc lòng

```
Viết một test nhỏ
    ↓
Chạy → phải ĐỎ (nếu xanh ngay → test sai hoặc feature đã có)
    ↓
Viết code tối thiểu để xanh
    ↓
Chạy → phải XANH (nếu vẫn đỏ → investigate, không đoán mò)
    ↓
Refactor nếu cần (test vẫn phải xanh)
    ↓
Lặp lại với test tiếp theo
```

Mỗi vòng lặp: 2-5 phút. Nếu một vòng mất hơn 10 phút → test quá lớn, chia nhỏ hơn.

---

## Tại sao phải thấy test ĐỎ trước

Đây là điểm nhiều người bỏ qua nhưng quan trọng nhất.

Nếu test xanh ngay mà không viết code → test không kiểm tra được gì.
Nếu không thấy đỏ → không có bằng chứng là test đang đo đúng thứ cần đo.

Superpowers nói rõ: *"Verify RED — run your test and confirm it fails for the right reason."*
"Right reason" nghĩa là: đỏ vì feature chưa có — không phải vì syntax error hay import sai.

---

## Test là specification, không phải verification

Đây là mental model quan trọng nhất.

**Verification** (test sau): "Code tôi vừa viết có đúng không?"
**Specification** (test trước): "Code tôi sắp viết cần làm gì?"

Khi viết test trước, bạn buộc phải trả lời:
- Function này nhận input gì, trả output gì?
- Edge case nào cần handle?
- Interface trông như thế nào từ góc độ người dùng?

Những câu hỏi này nếu không hỏi trước → sẽ hỏi sau khi đã viết code → thường dẫn đến refactor lớn.

---

## Một test tốt trông như thế nào

Test tốt trả lời được: *"Nếu test này fail, tôi biết ngay vấn đề ở đâu."*

```ts
// Test tốt — rõ ràng, một behavior, tên mô tả đủ để đọc không cần xem code
it('trả về lỗi khi email không có @ ', () => {
  const result = validateEmail('notanemail')
  expect(result.valid).toBe(false)
  expect(result.error).toBe('Email phải chứa @')
})

// Test xấu — test nhiều thứ cùng lúc, không biết fail ở đâu
it('validate email', () => {
  expect(validateEmail('a@b.com').valid).toBe(true)
  expect(validateEmail('').valid).toBe(false)
  expect(validateEmail('notanemail').valid).toBe(false)
  expect(validateEmail('a@').valid).toBe(false)
})
```

Nguyên tắc: **một test = một behavior = một lý do để fail**.

---

## Mock — dùng khi nào, tránh khi nào

Superpowers nói rõ: *"Prefer real code over mocks when possible."*

| Dùng real code | Dùng mock |
|---|---|
| Function thuần (input → output) | External API, database |
| Business logic | Hệ thống gửi email, SMS |
| Validation, transformation | File system trong test unit |

Dấu hiệu mock quá nhiều: test xanh nhưng production fail. Mock đã che đi vấn đề thực tế.

---

## Áp dụng vào Superpowers

Khi superpowers nói *"Write the test first. Watch it fail."* — bây giờ bạn hiểu tại sao:

- **Watch it fail** = verify RED = có bằng chứng test đang đo đúng thứ
- **Write minimal code** = GREEN phase, không over-engineer
- **Delete code if written before test** = vì code không có failing test = không có specification

Superpowers TDD là **rigid skill** — không có ngoại lệ. Hiểu mental model thì rule này tự nhiên, không thấy gò bó.

---

## Lộ trình thực hành

```
Ngày 1: Tự làm FizzBuzz theo TDD (không xem solution)
  → Mục tiêu: quen với vòng đỏ-xanh-refactor

Ngày 2: Tự làm validateEmail theo TDD
  → Mục tiêu: viết nhiều test nhỏ thay vì một test lớn

Ngày 3: Đọc lại TDD SKILL.md của superpowers
  → Lúc này các rule sẽ tự nhiên, không cần học thuộc

Tuần 2+: Áp dụng vào brownfield — bắt đầu từ characterization tests
  → Xem brownfield-no-tdd-guide.md
```

Không cần hoàn hảo trước khi bắt đầu. Làm FizzBuzz sai cũng không sao — sai ở đây tốt hơn sai trong production.
