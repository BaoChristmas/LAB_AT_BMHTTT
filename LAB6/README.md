# BÁO CÁO THỰC HÀNH LAB 6 - NGÀY 13/10/2026
**Đề tài:** Xác thực Active Directory và Chính sách truy cập trên pfSense

## Thông tin sinh viên
- **Họ và tên:** Ngô Gia Bảo
- **MSSV:** 1150080003
- **Lớp:** 11_THMT

---

## 1. Cấu hình hạ tầng mạng ảo hóa (VirtualBox & pfSense)

Hệ thống tường lửa pfSense được thiết lập với 4 giao diện mạng tương ứng với các phân vùng trong bài thực hành:

| Giao diện (Interface) | Cổng mạng (Port) | Chế độ kết nối (VirtualBox) | Địa chỉ IP / Subnet | Vai trò trong hệ thống |
| :--- | :--- | :--- | :--- | :--- |
| **WAN** | `le0` | NAT Network (`LAB6-WAN`) | DHCP (`192.168.250.0/24`) | Kết nối mạng ngoài cho pfSense |
| **MGMT** | `le1` | Host-only Adapter | `192.168.56.254/24` | Truy cập giao diện quản trị WebGUI từ máy thật |
| **LAN** | `le2` | Internal Network (`lab6-lan`) | `10.6.0.1/24` | Default Gateway cho mạng nội bộ Domain |
| **DMZ** | `em0` | Internal Network (`lab6-dmz`) | `172.16.6.1/24` | Default Gateway cho vùng DMZ |

---

## 2. Triển khai máy chủ Domain Controller (DC01)

- **Hệ điều hành:** Windows Server 2025 Standard Evaluation
- **Cấu hình mạng tĩnh:**
  - **IP Address:** `10.6.0.10`
  - **Subnet Mask:** `255.255.255.0`
  - **Default Gateway:** `10.6.0.1`
  - **Preferred DNS Server:** `127.0.0.1`
- **Cấu hình dịch vụ Active Directory (AD DS):**
  - Nâng cấp máy chủ thành Domain Controller quản lý miền `lab6.test`.
  - Tạo các tài khoản người dùng (`sv_a`, `sv_b`) và các nhóm (`Nhom_A`, `Nhom_B`), gán người dùng vào nhóm tương ứng thông qua PowerShell.
- **Kiểm tra kết nối mạng:**
  - Đã tắt tường lửa Windows Defender Firewall trên `DC01` và kiểm tra Ping từ `10.6.0.10` đến Gateway pfSense `10.6.0.1` thành công 100% (`Reply from 10.6.0.1: bytes=32 time<1ms TTL=64`).

---

## 3. Cấu hình tích hợp LDAP trên pfSense và Ghi nhận lỗi thực nghiệm

### 3.1. Thông số đã thiết lập (`System > User Manager > Authentication Servers`)
- **Descriptive name:** `DC01-AD`
- **Type:** `LDAP`
- **Hostname or IP address:** `10.6.0.10` | **Port:** `389` (`Standard TCP`)
- **Protocol version:** `3` | **Search scope:** `Entire Subtree`
- **Base DN:** `DC=lab6,DC=test`
- **Authentication containers:** `CN=Users,DC=lab6,DC=test`
- **Bind credentials:** `CN=Administrator,CN=Users,DC=lab6,DC=test` (và đã thử nghiệm với `Administrator@lab6.test`)
- **Naming attributes:** `sAMAccountName` / `cn` / `member`

### 3.2. Tình trạng lỗi gặp phải và các bước đã xử lý
Trong quá trình thực hiện liên kết LDAP và kiểm tra xác thực tài khoản `sv_a` tại mục **Diagnostics > Authentication**, mặc dù kết nối mạng tầng 3 (Ping `10.6.0.1`) và kiểm tra cổng dịch vụ tầng 4 (**Diagnostics > Test Port** tới `10.6.0.10:389`) đều báo thành công, hệ thống pfSense vẫn liên tục báo lỗi `Could not connect to the LDAP server` và `Authentication failed`.

**Các biện pháp khắc phục đã thực hiện lặp lại nhiều lần:**
1. Tắt toàn bộ các cấu hình tường lửa (`Domain`, `Private`, `Public`) trên Windows Server 2025.
2. Đặt lại mật khẩu đồng bộ và kích hoạt trạng thái tài khoản (`Enabled`, `PasswordNeverExpires`) cho `Administrator`, `sv_a`, `sv_b` bằng lệnh PowerShell và `net user`.
3. Can thiệp khóa Registry `LDAPServerIntegrity` trên dịch vụ `NTDS` của Domain Controller và khởi động lại dịch vụ AD DS.
4. Thay đổi luân phiên định dạng tài khoản Bind (`UPN` và `Distinguished Name`), điều chỉnh các thuộc tính truy vấn nhóm và cấu hình lại máy chủ xác thực trên pfSense nhiều lần.

**Nguyên nhân xác định:** Bản dựng Windows Server 2025 áp dụng chính sách bảo mật mặc định nghiêm ngặt đối với giao thức LDAP (yêu cầu ký số *LDAP Signing* và *Channel Binding*), dẫn đến việc từ chối các phiên kết nối xác thực dạng không mã hóa (*Simple Bind* qua cổng TCP 389) từ pfSense khiến quá trình thực hành bị kẹt tại bước xác thực người dùng.
