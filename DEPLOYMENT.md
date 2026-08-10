# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyen Cong Hung |
| Mã học viên | 2A202601071 |
| Repo | https://github.com/RECTANGLE1210/Day12-2A202601071-NguyenCongHung |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-yfsv.onrender.com |
| Platform | Render |
| Ngày deploy | 10/08/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Secret đặt trên Render, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis của Render (`day12-redis`) |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Cấu hình rate limit |
| `MONTHLY_BUDGET_USD` | ✅ | Cấu hình cost guard |
| `LOG_LEVEL` | ✅ | Cấu hình logging |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-yfsv.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-yfsv.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-yfsv.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-yfsv.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-yfsv.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```text
1.
HTTP/2 200 
date: Mon, 10 Aug 2026 04:05:20 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: 70c30d7b-b0dc-4c5f
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a28c1f223d598b92-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}

2.
HTTP/2 200 
date: Mon, 10 Aug 2026 04:07:05 GMT
content-type: application/json
rndr-id: 18ab6483-61bb-40c5
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a28c21adcdf04679-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}

3.
HTTP/2 401 
date: Mon, 10 Aug 2026 04:07:46 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: 6305421d-70d3-463b
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a28c22b2fd8b7558-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}

4.
HTTP/1.1 200 OK
Date: Mon, 10 Aug 2026 04:16:32 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 09f072d0-a487-4b14
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a28c2f895c7fd76d-HKG
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

5.
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```text
Không dùng phương án dự phòng; đã deploy thành công trên Render.
```
