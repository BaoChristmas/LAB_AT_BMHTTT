# LAB 1 – MÃ HÓA ĐỐI XỨNG CỔ ĐIỂN

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin  
**Video thực hành:** *https://youtu.be/hOKxGDzWrzs*  

## 2. Môi trường và công cụ thực hành

- **Hệ điều hành:** Windows 11
- **Ngôn ngữ & JDK:** Java (JDK 21, `maven.compiler.release = 17`)
- **IDE:** Apache NetBeans IDE 25 (GUI Builder)
- **Quản lý dự án & thư viện:** Maven (`pom.xml`), `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11`
- **Cơ sở dữ liệu:** Microsoft SQL Server 2022 Express (instance `SQLEXPRESS`, cổng TCP `14333`, chế độ Mixed Mode) và SQL Server Management Studio (SSMS) 22
- **Cấu trúc project:** `LAB1_MaHoaCoDien`, package bắt buộc `week_01`, lớp khởi chạy chính `week_01.frm_Menu`

> **Lưu ý bảo mật:** Thông tin mật khẩu tài khoản SQL Server không được lưu trong mã nguồn hay tệp `README.md` công khai.

## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Chuẩn bị dự án và lớp dùng chung (`FileTextIO`)

- Khởi tạo dự án Maven `LAB1_MaHoaCoDien`, khai báo dependency `mssql-jdbc:12.8.1.jre11` và `exec.mainClass` là `week_01.frm_Menu` trong `pom.xml`.
- Xây dựng lớp `FileTextIO` hỗ trợ hộp thoại mở/lưu tệp văn bản UTF-8 (`.txt`) dùng chung cho tất cả các form.

### 3.2. Sáu thuật toán mã hóa cổ điển cơ bản (Bài 1.1 – 1.6)

Cài đặt đầy đủ 6 lớp thuật toán và 6 giao diện JFrame Form tương ứng (`frm_Caesar`, `frm_Vigenere`, `frm_RailFence`, `frm_Playfair`, `frm_Transposition`, `frm_Autokey`). Tất cả các phép thử mã hóa, giải mã, mở/lưu tệp và bắt lỗi khóa không hợp lệ đều đạt kết quả chính xác:

| Bài | Thuật toán | Plaintext | Key | Ciphertext đúng |
| :--- | :--- | :--- | :--- | :--- |
| 1.1 | Caesar | `HELLO` | `3` | `KHOOR` |
| 1.2 | Vigenère | `HELLO` | `KEY` | `RIJVS` |
| 1.3 | Rail Fence | `HELLO WORLD` | `3` | `HOREL OLLWD` |
| 1.4 | Playfair | `HELLOWORLD` | `KEYWORD` | `GYIZSCOKCFBU` |
| 1.5 | Transposition | `BAOMAT` | `2,3,1` | `AAOTBM` |
| 1.6 | Autokey | `HELLO` | `KEY` | `RIJSS` |

- **Playfair giải mã:** Khôi phục bản rõ `HELXLOWORLDX` (giữ ký tự đệm `X` đúng thuật toán).
- **Kiểm tra dữ liệu không hợp lệ:** Chương trình hiển thị hộp thoại thông báo lỗi rõ ràng và không bị văng khi nhập sai định dạng khóa.

### 3.3. Bài nâng cao 01 – Chương trình tổng hợp và lưu lịch sử SQL Server

- **`CipherTongHop` & `frm_TongHop`:** Cho phép chọn cả 6 thuật toán qua `JComboBox`, tự động gợi ý khóa mẫu khi đổi thuật toán, hỗ trợ mã hóa, giải mã và đọc/ghi tệp.
- **`DbConfig` & `HistoryDao`:** Cấu hình kết nối JDBC tới `127.0.0.1:14333`, database `LAB1_MaHoaCoDien` (`encrypt=true;trustServerCertificate=true`); sử dụng `PreparedStatement` để ghi và đọc bảng `dbo.history`, tránh lỗi SQL Injection.
- **`frm_KetNoiSQL` & `frm_LichSu`:** Kiểm tra kết nối SQL Server thành công với tài khoản thực hành `lab1_user`; mỗi thao tác mã hóa/giải mã trên `frm_TongHop` tự động lưu vào `dbo.history`, hiển thị đầy đủ 6 dòng mới nhất trên `JTable` của `frm_LichSu` và truy vấn đối chiếu thành công trong SSMS.

