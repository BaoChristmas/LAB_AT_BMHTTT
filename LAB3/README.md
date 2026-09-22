# LAB 3 - Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

- Họ và tên: Ngô Gia Bảo
- MSSV: 1150080003
- Lớp: 11_THMT
- Môn học: An toàn và bảo mật hệ thống thông tin

## Môi trường thực hành

- Hệ điều hành: Windows 11 25H2 x64
- VMware Workstation Pro: 26H1
- Microsoft Defender Antivirus
- PowerShell 5.1
- Sysmon 15.22
- Autoruns 14.3
- Process Explorer 17.14
- Wireshark 4.6.8
- Python 3.14.7

## Nội dung thực hiện

- TH1: Xác định tài sản, lỗ hổng, mối đe dọa và rủi ro
- TH2: Kiểm chứng Microsoft Defender bằng EICAR
- TH3: Phân tích sự kiện xác thực
- TH4: Phát hiện persistence và listener cục bộ
- TH5: So sánh HTTP và HTTPS
- TH6: Phân tích DoS, DDoS và Mail Bombing
- TH7: Phân tích Social Engineering và Phishing
- Cleanup, phục hồi và kiểm tra tính toàn vẹn bằng SHA-256

## Kết quả

- [x] Tạo repository và cấu trúc báo cáo
- [x] Chuẩn bị bộ dữ liệu lab3_assets
- [x] Chuẩn bị Sysmon 15.22, Autoruns 14.3, Process Explorer 17.14 và Wireshark 4.6.8
- [x] Hoàn thành risk register và phân loại nguồn đe dọa
- [ ] Hoàn thành Windows 11 25H2 VM
- [ ] Thu thập đầy đủ bằng chứng TH2–TH7
- [ ] Cleanup, verify và tạo evidence_sha256.csv

## Lỗi gặp phải và cách khắc phục

Máy ban đầu chỉ có VM Kali Linux và Windows Server 2025, không phù hợp với môi trường Windows 11 25H2 của LAB 3. Không sử dụng ảnh hoặc kết quả từ hệ điều hành sai yêu cầu. Cần dựng lại Windows 11 VM trên VMware Workstation Pro, cấu hình Host-only và tạo snapshot sạch trước khi tiếp tục.
