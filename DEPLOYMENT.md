# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Đức Thiện |
| Mã học viên | 2A202601415 |
| Repo | (điền link repo DAY12-...) |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis service/add-on trên Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: 1lXppkX2jLPOhY1DQfzuRZUv-827DFGPc-IDEqG_uHg" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k3-day12-cloud-services-and-deployment-2a2026014-production.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: 1lXppkX2jLPOhY1DQfzuRZUv-827DFGPc-IDEqG_uHg" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:
# 1. Liveness
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:23:44 GMT
server: railway-hikari
x-railway-request-id: pn9dZ7oxTgyUSLiWV7rehQ
content-length: 57
x-hikari-trace: sin1.nzn2
x-railway-edge: sin1

# 2. Readiness
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:26:17 GMT
server: railway-hikari
x-railway-request-id: uwq2Hcj3TYi1Kl8B0ubPiw
content-length: 31
x-hikari-trace: sin1.hs0s
x-railway-edge: sin1

# 3. Không API KEY
HTTP/2 401 
content-type: application/json
date: Mon, 10 Aug 2026 05:27:21 GMT
server: railway-hikari
x-railway-request-id: YC7VSQLbQnGxOWgzV7rehQ
content-length: 39
x-hikari-trace: sin1.tr00
x-railway-edge: sin1

{"detail":"invalid or missing API key"}

# 4. Có API KEY
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:29:04 GMT
server: railway-hikari
x-railway-request-id: G7-BRnl_RpCzhlRlV7rehQ
content-length: 279
x-hikari-trace: sin1.tr00
x-railway-edge: sin1
vary: accept-encoding

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

# 5.Rate Limit
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

