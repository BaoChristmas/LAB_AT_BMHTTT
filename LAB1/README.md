LAB1 - BẮT GÓI TIN TELNET VÀ SSH
1. Thông tin sinh viên
Họ và tên: Ngô Gia Bảo
Mã số sinh viên: 1150080003
Lớp: 11_THMT
Tên bài Lab: Bắt gói tin Telnet - SSH

2. Mục tiêu bài thực hành
Thiết lập môi trường Client - Server phục vụ thực hành.
Sử dụng Wireshark để bắt và phân tích lưu lượng Telnet.
Kiểm tra ảnh hưởng của độ phức tạp mật khẩu đối với Telnet.
Sử dụng Wireshark để bắt và phân tích lưu lượng SSH.
So sánh khả năng bảo mật của Telnet và SSH.
Tìm hiểu phương pháp xác thực SSH bằng khóa công khai.

3. Môi trường thực hành
Máy thật: Windows 11, đóng vai trò Client.
Máy ảo: Kali GNU/Linux chạy trên Oracle VirtualBox, đóng vai trò Server.
Địa chỉ IP Server: 192.168.56.101
Công cụ sử dụng: Wireshark, PuTTY, PuTTYgen.
Dịch vụ: Telnet cổng 23 và SSH cổng 22.

4. Nội dung đã thực hiện

4.1. Thiết lập môi trường
Cấu hình hai card mạng NAT và Host-only cho máy ảo Kali.
Cấu hình địa chỉ IP 192.168.56.101 cho mạng Host-only.
Cài đặt và khởi động dịch vụ Telnet và SSH.
Tạo tài khoản riêng ngogiabao để thực hiện bài Lab.
Kiểm tra kết nối từ Windows đến Kali thành công.

4.2. Bắt và phân tích lưu lượng Telnet
Đăng nhập Telnet từ Windows vào Kali bằng PuTTY.
Thực hiện các lệnh whoami, pwd, mkdir và ls.
Bắt các gói tin Telnet bằng Wireshark với bộ lọc tcp.port == 23.
Phân tích phiên kết nối bằng chức năng Follow TCP Stream.
Thực hiện kiểm tra với cả mật khẩu đơn giản và mật khẩu phức tạp.

Kết quả: Nội dung phiên Telnet có thể đọc được dưới dạng văn bản rõ, bao gồm thông tin đăng nhập và câu lệnh. Việc sử dụng mật khẩu phức tạp không làm cho dữ liệu truyền qua Telnet được mã hóa.

4.3. Bắt và phân tích lưu lượng SSH
Đăng nhập SSH từ Windows vào Kali bằng PuTTY.
Kiểm tra và đối chiếu SSH host-key fingerprint giữa PuTTY và SSH Server.
Hai giá trị fingerprint hiển thị trùng khớp.
Bắt các gói tin SSH bằng Wireshark với bộ lọc tcp.port == 22.
Phân tích phiên kết nối SSH bằng chức năng Follow TCP Stream.

Kết quả: Nội dung phiên SSH đã được mã hóa nên không thể đọc trực tiếp tài khoản, mật khẩu và các câu lệnh như đối với Telnet.

4.4. Xác thực SSH bằng khóa công khai
Đã sử dụng PuTTYgen để tạo cặp khóa Ed25519.
Đã lưu khóa công khai và khóa riêng phục vụ quá trình xác thực.
Phần cài đặt public key trên Server và kiểm tra đăng nhập bằng khóa chưa hoàn tất.

5. Nhận xét và kết luận

Telnet truyền dữ liệu dưới dạng không mã hóa nên người bắt được lưu lượng mạng có thể đọc thông tin của phiên làm việc. Độ phức tạp của mật khẩu không khắc phục được điểm yếu này. Ngược lại, SSH mã hóa nội dung truyền trên mạng, giúp bảo vệ thông tin đăng nhập và câu lệnh. Vì vậy, SSH an toàn hơn và nên được sử dụng thay cho Telnet trong quản trị hệ thống từ xa.

Trạng thái thực hiện
Thiết lập môi trường thực hành.
Cấu hình Telnet và SSH Server.
Bắt và phân tích lưu lượng Telnet.
Kiểm tra Telnet với hai loại mật khẩu.
Bắt và phân tích lưu lượng SSH bằng mật khẩu.
Đối chiếu SSH host-key fingerprint.
Tạo cặp khóa Ed25519.
