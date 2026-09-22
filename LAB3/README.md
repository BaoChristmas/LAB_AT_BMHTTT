# LAB 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 1. Thông tin sinh viên

- Họ và tên: Ngô Gia Bảo
- MSSV: 1150080003
- Lớp: 11_THMT
- Môn học: An toàn và bảo mật hệ thống thông tin

## 2. Mục tiêu

Bài LAB3 thực hành nhận diện, phân tích và ứng phó một số mối đe dọa phổ biến như mã độc, tấn công mật khẩu, persistence, listener, truyền dữ liệu qua HTTP, DoS/DDoS, Mail Bombing, Social Engineering và Phishing.

Toàn bộ nội dung thử nghiệm được thực hiện trong máy ảo nhằm hạn chế ảnh hưởng đến máy thật.

## 3. Môi trường thực hành

- Phần mềm ảo hóa: VMware Workstation
- Hệ điều hành: Windows 11 Pro 25H2 x64
- CPU máy ảo: 2 lõi
- RAM máy ảo: 6 GB
- Ổ đĩa máy ảo: 80 GB
- Python: 3.14.7
- Wireshark: 4.6.8
- Npcap: 1.89
- Sysmon: 15.22
- Autoruns: 14.3
- Process Explorer: 17.14
- Cấu hình mạng chính: Host-only
- Snapshot an toàn: `LAB3_CLEAN_BEFORE_TESTS`

Mạng NAT chỉ được sử dụng tạm thời để tải công cụ cần thiết. Máy ảo được chuyển lại chế độ Host-only khi thực hiện thử nghiệm.

## 4. Nội dung đã thực hiện

### 4.1. Chuẩn bị môi trường

- Cài đặt Windows 11 trên VMware Workstation.
- Cài đặt VMware Tools và các công cụ phục vụ phân tích.
- Sao chép bộ dữ liệu `lab3_assets` vào máy ảo.
- Tạo snapshot trước khi thực hiện thử nghiệm.

### 4.2. Kiểm tra mã độc bằng Microsoft Defender

- Tạo tệp kiểm thử tiêu chuẩn EICAR.
- Microsoft Defender phát hiện `Virus:DOS/EICAR_Test_File`.
- Tệp được cách ly với trạng thái `Quarantined`.
- Không sử dụng mã độc thật trong bài thực hành.

### 4.3. Tấn công mật khẩu và nhật ký xác thực

- Tạo tài khoản thử nghiệm `lab3user`.
- Thực hiện đăng nhập bằng mật khẩu đúng và mật khẩu sai.
- Kiểm tra các sự kiện Security Log:
  - Event ID 4624: đăng nhập thành công.
  - Event ID 4625: đăng nhập thất bại.
  - Event ID 4648: sử dụng thông tin xác thực rõ ràng.
- Thay đổi mật khẩu yếu sang mật khẩu phức tạp hơn.

### 4.4. Persistence và listener

- Tạo mục Registry `LAB3_Run_Demo` để mô phỏng persistence lành tính.
- Sử dụng Autoruns để phát hiện mục tự khởi động.
- Tạo HTTP listener cục bộ tại `127.0.0.1:8080`.
- Xác định tiến trình Python đang lắng nghe trên cổng 8080.

### 4.5. Phân tích HTTP bằng Wireshark

- Cài đặt Npcap và sử dụng giao diện loopback.
- Gửi yêu cầu HTTP GET đến listener cục bộ.
- Wireshark ghi nhận được đường dẫn và tham số truyền qua HTTP.
- Kết quả cho thấy HTTP không mã hóa nội dung ứng dụng, trong khi HTTPS sử dụng TLS để bảo vệ dữ liệu.

## 5. Kết quả chính

- Microsoft Defender phát hiện và cách ly thành công mẫu thử EICAR.
- Windows Security Log ghi nhận được các lần xác thực thành công và thất bại.
- Autoruns hỗ trợ phát hiện cơ chế persistence qua Registry.
- Listener cục bộ được xác định thông qua cổng và tiến trình.
- Wireshark quan sát được nội dung yêu cầu HTTP ở dạng rõ.
- Môi trường máy thật không bị sử dụng để chạy các tình huống thử nghiệm.


## 6. Video minh chứng

- [Video thực hành LAB3](https://youtu.be/GfVfysl_PD4)


## 7. An toàn và khôi phục

- Các thao tác được thực hiện trong máy ảo Windows 11.
- Không sử dụng mã độc thật.
- EICAR chỉ là chuỗi kiểm thử antivirus tiêu chuẩn.
- Listener chỉ liên kết với địa chỉ loopback `127.0.0.1`.
- Máy ảo đã được tạo snapshot trước khi thử nghiệm.
- Các tài khoản, tiến trình và mục persistence thử nghiệm được dọn dẹp sau khi hoàn thành.
