# LAB 8 – LOCAL DNS ATTACK


**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin

**Báo cáo:** [11_THMT-Lab8_1150080003-NgoGiaBao.docx](./11_THMT-Lab8_1150080003-NgoGiaBao.docx)

## 2. Nội dung báo cáo

Báo cáo phân tích DNS spoofing và DNS cache poisoning trong mô hình mạng ảo cô lập theo tài liệu LAB 8. Nội dung gồm mô hình tham chiếu, cấu hình tham khảo, tiêu chí kiểm thử, khôi phục và phòng vệ. Các sơ đồ minh họa cơ chế; các giá trị mong đợi không được trình bày như kết quả đã đo trên máy ảo.



## 3. Mô hình tham chiếu

| Thành phần | Địa chỉ theo đề | Vai trò |
|---|---|---|
| pfSense DNSLAB | `10.9.0.1/24` | Gateway và cô lập các vùng mạng |
| DNS-SERVER | `10.9.0.10/24` | BIND9, zone nội bộ và resolver có cache |
| VICTIM | `10.9.0.100/24` | Client phân giải tên miền |
| ATTACKER | `10.9.0.200/24` | Kali, quan sát DNS và web mô phỏng |
| APP-TEST / AUTH-DNS | `172.16.6.20` | DNS authoritative mô phỏng |
| DMZ-WEB | `172.16.6.10` | Website hợp lệ để đối chiếu |

VICTIM, DNS-SERVER và Kali thuộc Internal Network `dns-lab`. Các phép thử trong thiết kế chỉ nằm trong phạm vi lab và tên miền `.test`.

## 4. Các nội dung phân tích

### 4.1. Cô lập mạng và cấu hình DNS

- Ma trận rule DNSLAB: cho phép các luồng DNS/HTTP cần thiết, bảo vệ LAN-DOMAIN và MGMT.
- Phân biệt lưu lượng đi qua pfSense với lưu lượng giữa các VM cùng subnet.
- Vai trò zone authoritative, recursive resolver và cache.
- Cấu hình forward zone `external.lab9.test` tới `172.16.6.20`.
- Các phép đối chiếu baseline với địa chỉ web hợp lệ `172.16.6.10`.

### 4.2. Ba tình huống Local DNS Attack

| Tình huống | Vị trí tác động | Căn cứ cần đối chiếu |
|---|---|---|
| Thay đổi hosts file | Cấu hình phân giải trên VICTIM | Khác biệt giữa `getent`/ứng dụng và `dig` hỏi DNS trực tiếp |
| Direct DNS spoofing | Truy vấn đang chờ của VICTIM | Query, response, địa chỉ/cổng, Transaction ID và phần câu hỏi |
| DNS cache poisoning | Cache tại DNS-SERVER | Cache dump, truy vấn lặp lại, TTL và kết quả sau khi flush |

### 4.3. Khôi phục và phòng vệ

- Khôi phục hosts file và làm sạch cache ở đúng lớp.
- Thu hẹp Promiscuous Mode sau khi quan sát lưu lượng.
- Phân đoạn mạng và kiểm soát DNS egress.
- Phân biệt tác dụng của DNSSEC, DNS COOKIE và ngẫu nhiên hóa cổng/Query ID.
- Phân tích tình huống chẩn đoán tham khảo, tránh đồng nhất timeout hoặc lỗi dịch vụ với tấn công thành công.

## 5. Câu hỏi phân tích

Báo cáo trình bày đủ 10 câu hỏi của đề, bao gồm hosts file và DNS, các trường khớp truy vấn/phản hồi, phạm vi ảnh hưởng của hai kiểu giả mạo, TTL, promiscuous mode, mạng cô lập, vai trò DNS, DNSSEC, DNS mã hóa và hardening.

Tài liệu tham khảo chính thức của Ubuntu, Netgate, Oracle, ISC BIND, systemd và các RFC liên quan được liệt kê trong báo cáo Word.
