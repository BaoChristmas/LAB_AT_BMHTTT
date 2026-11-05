# LAB 10 – REMOTE DNS CACHE POISONING TRONG MÔI TRƯỜNG PFSENSE CÔ LẬP

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin  
**Video thực hành:** https://youtu.be/aeYt8iak5fo

## 2. Môi trường và công cụ thực hành

### 2.1. Môi trường ảo hóa

- **Nền tảng:** Oracle VM VirtualBox
- **Tường lửa và định tuyến:** pfSense 2.7.2
- **Máy chủ DNS & Trình phân giải:** Ubuntu Server 24.04 LTS (BIND 9.18 & resolver lab)
- **Máy trạm kiểm thử (Victim):** Ubuntu Server 24.04 LTS
- **Máy chủ Authoritative DNS & Web hợp lệ:** Ubuntu Server 24.04 LTS trong phân vùng DMZ
- **Máy tấn công từ xa (Attacker):** Kali Linux
- **Phân vùng mạng DNS-LAB:** Internal Network `dns-lab` (`10.9.0.0/24`)
- **Phân vùng mạng ATTACK-NET:** Internal Network `attack-net` (`10.10.0.0/24`)
- **Phân vùng mạng DMZ:** Internal Network `lab6-dmz` (`172.16.6.0/24`)

Toàn bộ kịch bản sử dụng vùng tên miền thử nghiệm cô lập `example.lab10.test` theo chuẩn RFC 2606.

### 2.2. Các máy trong mô hình

| Thành phần | Địa chỉ IP | Vai trò | Kế thừa / Mới |
|---|---|---|---|
| pfSense DNSLAB | `10.9.0.1/24` | Gateway phân vùng DNS-LAB | Kế thừa LAB trước |
| DNS-SERVER | `10.9.0.10/24` | BIND9 baseline & Resolver mô phỏng | Kế thừa LAB trước |
| VICTIM | `10.9.0.100/24` | Máy trạm nạn nhân truy vấn DNS | Kế thừa LAB trước |
| pfSense ATTACKNET | `10.10.0.1/24` | Gateway phân vùng Attacker | Mới bổ sung |
| ATTACKER Kali | `10.10.0.200/24` | Remote Attacker, Evil DNS & Fake Web | Kali chuyển mạng |
| AUTH-DNS / RELAY | `172.16.6.20/24` | Authoritative DNS hợp lệ & Race Relay | Kế thừa APP-TEST |
| DMZ-WEB | `172.16.6.10/24` | Máy chủ Web DMZ hợp lệ | Kế thừa LAB 7/8 |
| pfSense MGMT | `192.168.56.254` | Quản trị pfSense WebGUI | Kế thừa |

### 2.3. Công cụ sử dụng

- **BIND 9.18:** máy chủ DNS phục vụ cấu hình baseline và pha phòng thủ (Hardening)
- **vuln_resolver_lab10.py:** resolver mô phỏng có chủ đích (cố định source port 33333)
- **auth_dns_relay_lab10.py:** authoritative DNS zone `example.lab10.test` và giao diện race relay cổng 5300
- **evil_dns_lab10.py & fake_index.html:** máy chủ DNS và Web giả mạo trên Kali
- **kaminsky_remote_lab10.py:** kịch bản kích hoạt truy vấn ngẫu nhiên và gửi batch Transaction ID
- **pfSense Firewall:** thiết lập luật kiểm soát phân vùng cô lập giữa DNSLAB, ATTACKNET và DMZ
- **Công cụ kiểm tra & bắt gói:** `tcpdump`, `Wireshark`, `resolvectl`, `dig`, `curl`, `ip`, `systemctl`

## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Thiết lập phân vùng ATTACKNET và chính sách tường lửa pfSense

- Bổ sung Adapter 6 trên pfSense nối vào Internal Network `attack-net`
- Gán interface mới với mô tả `ATTACKNET`, đặt IPv4 tĩnh `10.10.0.1/24`, không gán Upstream Gateway
- Thiết lập ma trận luật tường lửa (Firewall Rules):
  - Cho phép VICTIM truy vấn DNS-SERVER cổng 53
  - Cho phép DNS-SERVER liên lạc với AUTH-DNS `172.16.6.20:53`
  - Cho phép DNS-SERVER truy vấn Evil DNS `10.10.0.200:53` sau khi bị đầu độc
  - Cho phép VICTIM truy cập HTTP cổng 80 tới máy chủ DMZ và máy chủ Web của Kali
  - Cho phép Kali kích hoạt truy vấn tới `10.9.0.10:53` và gửi batch tới `172.16.6.20:5300`
  - Chặn toàn bộ các luồng truy cập trái phép tới mạng LAN, MGMT và Internet