### 3.4. Bài nâng cao 02 – Phân tích (tấn công) mật mã cổ điển

Cài đặt lớp `AttackDemos` và giao diện `frm_TanCong` với 3 kỹ thuật phân tích và hoàn thành 3 thí nghiệm bắt buộc:

1. **Vét cạn (Brute Force) Caesar:** Nhập bản mã `KHOOR`, chương trình thử đủ 26 khóa (`k = 0..25`), chấm điểm tần suất tiếng Anh và tự đánh dấu kết luận khóa `k = 3`, bản rõ `HELLO`.
2. **Phân tích tần suất (Frequency Analysis):** Nhập bản mã `PHHW PH DIWHU WKH WRJD SDUWB` (23 chữ cái), thống kê cho thấy chữ `H` và `W` xuất hiện nhiều nhất (`5` lần, `21.7%`). Giả sử `H` là ảnh của `E`, suy ra khóa `k = H - E = 7 - 4 = 3`.
3. **Biết trước bản rõ (Known Plaintext) Vigenère:** Nhập cặp `HELLO` / `RIJVS`, suy ra chuỗi khóa từng vị trí là `KEYKE` (khóa `KEY` lặp lại theo chu kỳ).

### 3.5. Menu chính và chạy toàn bộ chương trình (`frm_Menu`)

- Xây dựng `frm_Menu` làm cửa sổ khởi động chính (`EXIT_ON_CLOSE`) với 9 nút mở đúng 9 form chức năng; các form con đặt `DISPOSE_ON_CLOSE` nên đóng form con không làm tắt chương trình.
- Thực hiện `Clean and Build` đạt `BUILD SUCCESS` và chạy toàn bộ chương trình bằng `F6` ổn định.

## 4. Nhận xét điểm yếu và cách tấn công sáu thuật toán

| Thuật toán | Điểm yếu chính | Cách tấn công |
|---|---|---|
| **Caesar** | Không gian khóa nhỏ (chỉ 26 khóa). | Vét cạn (Brute Force) đủ 26 khóa; phân tích tần suất chữ cái. |
| **Vigenère** | Khóa lặp lại theo chu kỳ cố định. | Tìm độ dài khóa (Kasiski/Friedman) rồi phân tích tần suất từng cột; tấn công Known Plaintext. |
| **Rail Fence** | Số hàng nhỏ, không đổi chữ cái mà chỉ đổi vị trí. | Thử mọi số hàng `2..n`; tần suất chữ giữ nguyên như bản rõ. |
| **Playfair** | Thay thế theo cặp chữ trên ma trận 5×5 (25 chữ). | Thống kê tần suất cặp chữ (digram); dùng Known Plaintext dựng lại ma trận khóa. |
| **Transposition** | Chỉ đổi vị trí cột, giữ nguyên tần suất chữ cái. | Thử độ dài khóa nhỏ, hoán vị sắp xếp cột để ghép ra từ có nghĩa. |
| **Autokey** | Dòng khóa chứa chính bản rõ sau phần khóa mồi. | Đoán từ thông dụng (`THE`, `AND`...) để dò dần khóa; tấn công Known Plaintext. |

## 5. Kết quả đánh giá

**Trạng thái: PASS**

Hoàn thành đầy đủ 6 thuật toán mã hóa đối xứng cổ điển, chương trình tổng hợp, tích hợp lưu/đọc lịch sử trên Microsoft SQL Server, 3 kỹ thuật phân tích/tấn công mật mã, menu điều hướng chính và các tệp báo cáo/SQL theo yêu cầu của **Lab 1 – Mã hóa đối xứng cổ điển**.
