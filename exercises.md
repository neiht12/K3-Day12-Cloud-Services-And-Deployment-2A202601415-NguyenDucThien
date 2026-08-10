# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu deploy lên Railway mà quên set `AGENT_API_KEY`, app sẽ dừng ngay lúc khởi động. Nhờ vậy mình biết cấu hình secret đang thiếu trước khi public URL được mở. Nếu để mặc định `"changeme"`, service vẫn chạy và người khác có thể đoán key đó để gọi `/ask`, làm tốn quota hoặc chi phí mà mình chỉ phát hiện sau.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ log: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T04:39:14.490163+00:00","user_id":"sv-test","tokens_in":4,"tokens_out":41,"cost_usd":0.0000252}`. Với log JSON này mình có thể lọc theo `event` hoặc `user_id`, và có thể cộng/tổng hợp `cost_usd`, `tokens_in`, `tokens_out` để theo dõi chi phí. Một dòng `print("đã trả lời xong")` không có cấu trúc nên máy khó lọc, khó thống kê.

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
| 1 stage (bản đầu) | chưa đo trên máy này |
| Multi-stage | chưa đo trên máy này |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Multi-stage nhỏ hơn vì runtime image chỉ mang Python runtime, thư viện đã cài và source code cần chạy. Phần chênh lệch chủ yếu là cache build, compiler, tool build, file tạm của pip và những thứ chỉ cần ở giai đoạn cài dependency. Với Dockerfile hiện tại, stage `builder` cài dependency, stage `runtime` chỉ copy kết quả sang nên image cuối gọn hơn.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa một ký tự trong `app/main.py`, các layer `FROM`, `WORKDIR`, `COPY requirements.txt` và `RUN pip install` được dùng lại từ cache. Layer phải chạy lại là `COPY . .` và các layer sau nó nếu có. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa code Docker sẽ coi layer copy source thay đổi, làm `pip install` chạy lại dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app Python có lỗ hổng cho phép chạy lệnh trong container, kẻ tấn công sẽ có quyền của user đang chạy process. Nếu container chạy root, họ có quyền root trong container và nếu có thêm cấu hình mount/socket sai thì có thể leo thang ảnh hưởng tới host. Lệnh `USER appuser` cắt chuỗi này bằng cách làm process chỉ chạy bằng user thường, nên khi bị khai thác cũng không có quyền root trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với hạn mức 10/phút, nếu đếm theo phút đồng hồ thì người dùng có thể gửi 20 request trong khoảng 2 giây: 10 request lúc 10:00:59 và 10 request lúc 10:01:01. Vì bộ đếm reset ở giây 00 nên hai nhóm request nằm ở hai phút khác nhau, dù thực tế chỉ cách nhau khoảng 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trong một khoảng thời gian, còn cost guard giới hạn tổng chi phí đã tiêu. Rate limit có thể cho qua nếu user gửi ít request, nhưng cost guard phải chặn khi mỗi request rất đắt hoặc user đã gần hết ngân sách tháng. Ngược lại, cost guard có thể cho qua vì chi phí còn thấp, nhưng rate limit phải chặn khi user spam quá nhiều request nhỏ trong 60 giây.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp `/health` và `/ready` rồi cho nó kiểm tra Redis, khi Redis mất kết nối 30 giây thì cả 3 container sẽ bắt đầu trả healthcheck lỗi. Orchestrator hiểu nhầm là process chết, nên restart các container dù app vẫn còn sống. Các container mới khởi động lại cũng chưa gọi được Redis, tiếp tục fail healthcheck, làm cụm bị restart vòng lặp và traffic bị gián đoạn.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi lưu lịch sử trong Redis, các instance khác nhau vẫn nhìn thấy cùng dữ liệu nên `history_length` tăng ổn định theo các lần gọi cùng `X-User-Id`. Nếu lưu bằng dict Python trong RAM, mỗi container có một dict riêng. Khi load balancer gửi request sang container khác, `history_length` sẽ lúc tăng, lúc quay về 0 hoặc một số nhỏ hơn vì container đó không có lịch sử của request trước.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi mình gặp là Railway báo healthcheck/application failed hoặc `/ready` không nối được Redis. Mình xem Deploy Logs và thấy app đã trả `/health` 200, nên process sống; sau đó log Redis báo `Redis URL must specify one of the following schemes (redis://, rediss://, unix://)`. Nguyên nhân là biến `REDIS_URL` trên Railway bị set sai, không phải URL Redis đầy đủ. Cách sửa là vào service app trên Railway, tab Variables, đặt `REDIS_URL` thành reference variable đúng dạng `${{Redis.REDIS_URL}}` theo tên service Redis, rồi redeploy.