**Kết quả:** pfSense cô lập hoàn toàn phân vùng kẻ tấn công khỏi phân vùng máy chủ DNS nội bộ.

### 3.2. Chuyển Kali khỏi DNSLAB và xác nhận tính Remote

- Tháo card mạng `dns-lab` của Kali, gắn card `attack-net`
- Thiết lập chế độ mạng `Promiscuous Mode = Deny`
- Cấu hình địa chỉ IPv4 `10.10.0.200/24` và gateway `10.10.0.1`
- Chạy `tcpdump -ni any udp port 53` trên Kali trong lúc VICTIM truy vấn DNS

**Kết quả:** Kali hoàn toàn không bắt được bất kỳ gói tin nào trao đổi giữa DNS-SERVER và AUTH-DNS. Chứng minh attacker không thể nghe lén (sniff) Transaction ID, đảm bảo điều kiện bài toán Remote Attack.

### 3.3. Kiểm tra Baseline hệ thống hợp lệ

- Cấu hình BIND9 trên DNS-SERVER với vùng chuyển tiếp `example.lab10.test` tới `172.16.6.20`
- Khởi chạy dịch vụ authoritative DNS trên máy `172.16.6.20`
- Từ VICTIM (`10.9.0.100`), thực hiện xóa bộ nhớ đệm và truy vấn:

```bash
sudo resolvectl flush-caches
resolvectl query www.example.lab10.test
curl -i http://www.example.lab10.test
```

**Kết quả:** Tên miền phân giải chính xác về `172.16.6.10`, lệnh `curl` nhận phản hồi `HTTP/1.1 200 OK` từ trang web hợp lệ của DMZ-WEB.

### 3.4. Kích hoạt Resolver mô phỏng trên DNS-SERVER

- Tạm dừng dịch vụ BIND9: `sudo systemctl stop bind9`
- Chạy trình phân giải mô phỏng `vuln_resolver_lab10.py` lắng nghe tại `10.9.0.10:53`
- Resolver được thiết kế với lỗ hổng cố định cổng nguồn upstream phát đi là 33333 nhằm tái hiện lại điều kiện thực tế của các hệ thống resolver cũ trước bản vá Kaminsky



Cơ chế vận hành:

- Gửi truy vấn tên miền tiền tố ngẫu nhiên (ví dụ `r7bcxknta7n.example.lab10.test`) tới DNS-SERVER nhằm vượt qua bộ nhớ đệm
- DNS-SERVER chuyển tiếp truy vấn lên AUTH-DNS với cổng nguồn 33333 và Transaction ID 16-bit ngẫu nhiên
- AUTH-DNS ghi nhận log `[PENDING]` và mở cửa sổ trễ 3 giây
- Kali phát các batch Transaction ID (mỗi batch đại diện 4096 giá trị) tới cổng 5300
- Relay xác định batch chứa Transaction ID trùng khớp, phát gói phản hồi giả mạo kèm log `[HIT]`
- Resolver mô phỏng nhận phản hồi giả mạo trước khi gói hợp lệ đến, ghi nhận log `[CACHE] delegation poisoned`, lưu bản ghi NS `ns.evil.example.lab10.test` trỏ về IP `10.10.0.200`

**Kết quả:** Script trên Kali xác nhận thành công:

Verify www.example.lab10.test A = [‘10.10.0.200’]
SUCCESS: Cache poisoned successfully!


### 3.7. Kiểm tra hậu quả chuyển hướng từ máy trạm VICTIM

Xóa bộ nhớ đệm cục bộ trên VICTIM và truy vấn lại:

```bash
resolvectl query www.example.lab10.test
curl -i http://www.example.lab10.test
```

**Kết quả:**

