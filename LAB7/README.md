# LAB 7 – XÂY DỰNG CHÍNH SÁCH WEB PROTECTION VỚI PFSENSE VÀ WEB PROXY

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin  
**Video thực hành:** https://youtu.be/McXzo1L1TbY

## 2. Môi trường và công cụ thực hành

### 2.1. Môi trường ảo hóa

- **Nền tảng:** Oracle VM VirtualBox.
- **Tường lửa và định tuyến:** pfSense.
- **Máy trạm kiểm thử:** Windows 11.
- **Máy chủ Web Proxy và DMZ:** Ubuntu Server 24.04 LTS.
- **Mạng LAN:** Internal Network `lab6-lan`.
- **Mạng DMZ:** Internal Network `lab6-dmz`.

Các tên mạng `lab6-lan`, `lab6-dmz` được giữ từ môi trường thực hành trước. Website của LAB 7 sử dụng vùng DNS riêng `lab7.test`.

### 2.2. Các máy trong mô hình

| Thành phần | Địa chỉ IP | Vai trò |
|---|---|---|
| pfSense LAN | `10.6.0.1/24` | Gateway mạng LAN |
| pfSense DMZ | `172.16.6.1/24` | Gateway mạng DMZ |
| DC01 | `10.6.0.10/24` | Máy chủ DNS nội bộ |
| Client-A | `10.6.0.21/24` | Máy trạm kiểm thử chính sách Web |
| WEB-PROXY | `10.6.0.30/24` | E2Guardian, Squid và ClamAV |
| DMZ-WEB | `172.16.6.10/24` | Nginx phục vụ các website mô phỏng |

WEB-PROXY và Client-A sử dụng gateway `10.6.0.1`. DMZ-WEB sử dụng gateway `172.16.6.1`. DNS nội bộ của các máy là `10.6.0.10`.

### 2.3. Công cụ sử dụng

- **E2Guardian:** lọc website, hiển thị cảnh báo và quản lý truy cập tạm.
- **Squid:** proxy phía sau E2Guardian.
- **ClamAV:** quét nội dung và phát hiện mẫu kiểm thử antivirus.
- **Nginx:** cung cấp các website mô phỏng trong DMZ.
- **Windows DNS Server:** quản lý vùng `lab7.test`.
- **Microsoft Edge:** kiểm thử truy cập từ Client-A.
- **Công cụ kiểm tra:** `curl`, `ping`, `ip`, `ss`, `systemctl`, `getent` và PowerShell.

## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Chuẩn bị máy chủ WEB-PROXY

- Triển khai Ubuntu Server 24.04 LTS.
- Cấu hình máy ảo với 2 vCPU và 2 GB RAM.
- Đặt địa chỉ IPv4 tĩnh `10.6.0.30/24`.
- Thiết lập gateway `10.6.0.1` và DNS `10.6.0.10`.
- Kiểm tra hostname, số CPU, địa chỉ IP và bảng định tuyến.
- Kiểm tra kết nối tới gateway, DC01 và phân giải tên miền kho Ubuntu qua DNS nội bộ.

Kết quả xác nhận WEB-PROXY có kết nối mạng và phân giải DNS phục vụ cài đặt, vận hành các dịch vụ.

### 3.2. Cài đặt và cấu hình Squid

- Cài đặt Squid trên WEB-PROXY.
- Cấu hình Squid lắng nghe tại:

`127.0.0.1:3128`

- Kiểm tra cú pháp cấu hình và cổng dịch vụ.
- Gửi yêu cầu HTTP bằng `curl` qua Squid.

**Kết quả:** nhận phản hồi **HTTP 200**, có thông tin Squid trong header `Via`.

Squid chỉ lắng nghe trên loopback, phục vụ vai trò proxy phía sau E2Guardian.

### 3.3. Cấu hình E2Guardian kết nối với Squid

Thiết lập các tham số chính:

| Tham số | Giá trị |
|---|---|
| `filterip` | `10.6.0.30` |
| `filterports` | `8080` |
| `proxyip` | `127.0.0.1` |
| `proxyport` | `3128` |
| `loglevel` | `2` |

Kiểm tra E2Guardian tiếp nhận kết nối tại `10.6.0.30:8080` và chuyển yêu cầu tới Squid.

