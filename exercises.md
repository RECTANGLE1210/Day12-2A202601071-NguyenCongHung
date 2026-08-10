# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu bên dưới mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Công Hùng  Mã học viên: 2A202601071

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu deploy thiếu `AGENT_API_KEY`, Settings ném lỗi ngay khi process khởi động.
Nhờ vậy cloud báo deploy thất bại thay vì chạy với khóa chung `changeme`, qua đó
không cho người lạ gọi API và phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Ví dụ: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T00:00:00+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":20,"cost_usd":0.00001245}`.
Từ đó có thể lọc theo user để tìm người tiêu nhiều nhất và tính tỷ lệ lỗi/cost
theo khoảng thời gian; một câu `print` không có schema để truy vấn tự động.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Chưa đo được bản single-stage vì Docker daemon chưa chạy trong môi trường hiện
tại. Multi-stage dùng `python:3.11-slim` và chỉ mang dependency đã cài từ builder
sang runtime, nên loại compiler/cache và thường nhỏ hơn đáng kể.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi chỉ sửa `app/main.py`, layer copy và các layer sau nó phải chạy lại, còn
`COPY requirements.txt` và `pip install` được dùng lại từ cache. Nếu copy toàn
bộ source trước khi cài dependency thì mọi sửa code làm mất cache pip và cài lại
toàn bộ thư viện.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu code có lỗ hổng cho phép thực thi lệnh, process trong container có thể dùng
quyền root để đọc/sửa nhiều tài nguyên hơn hoặc khai thác đường thoát container.
`USER appuser` giới hạn process ở tài khoản không đặc quyền, giảm hậu quả nếu
lỗ hổng bị khai thác.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Có thể gửi 20 request trong 2 giây: 10 request ngay trước giây 00 của phút
trước và 10 request ngay sau giây 00 của phút kế tiếp. Hai nhóm thuộc hai phút
đồng hồ khác nhau dù cách nhau chỉ khoảng hai giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn số lần gọi trong một cửa sổ thời gian; cost guard giới hạn
tổng USD trong tháng. Một request ít nhưng prompt/output rất lớn có thể làm cost
guard chặn dù rate limit còn quota. Ngược lại, nhiều request rẻ có thể bị rate
limit chặn dù ngân sách tháng vẫn còn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Redis mất kết nối, endpoint chung trả 503, orchestrator coi cả ba container là
unhealthy và restart chúng. Load balancer không còn instance phục vụ; khi Redis
trở lại cả cụm có thể đã bị dừng, biến sự cố 30 giây thành outage lớn hơn.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Với Redis, `history_length` tăng 0, 2, 4... theo từng lượt hỏi dù request đi qua
instance nào. Với dict trong RAM, mỗi instance có bản sao riêng nên con số sẽ
nhảy/reset theo instance, không tăng ổn định.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Chưa thực hiện deploy cloud trong môi trường này vì chưa có tài khoản/token
platform. Vì vậy chưa có lỗi deploy thực tế để ghi nhận; bước tiếp theo là tạo
Redis, đặt `AGENT_API_KEY` trong dashboard, deploy rồi ghi lại log lỗi (nếu có)
và kết quả kiểm tra `/health` và `/ready`.
