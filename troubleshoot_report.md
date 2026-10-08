# Bài 5: Chẩn đoán và xử lý xung đột cổng mạng (Address Already in Use)

## 1. Quy trình chẩn đoán và xử lý

### Bước 1: Giả lập chiếm dụng cổng 8082
```bash
python3 -m http.server 8082 > /dev/null 2>&1 &
```

### Bước 2: Tìm tiến trình đang chiếm cổng 8082
Gõ lệnh sau để kiểm tra:
```bash
sudo ss -tlnp | grep 8082
```
*Kết quả hiển thị ví dụ:*
```text
LISTEN 0      50           0.0.0.0:8082      0.0.0.0:*    users:(("python3",pid=12345,fd=3))
```
=> Tiến trình đang chiếm cổng là `python3` với PID là **12345** (PID thực tế có thể khác).

### Bước 3: Tắt tiến trình chiếm dụng
Sử dụng lệnh `kill` để kết thúc tiến trình:
```bash
sudo kill -9 12345
```
*(Thay 12345 bằng PID thực tế tìm được ở Bước 2)*

### Bước 4: Khởi động lại dịch vụ Java và kiểm tra
```bash
sudo systemctl restart spring-app.service
sudo ss -tlnp | grep 8082
```
*Kết quả hiển thị (đã giải phóng và được java chạy)*
```text
LISTEN 0      100                *:8082            *:*    users:(("java",pid=12399,fd=8))
```

## 2. Giải trình ngắn gọn
- Khi ứng dụng báo lỗi `Address already in use`, tức là cổng mạng (port) mong muốn đang bị một tiến trình khác lắng nghe (listening).
- Lệnh `ss -tlnp` (hoặc `lsof -i`) giúp ánh xạ cổng đang mở với Process ID (PID) tương ứng.
- Lệnh `kill -9 <PID>` sẽ ép hệ điều hành chấm dứt ngay lập tức (SIGKILL) tiến trình đó, qua đó giải phóng cổng mạng để dịch vụ Spring Boot có thể sử dụng.