**Kết quả:** yêu cầu HTTP qua E2Guardian nhận phản hồi thành công từ proxy phía sau.

### 3.4. Khai báo danh sách chính sách website

Các tên miền dưới đây sử dụng hậu tố `.lab7.test`.

| Nhóm chính sách | Website |
|---|---|
| Chặn cứng | `china`, `mideast`, `jobs`, `blockedgame` |
| Cảnh báo và cho phép truy cập tạm sau xác nhận | `weapons`, `politics`, `religion`, `streaming` |
| Cho phép | `normal`, `education`, `download` |

Cấu hình các danh sách:

- `bannedsitelist`: chứa các website thuộc nhóm chặn và cảnh báo.
- `domainsnobypass`: chứa các website thuộc nhóm chặn cứng.
- `exceptionsitelist`: chứa các website thuộc nhóm cho phép.

Thiết lập `bypass = 300` để cấu hình thời gian truy cập tạm sau khi xác nhận cảnh báo.

### 3.5. Tùy chỉnh trang chặn

- Thay đổi tiêu đề thành **LAB 7 – Truy cập bị chặn**.
- Điều chỉnh màu nền và phần hiển thị thông báo.
- Bổ sung thông tin liên hệ quản trị phòng lab.
- Hiển thị website được yêu cầu, lý do chặn và địa chỉ IP client.

Kết quả giúp người dùng nhận biết nguyên nhân truy cập bị từ chối và đầu mối hỗ trợ.

### 3.6. Triển khai website mô phỏng trên DMZ-WEB

- Đặt IP tĩnh `172.16.6.10/24`.
- Cấu hình gateway `172.16.6.1` và DNS `10.6.0.10`.
- Cài đặt Nginx.
- Tạo 13 virtual host:

`normal`, `china`, `mideast`, `jobs`, `weapons`, `politics`, `religion`, `education`, `entertainment`, `games`, `streaming`, `blockedgame`, `download`.

Mỗi website có `server_name` và thư mục nội dung riêng trong `/var/www/lab7`.

- Kiểm tra cấu hình bằng `nginx -t`.
- Kiểm tra số lượng virtual host.
- Gửi yêu cầu HTTP cục bộ với header `Host: normal.lab7.test`.

**Kết quả:** cấu hình Nginx hợp lệ; website `normal.lab7.test` trả **HTTP 200** và đúng nội dung mô phỏng.

### 3.7. Cấu hình DNS nội bộ trên DC01

- Tạo vùng DNS `lab7.test`.
- Khai báo 13 bản ghi A tương ứng với các website.
- Các bản ghi cùng trỏ tới máy chủ DMZ-WEB `172.16.6.10`.
- Kiểm tra bằng PowerShell và công cụ phân giải tên miền trên Ubuntu.

**Kết quả:** `normal.lab7.test` phân giải thành công tới `172.16.6.10`; danh sách DNS hiển thị đủ 13 bản ghi A.

Nginx phân biệt các website dùng chung IP thông qua tên host trong yêu cầu HTTP.

### 3.8. Kiểm tra đường truy cập qua proxy

Tại WEB-PROXY, gửi yêu cầu tới `normal.lab7.test` thông qua:

`http://10.6.0.30:8080`

Luồng yêu cầu:

**E2Guardian → Squid → pfSense → DMZ-WEB**

**Kết quả:**

- Squid, E2Guardian và ClamAV ở trạng thái hoạt động.
- Phản hồi **HTTP 200**.
- Header `Via` có thông tin Squid.
- Nội dung trả về đúng website `normal.lab7.test`.

### 3.9. Client-A truy cập website được phép

- Thiết lập proxy thủ công trên Client-A:
  - **Địa chỉ:** `10.6.0.30`.
  - **Cổng:** `8080`.
- Mở Microsoft Edge và truy cập:

`http://normal.lab7.test`

**Kết quả:** trình duyệt hiển thị nội dung website mô phỏng `normal.lab7.test`.

### 3.10. Kiểm thử chặn website China

Từ Client-A, truy cập:

`http://china.lab7.test`

**Kết quả:**

- E2Guardian hiển thị trang chặn đã tùy chỉnh.
- Lý do hiển thị: `Blocked site: china.lab7.test`.
- Địa chỉ client được nhận diện là `10.6.0.21`.
- Sau khi bấm **Acknowledge**, website vẫn bị chặn.

