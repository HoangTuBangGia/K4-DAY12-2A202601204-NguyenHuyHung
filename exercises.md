# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng trả lời mẫu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Huy Hưng  Mã học viên: 2A202601204

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu tôi quên đặt `API_TOKEN` khi deploy, ứng dụng sẽ dừng ngay và log của
> Render báo thiếu cấu hình. Nhờ vậy service chưa nhận traffic và người lạ
> không thể dùng một token mặc định như `"changeme"` để gọi API. Tôi phát hiện
> sai sót ngay trong lúc deploy thay vì sau khi service đã chạy công khai.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi nhận được có dạng:
> `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T07:39:30+00:00","client_id":"sv-test","prompt_tokens":5,"completion_tokens":40,"usd_cost":0.00002475}`.
> Từ log JSON này tôi có thể lọc và đếm request theo `client_id`, đồng thời cộng
> `usd_cost` để theo dõi hoặc cảnh báo chi phí. Dòng `print("đã trả lời xong")`
> không có các trường dữ liệu ổn định để máy thực hiện hai việc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Chưa đo riêng |
| Multi-stage | 267 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo image multi-stage `day12-chat:cp2-test` được 267 MB. Tôi chưa lưu và
> build riêng image một stage nên không ghi một số đo giả cho bản đó. Phần
> chênh lệch chủ yếu sẽ là base image Python đầy đủ, công cụ build/compiler và
> các file trung gian chỉ cần khi cài dependency. Multi-stage chỉ chép kết quả
> cài đặt từ builder sang runtime dùng image slim.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer base, `COPY requirements.txt` và
> `RUN pip install` vẫn dùng lại cache. Các layer `COPY app`, `COPY utils` và
> những layer đứng sau phần source phải được dựng lại. Nếu đặt `COPY . .` trước
> `RUN pip install`, mọi thay đổi source sẽ làm mất cache của layer cài thư viện,
> khiến pip chạy lại dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Giả sử code Python có lỗ hổng thực thi lệnh, kẻ tấn công có thể chạy lệnh với
> quyền của process trong container. Nếu process là root, hậu quả của một lỗi
> cấu hình mount, capability hoặc lỗ hổng thoát container sẽ nghiêm trọng hơn
> và có thể dẫn tới quyền cao trên host. `USER appuser` cắt chuỗi này tại bước
> thực thi lệnh: mã bị chiếm quyền chỉ chạy với UID 10001 và quyền hạn tối thiểu
> trong container.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> Response 401 cần `WWW-Authenticate: Bearer` để tuân theo chuẩn HTTP và cho
> client biết cơ chế xác thực cần dùng. Cả trường hợp thiếu header, sai scheme
> và sai token đều trả cùng thông báo để không tiết lộ bước xác thực nào đã
> đúng; nếu trả quá chi tiết, người dò token có thêm thông tin để thu hẹp cách
> tấn công.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Client gửi được tối đa 10 request liên tiếp vì xô chỉ chứa tối đa 10 token.
> Nếu bỏ `min(capacity, ...)`, sau 10 phút xô sẽ tính thêm 100 token; nếu trước
> lúc chờ xô đang đầy thì giá trị thành 110 và client có thể gửi 110 request
> liên tiếp. Giới hạn `capacity` ngăn thời gian im lặng biến thành một lượng
> burst không giới hạn.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD/tháng, sự cố từ 2 giờ sáng có thể tiêu gần 30 USD trước khi
> bị chặn và chỉ tự hồi phục khi sang tháng mới. Với hạn mức 1 USD/ngày, thiệt
> hại của client trong ngày bị giới hạn khoảng 1 USD và ngân sách tự tạo khóa
> mới khi sang ngày UTC tiếp theo. Hạn mức ngày vì vậy thu hẹp đáng kể phạm vi
> thiệt hại của một sự cố ngắn.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu endpoint chung kiểm tra Redis, khi Redis mất kết nối cả ba container cùng
> trả health check lỗi. Orchestrator coi cả ba process bị hỏng, loại chúng khỏi
> traffic rồi lần lượt restart chúng. Redis vẫn chưa phục hồi nên container mới
> lại fail health check, tạo vòng lặp restart và làm sự cố dependency lan thành
> sự cố toàn cụm. Tách `/healthz` giúp process vẫn được coi là sống; `/readyz`
> chỉ tạm rút instance khỏi traffic cho tới khi Redis hoạt động lại.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy Render của tôi thành công ngay lần đầu nên không có thông báo lỗi
> deploy thực tế để ghi lại. Tôi xác minh bằng deployment trạng thái Live,
> `/healthz` trả 200, `/readyz` trả 200 với `redis: true`, và `/chat` không có
> token trả 401 kèm `WWW-Authenticate: Bearer`. Điều này cũng xác nhận `$PORT`,
> `REDIS_URL` và `API_TOKEN` đã được cấu hình đúng trên Render.
