# Báo cáo Bài tập 4: Cấu hình Tường lửa Bảo vệ Máy chủ (UFW & Cloud Firewall Integration)

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 4

---

## 1. Mục tiêu & Bối cảnh bài toán

### 1.1. Mục tiêu
- Triển khai mô hình phòng thủ theo chiều sâu (**Defense in Depth**) với chiến lược **Tường lửa 2 lớp (2-Layer Firewall Architecture)** nhằm bảo vệ máy chủ đám mây khỏi các nguy cơ tấn công dò quét cổng (port scanning) và khai thác dịch vụ trái phép.
- **Lớp 1 (Host-level Firewall):** Cấu hình **UFW (Uncomplicated Firewall)** trực tiếp trong nhân Linux (iptables/nftables) trên Ubuntu Server.
- **Lớp 2 (Edge / Cloud Firewall):** Cấu hình **DigitalOcean Cloud Firewall** tại tầng biên mạng hạ tầng trước khi gói tin độc hại tiếp cận Droplet.
- Chỉ mở đúng các cổng dịch vụ cần thiết: `22/tcp` (SSH) và `80/tcp` (HTTP), chặn toàn bộ lưu lượng không xác định đi vào (Default Deny).

---

## 2. Kiến trúc Tường lửa 2 Lớp (Defense in Depth)

```
                       INTERNET (Clients / Hackers)
                                   │
                                   ▼
          ┌─────────────────────────────────────────────────┐
          │  LỚP 1: DIGITALOCEAN CLOUD FIREWALL (Edge)     │
          │  - Inbound: Allow Port 22 (SSH), Port 80 (HTTP) │
          │  - All other incoming ports: DROPPED            │
          └────────────────────────┬────────────────────────┘
                                   │ Filtered Traffic
                                   ▼
          ┌─────────────────────────────────────────────────┐
          │  LỚP 2: UBUNTU HOST FIREWALL (UFW / iptables)   │
          │  - Default Policy: DENY INCOMING, ALLOW OUTGOING│
          │  - Inbound: Allow 22/tcp, 80/tcp                │
          └────────────────────────┬────────────────────────┘
                                   │ Authorized Traffic
                                   ▼
                   ┌───────────────────────────────┐
                   │   DROPLET SERVICES (UBUNTU)   │
                   │   - SSH Daemon (Port 22)      │
                   │   - Nginx Web Server (Port 80)│
                   └───────────────────────────────┘
```

---

## 3. Quy trình thực hiện chi tiết

### Bước 1: Cấu hình Tường lửa UFW trên Ubuntu Server (Host Firewall)

Đăng nhập vào Droplet qua SSH và thực hiện lần lượt các bước cấu hình UFW:

```bash
# 1. Đặt chính sách mặc định: Chặn tất cả chiều vào, cho phép tất cả chiều ra
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. Mở cổng SSH (Port 22/tcp) - BẮT BUỘC thực hiện trước khi enable để tránh bị lock out
sudo ufw allow 22/tcp comment 'Allow SSH access'

# 3. Mở cổng HTTP (Port 80/tcp) cho Nginx Web Server
sudo ufw allow 80/tcp comment 'Allow HTTP access'

# 4. Kích hoạt UFW
sudo ufw enable
```

*Xác nhận khi terminal hỏi `Command may disrupt existing ssh connections. Proceed with operation (y|n)?`: Nhập `y` và nhấn Enter.*

---

### Bước 2: Cấu hình DigitalOcean Cloud Firewall (Infrastructure Firewall)

1. **Truy cập DigitalOcean Console:**
   - Điều hướng tới **Networking** > **Firewalls** > Nhấp chọn **Create Firewall**.
   - Đặt tên Firewall: `firewall-sgp-dev01-it209`.

2. **Thiết lập Inbound Rules (Luồng đi vào):**
   - **Rule 1 (SSH):**
     - Type: `SSH`
     - Protocol: `TCP`
     - Port Range: `22`
     - Sources: `All IPv4`, `All IPv6` (hoặc giới hạn IP máy cá nhân).
   - **Rule 2 (HTTP):**
     - Type: `HTTP`
     - Protocol: `TCP`
     - Port Range: `80`
     - Sources: `All IPv4`, `All IPv6`.

3. **Thiết lập Outbound Rules (Luồng đi ra):**
   - Giữ mặc định cho phép tất cả các luồng ra: `All TCP`, `All UDP`, `All ICMP` tới `All IPv4`, `All IPv6`.

4. **Gán Droplet vào Cloud Firewall (Apply to Droplets):**
   - Trong mục **Apply to Droplets**, tìm kiếm và chọn tên máy chủ: `it209-ubuntu-sgp-dev01`.
   - Nhấn **Create Firewall**.

---

## 4. Kiểm tra & Nhật ký Terminal (Verification Logs)

### 4.1. Kiểm tra trạng thái UFW trên máy chủ Droplet
```bash
devops@it209-ubuntu-sgp-dev01:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                   # Allow SSH access
80/tcp                     ALLOW IN    Anywhere                   # Allow HTTP access
22/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow SSH access
80/tcp (v6)                ALLOW IN    Anywhere (v6)              # Allow HTTP access
```

### 4.2. Kiểm tra kết nối dịch vụ hợp lệ từ máy cá nhân (Local Machine)

- **Kiểm tra kết nối SSH (Port 22):**
  ```bash
  PS C:\Users\ACER> ssh -i ~/.ssh/id_digitalocean_ed25519 devops@159.223.45.188 "echo 'SSH Connection OK'"
  SSH Connection OK
  ```

- **Kiểm tra truy cập HTTP (Port 80):**
  ```bash
  PS C:\Users\ACER> curl -I http://159.223.45.188
  HTTP/1.1 200 OK
  Server: nginx/1.18.0 (Ubuntu)
  ...
  ```

### 4.3. Kiểm tra cơ chế chặn các cổng không được phép (Unauthorized Ports)

Thử nghiệm kết nối tới các cổng chưa được cấp phép (ví dụ cổng MySQL `3306` hoặc cổng test `8080`):
```bash
PS C:\Users\ACER> Test-NetConnection -ComputerName 159.223.45.188 -Port 8080 -WarningAction SilentlyContinue
ComputerName     : 159.223.45.188
RemoteAddress    : 159.223.45.188
RemotePort       : 8080
TcpTestSucceeded : False
```
*Gói tin đến các cổng ngoài 22 và 80 bị Drop ngay tại Cloud Firewall và UFW, không nhận được phản hồi (Timeout), bảo vệ máy chủ an toàn tuyệt đối.*

---

## 5. Tổng kết & Đánh giá bảo mật

| Lớp bảo vệ | Vị trí thực thi | Ưu điểm & Vai trò |
| :--- | :--- | :--- |
| **DO Cloud Firewall** | Hạ tầng mạng đám mây (Edge) | Lọc bỏ gói tin tấn công/dò quét trước khi chạm đến băng thông và CPU của Droplet. Giảm tải tài nguyên phần cứng. |
| **Ubuntu UFW** | Bên trong hệ điều hành (Kernel) | Phòng thủ lớp 2 trong trường hợp cấu hình Cloud Firewall bị tắt hoặc thay đổi. Kiểm soát chi tiết từng tiến trình trong OS. |
| **Default Deny Policy** | Cả 2 tầng | Tuân thủ nguyên tắc Zero Trust / Least Privilege, chỉ mở đúng dịch vụ cần phục vụ công chúng. |
