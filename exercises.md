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

> Ví dụ khi mình deploy một revision mới mà quên thêm `AGENT_API_KEY`, app sẽ dừng ngay lúc khởi động và báo thiếu cấu hình, trước khi nhận traffic. Nếu có mặc định `changeme`, service vẫn chạy; người biết khóa mặc định có thể gọi `/ask` và dùng quota hoặc ngân sách của mình.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log mình lấy sau ba lượt gọi thử `/ask`: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:51:29.414915+00:00", "user_id": "exercise-cp1", "tokens_in": 143, "tokens_out": 56, "cost_usd": 5.505e-05}`. Mình có thể lọc log theo `event` hoặc `user_id` để tìm request cần xem, và cộng `tokens_in`, `tokens_out`, `cost_usd` để theo dõi mức sử dụng. Một dòng `print("đã trả lời xong")` không có các trường này để máy lọc hay tổng hợp.

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

> Mình dùng Dockerfile một stage ban đầu và Dockerfile multi-stage hiện tại để build cùng mã nguồn. Docker hiển thị image một stage là 1.73 GB, còn image multi-stage là 271 MB — giảm khoảng 1.46 GB. Phần chênh lệch chủ yếu đến từ image nền `python:3.11` đầy đủ của bản cũ; bản mới dùng `python:3.11-slim` làm runtime và chỉ chép thư viện Python cần thiết cùng mã `app` và `utils` từ stage build. Mã nguồn ứng dụng nhỏ nên không phải nguyên nhân chính của phần chênh lệch.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa một ký tự trong `app/main.py`, Docker vẫn dùng cache cho layer chép `requirements.txt` và cài package vì các layer đó đứng trước source code. Layer chép `app` phải chạy lại, cùng các layer phía sau nó. Nếu chuyển `COPY . .` lên trước `RUN pip install`, mỗi lần sửa code layer `COPY` đổi; layer cài package phía sau cũng mất cache và phải chạy lại dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu lỗ hổng cho phép chạy lệnh tùy ý, kẻ tấn công chiếm được process của app. Khi process chạy bằng root, họ có quyền root bên trong container, có thể đọc secret và sửa file; nếu container có mount hoặc runtime/kernel có lỗ hổng, họ có thể tìm đường ảnh hưởng tới host. `USER app` bỏ quyền root của process ngay từ đầu, nên giảm quyền và tác động của vụ khai thác; nó không bảo đảm rằng mọi lỗ hổng container escape đều biến mất.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: gửi 10 request ngay trước khi phút hiện tại kết thúc, rồi 10 request ngay sau khi phút mới bắt đầu. Bộ đếm theo phút tường sẽ reset ở ranh giới đó nên cả hai nhóm đều được chấp nhận. Sliding window 60 giây vẫn nhìn thấy đủ 20 request trong cùng cửa sổ và chặn khi vượt hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm số request trong 60 giây gần nhất; cost guard cộng chi phí theo user trong tháng. Nếu mình còn dưới 10 request trong cửa sổ nhưng số đã ghi nhận đã vượt ngân sách tháng, rate limit vẫn cho qua còn cost guard trả 402. Ngược lại, nếu gửi request thứ 11 trong 60 giây dù mỗi request rẻ và ngân sách tháng còn nhiều, rate limit trả 429 còn cost guard chưa chặn. Trong code hiện tại, cost guard kiểm tra số đã ghi nhận trước request; vì route không truyền chi phí ước tính, nó không dự đoán chính xác chi phí của chính request sắp chạy.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Đầu tiên, Redis mất kết nối nên endpoint gộp `/health` kiểm tra Redis và trả 503 ở cả ba container. Nếu nền tảng dùng endpoint này làm liveness probe, nó coi cả ba instance là hỏng và khởi động lại chúng; Redis vẫn mất kết nối nên các instance mới tiếp tục fail probe, làm service chập chờn hoặc không còn instance nhận request. Vì vậy `/health` nên chỉ kiểm tra process còn sống, còn `/ready` trả 503 để ngừng route traffic khi Redis lỗi. Với Docker Compose hiện tại, healthcheck đánh dấu container `unhealthy`, nhưng restart policy không tự restart container chỉ vì trạng thái đó.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Trong lần kiểm tra CP4 trước, với Redis dùng chung mình thấy `history_length` lần lượt là 0, 2, 4; mỗi lượt hỏi trước đó thêm một message của user và một của assistant. Nếu thay Redis bằng dict trong RAM, mỗi container sẽ giữ một bản lịch sử riêng. Request tới container chưa từng nhận user đó có thể lại trả 0, còn container đã xử lý vài lượt có thể trả 2 hoặc 4; vì vậy con số sẽ nhảy tùy request rơi vào replica nào, thay vì phản ánh một lịch sử chung.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi thật mình gặp khi chuẩn bị GCP là `The project property must be set to a valid project ID, not the project name [Day12]`. Mình đã nhập tên hiển thị `Day12` vào lệnh chọn project; thông báo của `gcloud` chỉ ra rằng chỗ này cần Project ID. Cách sửa là lấy đúng trường Project ID trong Cloud Console rồi dùng `gcloud config set project <PROJECT_ID>`. Đây là lỗi trước khi deploy: hiện mình chưa deploy Cloud Run nên chưa có lỗi build hay health check thực tế để ghi.
