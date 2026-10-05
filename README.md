# Bài tập 4: Cấu hình Tường lửa Bảo vệ Máy chủ (UFW & Cloud Firewall Integration)

## Thông tin sinh viên
- **Họ và tên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 4

---

## Nội dung thực hiện
1. **Lớp 1: Cấu hình UFW trên Ubuntu OS:**
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow 22/tcp
   sudo ufw allow 80/tcp
   sudo ufw enable
   ```
2. **Lớp 2: Cấu hình DigitalOcean Cloud Firewall:**
   - Inbound Rules: Cho phép Port 22 (SSH) và Port 80 (HTTP).
   - Áp dụng vào Droplet: `it209-ubuntu-sgp-dev01`.
3. **Kiểm tra và xác thực:**
   - Trạng thái UFW: `sudo ufw status verbose` (Active, Port 22/80 Allowed).
   - Kiểm tra kết nối hợp lệ (Port 22/80) thành công.
   - Kiểm tra cổng đóng/chặn (Port 8080, 3306) bị Drop/Timeout.

Chi tiết xem tại: [IT209_Session02_BT4_PhamThanhPhong_N24DTCN120.md](./IT209_Session02_BT4_PhamThanhPhong_N24DTCN120.md)
