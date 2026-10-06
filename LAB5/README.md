# LAB 5 – THIẾT LẬP MÔ HÌNH TƯỜNG LỬA PFSENSE

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo
**MSSV:** 1150080003
**Lớp:** 11_THMT
**Môn học:** An toàn và bảo mật hệ thống thông tin
**Video thực hành:** https://youtu.be/w69bo1rIbEo
## 2. Môi trường và công cụ thực hành

### 2.1. Môi trường ảo hóa

* **Nền tảng ảo hóa:** Oracle VM VirtualBox 7.1.x
* **Tường lửa:** pfSense CE 2.7.2-RELEASE (amd64)
* **Cấu hình pfSense:** 3 card mạng độc lập

| Card mạng  | Kiểu kết nối               | Dải mạng       | Địa chỉ IP      |
| ---------- | -------------------------- | -------------- | --------------- |
| WAN        | Bridged Adapter            | 192.168.1.0/24 | 192.168.1.46/24 |
| LAN        | Host-Only Adapter          | 10.0.0.0/8     | 10.0.0.1/8      |
| DMZ (OPT1) | Internal Network `dmz-net` | 172.16.0.0/16  | 172.16.0.1/16   |

WAN nhận địa chỉ IP thông qua DHCP từ mạng vật lý. LAN và DMZ được cấu hình làm các vùng mạng nội bộ độc lập.

### 2.2. Các máy trạm và máy chủ

**Management Station**

* Kết nối thông qua VirtualBox Host-Only Ethernet Adapter.
* Địa chỉ IP: `10.0.0.100/8`.
* Không cấu hình Gateway và DNS.
* Dùng để quản trị pfSense thông qua WebConfigurator bằng HTTPS trên cổng 443.

**Máy trạm kiểm thử LAN**

* Kali Linux.
* Card mạng `eth0`: `10.0.0.2/8`.
* Alias IP: `10.0.0.3/8`.
* Default Gateway: `10.0.0.1`.

**Máy chủ dịch vụ DMZ**

* Metasploitable 2 Linux.
* Card mạng `eth0`: `172.16.0.2/16`.
* Default Gateway: `172.16.0.1`.
* Web Server Apache được tích hợp sẵn và cung cấp dịch vụ HTTP trên cổng 80.

### 2.3. Công cụ sử dụng

* pfSense WebGUI.
* Diagnostics Tools: Ping, Reset States, System Logs.
* Terminal: `ping`, `curl`, `ifconfig`, `ip route`.
* Trình duyệt Web.

## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Khởi tạo và cấu hình pfSense

* Cài đặt thành công pfSense CE 2.7.2-RELEASE.
* Gán các cổng mạng thông qua Console.
* Hoàn thành đầy đủ 9 bước của Setup Wizard.
* Đồng bộ múi giờ `Asia/Ho_Chi_Minh`.
* Thiết lập DNS Upstream là `8.8.8.8`.
* Thay đổi mật khẩu tài khoản quản trị để tăng mức độ bảo mật.

### 3.2. Khai báo và cấu hình vùng DMZ

* Kích hoạt cổng mạng thứ ba `le2` và kết nối với mạng nội bộ `dmz-net`.
* Cấu hình địa chỉ IP tĩnh cho DMZ: `172.16.0.1/16`.
* Không thiết lập Upstream Gateway cho cổng DMZ.
* Kiểm tra và xác nhận trạng thái hoạt động của DMZ trên Dashboard pfSense.

### 3.3. Cấu hình Outbound NAT

* Chuyển cơ chế tạo Outbound NAT sang chế độ **Hybrid Outbound NAT rule generation**.
* Kiểm tra các Automatic Rules được pfSense tự động sinh.
* Xác nhận các subnet:

  * LAN `10.0.0.0/8`
  * DMZ `172.16.0.0/16`
* Cả hai subnet đều có thể được NAT ra Internet thông qua địa chỉ IP WAN của pfSense.

### 3.4. Chuẩn hóa Ruleset LAN và thiết lập rule nền tảng

* Vô hiệu hóa hai rule mặc định:

  * `Default allow LAN to any IPv4`
  * `Default allow LAN to any IPv6`
* Duy trì **Anti-Lockout Rule** để bảo đảm quyền truy cập quản trị pfSense.
* Tạo rule nền tảng do quản trị viên quản lý:

`Pass IPv4 | LAN subnets -> Any`

* Sau mỗi lần thay đổi chính sách, thực hiện **Reset State Table** để loại bỏ các trạng thái kết nối cũ và bảo đảm kết quả kiểm thử phản ánh đúng ruleset hiện tại.

