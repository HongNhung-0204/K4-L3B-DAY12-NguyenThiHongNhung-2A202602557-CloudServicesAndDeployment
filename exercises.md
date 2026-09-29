# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay phần giữ chỗ dưới mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Thị Hồng Nhung
> Mã học viên: 2A202602557

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ, khi deploy một revision mới mà quên cấu hình `AGENT_API_KEY`, app sẽ báo thiếu cấu hình lúc khởi động nên revision đó không sẵn sàng nhận traffic. Nếu mặc định là `changeme`, app vẫn chạy và ai biết giá trị này cũng có thể gọi `/ask` như người dùng hợp lệ. Trong bài lab, LLM là bản giả lập nên không phát sinh hóa đơn API thật, nhưng request trái phép vẫn dùng tài nguyên service và làm sai lệch số liệu sử dụng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Đây là một dòng log mình ghi lại sau khi gọi thử `/ask`: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:51:29.414915+00:00", "user_id": "exercise-cp1", "tokens_in": 143, "tokens_out": 56, "cost_usd": 5.505e-05}`. Nhờ có cấu trúc này, mình lọc được các lần gọi theo `event` hoặc `user_id`, và có thể cộng token cùng chi phí để xem mức sử dụng. Dòng `print("đã trả lời xong")` vẫn tìm được bằng chữ, nhưng không có trường riêng để lọc theo user hay tổng hợp token và chi phí.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản               | Dung lượng                    |
| ----------------- | ----------------------------- |
| 1 stage (bản đầu) | 1.73 GB (1,727,798,982 bytes) |
| Multi-stage       | 271 MB (270,914,944 bytes)    |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Mình build hai image từ cùng mã nguồn: bản một stage là 1.73 GB, còn bản multi-stage là 271 MB, giảm khoảng 1.46 GB. Khác biệt lớn nhất là image nền `python:3.11` đầy đủ ở bản cũ so với `python:3.11-slim` ở runtime của bản mới. Bản multi-stage cũng chỉ đưa các thư viện đã cài và mã `app` với `utils` vào image cuối; các file của stage build không cần thiết lúc chạy sẽ không được giữ lại. Vì mã ứng dụng nhỏ, phần lớn mức giảm đến từ môi trường nền và nội dung không cần ở runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi mình sửa `app/main.py`, Docker vẫn dùng cache cho `COPY requirements.txt` và `RUN pip install`, vì nội dung dependency chưa đổi. Layer chép `app` bị làm lại; các lệnh sau nó cũng phải được xử lý lại theo layer mới. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa file trong build context sẽ làm layer `COPY` đổi, kéo theo cài package chạy lại dù `requirements.txt` vẫn y nguyên.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu lỗ hổng trong app cho phép chạy lệnh tùy ý, kẻ tấn công có thể điều khiển process của app. Chạy bằng root sẽ cho process quyền root trong container; nếu container có mount hoặc quyền đặc biệt, hay runtime/kernel có lỗ hổng, kẻ tấn công có thể tìm đường gây ảnh hưởng tới host. `USER app` chạy process bằng tài khoản ít quyền, cắt bớt bước leo từ quyền của app lên root trong container. Nó giảm rủi ro nhưng không tự ngăn mọi kiểu container escape, và cũng không giấu được secret đã cấp cho chính process.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request: gửi 10 request ngay trước mốc chuyển phút và 10 request ngay sau đó. Bộ đếm theo phút đồng hồ vừa reset nên cả hai nhóm đều lọt qua, dù chúng được gửi cách nhau chỉ khoảng hai giây. Với sliding window, 20 request đó vẫn nằm trong cùng cửa sổ 60 giây nên request vượt hạn mức sẽ bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong 60 giây; cost guard cộng chi phí đã ghi nhận theo từng user trong tháng. Nếu mình chưa chạm rate limit nhưng chi phí đã ghi nhận vượt ngân sách tháng, request vẫn qua bước rate limit rồi bị cost guard trả 402. Ngược lại, request thứ 11 trong 60 giây bị rate limit trả 429 dù ngân sách tháng còn nhiều. Trong code hiện tại, route gọi `guard.check(user_id)` mà không truyền chi phí ước tính, nên guard chỉ kiểm tra khoản đã ghi nhận trước đó, chưa tính được chi phí của request sắp chạy.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối nên endpoint gộp `/health` trả 503 ở cả ba container. Probe sẽ đánh dấu chúng không khỏe; nếu nền tảng dùng endpoint này làm liveness probe, nền tảng có thể khởi động lại cả ba. Vì Redis vẫn đang lỗi, instance mới cũng tiếp tục fail probe và vòng lặp có thể kéo dài. Riêng Docker Compose hiện tại chỉ đánh dấu container `unhealthy`; `restart: unless-stopped` không tự khởi động lại container chỉ vì trạng thái đó. Khi Redis kết nối lại, probe có thể thành công trở lại. Tách hai endpoint giúp `/health` báo process còn sống, còn `/ready` báo chưa thể nhận traffic khi Redis không truy cập được.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi kiểm tra với Redis dùng chung, mình thấy `history_length` lần lượt là 0, 2, 4; mỗi câu hỏi trước đó thêm một message của user và một của assistant. Nếu dùng dict trong RAM, mỗi container sẽ có lịch sử riêng. Một request có thể rơi vào replica chưa xử lý user đó và trả 0, trong khi replica khác đã có lịch sử và trả 2 hoặc 4. Vì vậy con số có thể tăng giảm giữa các lần gọi tùy request tới replica nào, thay vì thể hiện một lịch sử chung.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra `/ask` trên Render bằng CP5, test báo `{"detail":"invalid or missing API key"}` thay vì nhận được câu trả lời. Mình đối chiếu cấu hình mà test dùng với biến `AGENT_API_KEY` trên Render và phát hiện khóa kiểm tra ở máy chưa khớp. Mình cập nhật khóa trong môi trường local cho trùng với khóa đã lưu trên Render, không đưa giá trị khóa vào log hay tài liệu, rồi chạy lại test và `/ask` trả về thành công. Lỗi nằm ở xác thực request chứ không phải ở bước build image.
