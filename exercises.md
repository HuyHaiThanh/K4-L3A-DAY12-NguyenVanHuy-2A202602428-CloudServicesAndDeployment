# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> C?c c?u tr? l?i ?? ???c ?i?n b?n d??i.` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> H? v? t?n: NguyenVanHuy  M? h?c vi?n: 2A202602428

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

N?u deploy service l?n cloud m? qu?n ??t `AGENT_API_KEY`, c?ch fail fast l?m process d?ng ngay khi kh?i ??ng v? log cho bi?t c?u h?nh b?t bu?c ?ang thi?u. Nh? v?y t?i ph?t hi?n l?i tr??c khi service nh?n traffic ho?c ph?t sinh chi ph?. N?u ??t m?c ??nh l? `changeme`, app v?n ch?y v? ng??i kh?c c? th? ?o?n ???c kh?a ?? g?i `/ask`; l?i c?u h?nh c? th? ch? ???c ph?t hi?n sau khi b? d?ng tr?i ph?p ho?c ph?t sinh h?a ??n.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

M?t log t?i quan s?t ???c sau khi g?i `/ask` l?:

```json
{"event":"ask_completed","level":"info","timestamp":"2026-09-28T09:47:57+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":35,"cost_usd":0.00002145}
```

V? c?c tr??ng c? c?u tr?c, t?i c? th? l?c request c?a m?t `user_id` v? t?nh t?ng `cost_usd` theo user ho?c kho?ng th?i gian. T?i c?ng c? th? ??m l?i ho?c t?o c?nh b?o d?a tr?n `event` v? `level`. M?t d?ng `print("?? tr? l?i xong")` kh?ng c? c?c tr??ng ?? h? th?ng log truy v?n v? t?ng h?p t? ??ng.

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

T?i ?o b?ng hai image d?ng c?ng base `python:3.11-slim` v? c?ng requirements:

| B?n | Dung l??ng |
|-----|-----------:|
| 1 stage (b?n t?m ?o) | 64.8 MiB (67,937,863 bytes) |
| Multi-stage | 60.9 MiB (63,863,885 bytes) |

B?n multi-stage ch? copy k?t qu? c?n ch?y t? builder sang runtime, n?n kh?ng mang theo c?ng c? ho?c ph?n trung gian c?a b??c build. Trong repo n?y dependency kh?ng c?n compiler n?n ch?nh l?ch kh?ng l?n, nh?ng c?ch t?ch stage v?n l?m ranh gi?i runtime r? h?n v? tr?nh ??a tool build v?o production image.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Trong Dockerfile hi?n t?i, `requirements.txt` ???c copy v? c?i ??t tr??c khi source code ???c copy. N?u t?i s?a m?t k? t? trong `app/main.py`, c?c layer base image, `WORKDIR`, copy `requirements.txt` v? `pip install` v?n ???c d?ng l?i t? cache; c?c layer copy source v? sau ?? m?i ch?y l?i. N?u ??t `COPY . .` tr??c `RUN pip install`, m?i thay ??i trong code c?ng l?m layer copy source thay ??i. Docker s? b? cache t? ?? tr? ?i v? c?i l?i dependency d? `requirements.txt` kh?ng ??i.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

N?u ?ng d?ng c? l? h?ng cho ph?p attacker th?c thi l?nh, process trong container c? quy?n root s? c? th? ??c ho?c s?a nhi?u file h?n, c?i c?ng c? kh?c v? t?m c?ch khai th?c ti?p c?c quy?n c?a container ho?c host. Khi khai th?c container ch?y root, h?u qu? ti?m t?ng l?n h?n. L?nh `USER appuser` l?m process ch?nh ch?y b?ng user th??ng UID 10001, c?t quy?n root t?i ranh gi?i container v? gi?m t?c ??ng n?u ?ng d?ng b? x?m nh?p.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

V?i h?n m?c 10 request/ph?t nh?ng b? ??m reset theo ph?t ??ng h?, user c? th? g?i 10 request l?c 10:00:59 r?i g?i th?m 10 request l?c 10:01:00 ho?c 10:01:01. Nh? v?y c? th? g?i 20 request trong kho?ng hai gi?y. Sliding window lu?n ??m c?c request trong 60 gi?y g?n nh?t n?n nh?m ??u v?n c?n trong c?a s? v? nh?m th? hai s? b? ch?n.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit gi?i h?n s? l?n g?i trong m?t kho?ng th?i gian; cost guard gi?i h?n t?ng chi ph? c?a t?ng user trong m?t th?ng. Rate limit cho qua nh?ng cost guard ph?i ch?n khi user m?i g?i m?t request duy nh?t c? chi ph? l?n h?n ph?n ng?n s?ch c?n l?i. Ng??c l?i, cost guard v?n c? th? cho qua khi user c?n ng?n s?ch, nh?ng rate limit ph?i ch?n khi user g?i request th? 11 trong c?ng 60 gi?y.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

N?u g?p hai endpoint v? cho endpoint ?? ki?m tra Redis, khi Redis m?t k?t n?i 30 gi?y c? ba container s? b?o unhealthy. Orchestrator c? th? hi?u nh?m r?ng process c?n restart v? restart c? ba instance c?ng l?c. Trong l?c Redis l?i, c?m m?t c?c instance ?ang ph?c v?; khi Redis kh?i ph?c, c?c container c? th? v?n ?ang kh?i ??ng l?i ho?c ch?a s?n s?ng. Thi?t k? hi?n t?i t?ch hai c?u h?i: `/health` ch? ki?m tra process ?? tr?nh restart oan, c?n `/ready` ki?m tra Redis ?? load balancer ng?ng g?i traffic t?i instance ch?a s?n s?ng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Khi history n?m trong Redis, c?c request c?a c?ng user v?n ??c ???c c?ng m?t l?ch s? d? load balancer chuy?n request gi?a ba container. `history_length` v? v?y t?ng theo c?c message ?? l?u. N?u history n?m trong m?t dict Python, m?i container s? c? m?t dict ri?ng. Khi request chuy?n sang container kh?c, container ?? kh?ng bi?t c?c message tr??c v? `history_length` c? th? gi?m v? 0 ho?c t?ng kh?ng ??u; agent s? m?t tr? nh? ng?u nhi?n.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

L?i ??u ti?n khi deploy l? `/health` tr? 200 nh?ng `/ready` tr? 503 v?i `{"status":"not ready","redis":false}`. T?i ki?m tra Environment c?a web service tr?n Render v? th?y kh?ng c? `REDIS_URL`. Nguy?n nh?n l? t?i ?? t?o Web Service th? c?ng n?n ch? c? service web, ch?a c? Redis ?? cung c?p state. T?i t?o Render Blueprint t? `render.yaml`; Blueprint t?o hai service `day12-agent` v? `day12-redis`, r?i t? n?i `REDIS_URL` t? Redis service. Sau khi deploy l?i, `/ready` tr? 200 v?i `redis: true` v? `/ask` ho?t ??ng v?i API key.