### 3.5. Kiểm chứng rule nền tảng từ LAN

Từ máy trạm LAN `10.0.0.2`, thực hiện các kiểm thử:

* Ping Gateway `10.0.0.1`: thành công.
* Ping Internet `8.8.8.8`: thành công, **0% packet loss**.
* Truy cập Google bằng `curl`: nhận **HTTP/2 200**.

Kết quả xác nhận rule nền tảng cho phép lưu lượng từ LAN đi đến các đích cần thiết.

### 3.6. Tình huống 1 – Chặn ICMP nhưng vẫn cho phép Web/DNS

* Tạo rule `Block ICMP` và đặt phía trên rule nền tảng.
* Thực hiện kiểm thử:

  * Ping `8.8.8.8`: bị chặn hoàn toàn, **100% packet loss**.
  * Truy cập Google qua HTTPS bằng `curl`: vẫn phản hồi **HTTP/2 200**.

Kết quả chứng minh pfSense có thể lọc riêng lưu lượng ICMP trong khi vẫn cho phép các dịch vụ Web/DNS hoạt động theo chính sách đã cấu hình.

### 3.7. Tình huống 2 – Lọc gói theo địa chỉ IP nguồn

* Thiết lập chính sách chỉ cho phép duy nhất IP `10.0.0.2` truy cập Internet.
* Chặn toàn bộ các máy còn lại trong mạng LAN.
* Kiểm thử đối chứng:

  * `10.0.0.2` ping ra Internet: **0% packet loss**.
  * `10.0.0.3` ping ra Internet: **100% packet loss**.

Kết quả xác nhận khả năng kiểm soát lưu lượng dựa trên địa chỉ IP nguồn.

### 3.8. Tình huống 3 – Cô lập DMZ khỏi LAN

Thực hiện kiểm thử theo hai giai đoạn để đối chứng.

**Baseline – trước khi chặn**

* Từ DMZ `172.16.0.2` ping đến máy LAN `10.0.0.2`.
* Kết quả: **0% packet loss**.

**Sau khi thêm rule Block DMZ to LAN**

* Lưu lượng từ DMZ đến LAN bị chặn hoàn toàn: **100% packet loss**.
* Lưu lượng từ DMZ ra Internet vẫn hoạt động:

  * Ping `8.8.8.8`: **0% packet loss**.

Kết quả xác nhận DMZ được cô lập khỏi LAN nhưng vẫn duy trì khả năng truy cập Internet theo chính sách đã cấu hình.

### 3.9. Tình huống 4 – Port Forwarding từ WAN vào DMZ

* Tắt tùy chọn **Block private & bogon networks** trên WAN.
* Tạo rule NAT Port Forward:

  * Địa chỉ WAN: `192.168.1.46`
  * Cổng công khai: `8080`
  * Máy chủ đích: `172.16.0.2`
  * Cổng dịch vụ: `80`
* Thực hiện truy cập từ mạng ngoài thông qua:

`192.168.1.46:8080`

* Kết quả: truy cập thành công giao diện Web của Web Server Apache trong DMZ.

### 3.10. Tình huống 5 – Bật Logging và phân tích Firewall Log

* Kích hoạt cơ chế ghi nhật ký trên rule chặn.
* Tạo lưu lượng bị từ chối từ IP `10.0.0.3`.
* Kiểm tra log tại:

`Status > System Logs > Firewall`

* Trích xuất và phân tích thành công bản ghi sự kiện Firewall thể hiện việc pfSense chặn gói tin ICMP.

Kết quả xác nhận hệ thống có khả năng ghi nhận và truy vết các sự kiện bị Firewall từ chối.

## 4. Nội dung lý thuyết đã hoàn thành

Hoàn thành giải đáp 6 câu hỏi lý thuyết liên quan đến các nội dung:

1. Phân biệt vai trò của NAT và Firewall Rule.
2. Nguyên lý của vùng DMZ trong việc hạn chế nguy cơ tấn công theo chiều ngang (Lateral Movement).
3. Cơ chế xử lý **First-match rule wins** của Firewall.
4. Cấu hình chính sách lọc gói tin theo dịch vụ.
5. Vai trò của Firewall Log trong giám sát và phân tích sự kiện.
6. Ba biện pháp hardening tường lửa pfSense.

## 5. Kết quả đánh giá

**Trạng thái: PASS**

Hoàn thành đầy đủ các nội dung thực hành, cấu hình, kiểm thử, phân tích log và phần câu hỏi lý thuyết theo yêu cầu của **Lab 5 – Thiết lập mô hình tường lửa pfSense**.

