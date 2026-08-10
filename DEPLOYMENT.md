# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Huy Hưng |
| Mã học viên | 2A202601204 |
| Repo | https://github.com/HoangTuBangGia/K4-DAY12-2A202601204-NguyenHuyHung |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-lvj5.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi nguồn cấu hình; không lưu giá trị token trong repository.

| Biến | Đã set | Nguồn giá trị |
|------|--------|---------------|
| `PORT` | ✅ | Render tự gán |
| `API_TOKEN` | ✅ | Render Dashboard, secret không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value connection string |
| `BUCKET_CAPACITY` | ✅ | Blueprint: 10 |
| `REFILL_PER_MINUTE` | ✅ | Blueprint: 10 |
| `DAILY_BUDGET_USD` | ✅ | Blueprint: 1.0 |
| `LOG_LEVEL` | ✅ | Blueprint: INFO |

## Lệnh Kiểm Tra

```bash
curl -i https://day12-chat-lvj5.onrender.com/healthz
curl -i https://day12-chat-lvj5.onrender.com/readyz
curl -i -X POST https://day12-chat-lvj5.onrender.com/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'
```

## Kết Quả Chạy Thật

```text
GET /healthz
HTTP/2 200
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

GET /readyz
HTTP/2 200
{"status":"ready","redis":true}

POST /chat không có Authorization
HTTP/2 401
www-authenticate: Bearer
{"detail":"invalid or missing bearer token"}
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png`: trang quản lý service trên Render
- `screenshots/healthz.png`: kết quả gọi endpoint `/healthz`