Kết quả thể hiện hành vi từ chối truy cập đối với website được đưa vào nhóm chặn cứng.

### 3.11. Kiểm thử cảnh báo và truy cập sau xác nhận

Từ Client-A, truy cập:

`http://politics.lab7.test`

Thực hiện đối chiếu hai thời điểm:

**Trước khi xác nhận**

- Trình duyệt hiển thị trang cảnh báo.
- Có liên kết **Acknowledge** để xác nhận truy cập.

**Sau khi xác nhận**

- Bấm **Acknowledge**.
- Trình duyệt hiển thị nội dung website `politics.lab7.test`.

Hai ảnh minh chứng thể hiện sự thay đổi hành vi truy cập trước và sau khi người dùng xác nhận cảnh báo.

### 3.12. Tích hợp ClamAV và kiểm thử EICAR

- Bật plugin `clamdscan` trong E2Guardian.
- Kiểm tra kết nối tới dịch vụ ClamAV.
- Cấu hình:
  - `disablecontentscan = off`.
  - `contentscanexceptions = on`.
  - `infectionbypass = 0`.
- Sử dụng mẫu kiểm thử EICAR trên website DMZ:

`http://download.lab7.test/files/eicar.com.txt`

Thực hiện yêu cầu HTTP qua E2Guardian tại WEB-PROXY và đối chiếu nội dung phản hồi với nhật ký.

**Kết quả:**

- Nội dung phản hồi có cảnh báo:

`Virus or bad content detected. Eicar-Test-Signature`

- Nhật ký E2Guardian ghi nhận:

`*INFECTED*` và `*DENIED*`

- Bản ghi kiểm thử có mã `403` trong log; lệnh `curl` nhận HTTP `200` cho phản hồi chứa trang cảnh báo.

Kết quả xác nhận ClamAV nhận diện mẫu EICAR và E2Guardian ghi nhận sự kiện phát hiện nội dung kiểm thử antivirus.

### 3.13. Theo dõi nhật ký E2Guardian

Kiểm tra nhật ký tại:

`/var/log/e2guardian/access.log`

Đối chiếu các trường thông tin:

- Thời điểm phát sinh yêu cầu.
- Địa chỉ nguồn.
- URL được truy cập.
- Kết quả xử lý.
- Lý do lọc hoặc phát hiện AV.

Nhật ký cung cấp bằng chứng đối chiếu cho yêu cầu truy cập website normal và phép thử phát hiện EICAR.

## 4. Nội dung lý thuyết và phân tích

Báo cáo trình bày câu trả lời cho 10 câu hỏi:

1. Lý do tách Squid/SquidGuard khỏi pfSense trong kiến trúc LAB 7.
2. Sự khác nhau giữa firewall và Web Proxy.
3. Vai trò của việc kiểm soát truy cập web trực tiếp khi sử dụng proxy.
4. Hạn chế của việc phân loại website chỉ dựa trên địa chỉ IP.
5. Phân biệt temporary bypass và hard block.
6. Quan hệ giữa mã hóa HTTPS, SSL MITM và khả năng quét EICAR.
7. Rủi ro khi private key của CA phục vụ HTTPS inspection bị lộ.
8. Ý nghĩa của `maxcontentfilecachescansize`.
9. Phân biệt Schedule 60 phút và daily category quota.
10. Hướng kiểm tra khi trình duyệt báo lỗi nhưng E2Guardian không ghi nhận yêu cầu.

## 5. Tổng hợp kết quả

Các kết quả thực hành được trình bày bằng ảnh chụp và phân tích trong báo cáo:

- Xây dựng máy chủ WEB-PROXY với E2Guardian, Squid và ClamAV.
- Triển khai 13 website mô phỏng trên Nginx và DNS nội bộ.
- Kiểm tra luồng HTTP qua E2Guardian và Squid tới website DMZ.
- Kiểm thử truy cập website được phép từ Client-A.
- Kiểm thử chặn China và cảnh báo trước khi truy cập Politics.
- Tùy chỉnh giao diện trang chặn.
- Phát hiện mẫu EICAR và đối chiếu nhật ký E2Guardian.
- Trình bày mô hình, cấu hình, minh chứng và câu trả lời phân tích trong báo cáo Word.
