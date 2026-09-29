# LAB 4 - KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên
- Họ và tên: Ngô Gia Bảo
- MSSV: 1150080003
- Lớp: 11_THMT
- Môn học: An toàn và bảo mật hệ thống thông tin
- Link video thực hành YouTube: [https://youtu.be/XI4QvWrKKv0]

## 2. Môi trường và công cụ thực hành
- Ảo hóa: Oracle VM VirtualBox 7.x, cấu hình mạng Host-Only (192.168.56.0/24).
- Máy quét: Kali Linux 2026.x (IP: 192.168.56.103/24).
- Máy đích: Metasploitable 2 Linux (IP: 192.168.56.102/24).
- Công cụ chính: Nmap 7.99, xsltproc, Firefox ESR.

## 3. Các nội dung kỹ thuật đã hoàn thành
- Host Discovery (-sn): Quét toàn dải 192.168.56.0/24, phát hiện 5 host hoạt động.
- So sánh quét TCP (-sS vs -sT): Phân tích cơ chế bắt tay 3 bước và half-open scan; phát hiện 23 cổng TCP open trên máy đích.
- Quét UDP (--top-ports 20): Khảo sát 20 cổng UDP phổ biến, ghi nhận và phân tích hiện tượng open|filtered do cơ chế rate-limiting của Linux.
- Nhận diện dịch vụ và OS (-sV, -O): Nhận diện chính xác 5 dịch vụ trọng yếu (vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, Samba 3.X, MySQL 5.0.51a) và hệ điều hành Linux 2.6.X.
- Nmap Scripting Engine (NSE): Chạy kịch bản smb-os-discovery và smb-vuln-ms17-010 trên cổng 445; xác định hệ thống chạy Unix Samba và không bị ảnh hưởng bởi lỗ hổng MS17-010.
- Xuất hồ sơ bằng chứng: Xuất đầy đủ các định dạng .nmap, .xml, .gnmap và chuyển đổi sang giao diện web HTML bằng xsltproc.
- Tình huống Hardening: Cấu hình iptables trên máy đích drop lưu lượng cổng 21, quét đối chiếu chứng minh cổng chuyển từ open sang filtered.

## 4. Kết quả đánh giá
- Tình trạng: PASS toàn bộ các nội dung và minh chứng bắt buộc theo yêu cầu Lab 4.