- Địa chỉ IP phân giải của `www.example.lab10.test` bị chuyển hướng hoàn toàn sang IP của kẻ tấn công (`10.10.0.200`)
- Trình duyệt và lệnh `curl` nhận nội dung từ trang web giả mạo "LAB 10 - Fake Site" trên Kali thay vì DMZ-WEB hợp lệ

### 3.8. Khôi phục máy chủ BIND9 và cấu hình Hardening

- Dừng resolver mô phỏng trên DNS-SERVER (`Ctrl + C`)
- Khởi động lại dịch vụ BIND9: `sudo systemctl start bind9` — BIND9 hoạt động ổn định ở trạng thái `active (running)`
- Xác nhận các giải pháp phòng thủ tích hợp sẵn trên BIND9 hiện đại:
  - **Source-port Randomization:** mỗi truy vấn đệ quy được gán một cổng UDP nguồn ngẫu nhiên (thay vì cố định 33333), nâng không gian tìm kiếm lên 2^32, vô hiệu hóa kỹ thuật đoán vét cạn
  - **Recursion ACL:** giới hạn `allow-recursion` chỉ cho phép dải mạng nội bộ `10.9.0.0/24`, chặn kẻ tấn công bên ngoài lợi dụng làm bàn đạp kích hoạt truy vấn
  - **DNSSEC Validation:** tự động kiểm tra tính xác thực và toàn vẹn của dữ liệu phân giải bằng chữ ký số

## 4. Nội dung lý thuyết và phân tích

Báo cáo trình bày chi tiết câu trả lời cho 12 câu hỏi chuyên sâu:

1. Lý do buộc Kali rời DNSLAB và cấu hình Promiscuous Mode = Deny
2. Những tham số thông tin kẻ tấn công bị thiếu khi thực hiện Remote Poisoning so với Local Spoofing
3. Công thức tính xác suất đoán trúng Transaction ID 16-bit theo kích thước batch K
4. Cơ chế tăng không gian tìm kiếm khi kết hợp Source-port Randomization và Transaction ID
5. Vai trò của việc sinh tên miền phụ ngẫu nhiên nhằm vô hiệu hóa bộ nhớ đệm (Cache Effect)
6. Tác động của bản ghi NS và Glue A độc hại trong việc chiếm quyền phân giải toàn bộ zone
7. Quy tắc Bailiwick và nguyên lý loại bỏ các bản ghi Additional nằm ngoài phạm vi thẩm quyền
8. Mục đích sử dụng không gian tên miền `.test` theo tiêu chuẩn RFC 2606
9. So sánh vai trò của DNSSEC Validation, Recursion ACL và Firewall Segmentation
10. Bản chất thử nghiệm của Race Relay và lý do không thể áp dụng nguyên bản trên Internet thực tế
11. Rủi ro an ninh khi để mở quyền truy vấn đệ quy (Open Resolver) cho mạng không tin cậy
12. Bốn giải pháp cốt lõi để gia cố (Hardening) hệ thống DNS Resolver trong môi trường sản xuất

## 5. Tổng hợp kết quả

Toàn bộ quá trình thực hành được ghi nhận minh chứng đầy đủ qua ảnh chụp và phân tích nhật ký:

- Tách phân vùng mạng và thiết lập chính sách định tuyến, cô lập nghiêm ngặt bằng pfSense
- Chứng minh tính chất tấn công từ xa thông qua kiểm thử bắt gói không thấy lưu lượng DNS nội bộ
- Thiết lập trạng thái phân giải chuẩn (Baseline) trỏ về đúng dịch vụ DMZ
- Thực hiện thành công kịch bản tấn công Kaminsky mô phỏng thông qua cơ chế chạy đua Transaction ID
- Ghi nhận đầy đủ chuỗi nhật ký nghiệp vụ: `[PENDING]` trên AUTH-DNS, `[HIT]` trên Relay, và `[CACHE] delegation poisoned` trên Resolver
- Kiểm chứng thành công hành vi chiếm quyền điều hướng truy cập web trên máy trạm nạn nhân
- Khôi phục máy chủ BIND9 và phân tích các giải pháp Hardening phòng vệ thực tế
- Trình bày toàn bộ dữ liệu đối chiếu, cấu hình và nội dung trả lời câu hỏi trong báo cáo Word hoàn chỉnh
