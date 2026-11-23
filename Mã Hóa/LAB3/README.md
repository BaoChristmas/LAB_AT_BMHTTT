# LAB 3 – MÃ HÓA KHÓA CÔNG KHAI

## 1. Thông tin sinh viên

**Họ và tên:** Ngô Gia Bảo  
**MSSV:** 1150080003  
**Lớp:** 11_THMT  
**Môn học:** An toàn và bảo mật hệ thống thông tin  
**Video thực hành:** *https://youtu.be/HSfmPA0i71U*  

## 2. Môi trường và công cụ thực hành

- **Hệ điều hành:** Windows 11
- **Ngôn ngữ & JDK:** Java (JDK 21, `maven.compiler.release = 17`)
- **IDE:** Apache NetBeans IDE 25 (GUI Builder)
- **Quản lý dự án & thư viện:** Maven (`pom.xml`), `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11`
- **Cơ sở dữ liệu:** Microsoft SQL Server 2022 Express (instance `SQLEXPRESS`, cổng TCP `14333`, chế độ Mixed Mode) và SQL Server Management Studio (SSMS) 22
- **Cấu trúc project:** `LAB3_MaHoaKhoaCongKhai`, package bắt buộc `week_03`, lớp khởi chạy chính `week_03.frm_Menu`

> **Lưu ý bảo mật:** Không đưa mật khẩu tài khoản SQL Server hay các tệp khóa riêng thật lên GitHub.

## 3. Nội dung kỹ thuật và kết quả thực hành

### 3.1. Chuẩn bị dự án, lớp dùng chung và SQL Server

- Khởi tạo dự án Maven `LAB3_MaHoaKhoaCongKhai`, khai báo dependency `mssql-jdbc:12.8.1.jre11` trong `pom.xml`.
- Xây dựng các lớp tiện ích và kết nối: `CryptoSupport` (Base64, UTF-8, `SecureRandom`, AES-256-GCM, HKDF-SHA256, định dạng tệp khóa), `FileTextIO`, `DbConfig`, `HistoryDao`, cùng hai form `frm_KetNoiSQL` và `frm_LichSu`.
- Khởi tạo cơ sở dữ liệu `LAB3_MaHoaKhoaCongKhai` với bảng `dbo.History` ghi nhận lịch sử thao tác (`Algorithm`, `Operation` thuộc `ENCRYPT/DECRYPT/SIGN/VERIFY`, `InputLength`, `Success`, `CreatedAt`).

### 3.2. Các bài thực hành mật mã khóa công khai (Bài 3.1 – 3.5)

| Bài | Nội dung / Thuật toán | Dữ liệu kiểm thử | Kết quả đạt được |
| :--- | :--- | :--- | :--- |
| 3.1 | RSA Console (`RSADemo`, OAEP SHA-256) | `HELLO RSA` (2048 & 3072 bit) | Tạo khóa 2048 bit (`149 ms`, giới hạn `190 byte`, bản mã `344 ký tự`), 3072 bit (`322 ms`, giới hạn `318 byte`, bản mã `512 ký tự`); giải mã đúng `true`. |
| 3.2 | RSA Swing (`frm_RSA`) | `HELLO RSA` (RSA-2048) | Mã hóa/giải mã đúng; mã hóa cùng bản rõ hai lần sinh hai bản mã khác nhau nhờ tính ngẫu nhiên của OAEP. |
| 3.3 | ECC / ECIES (`secp256r1`, ECDH + HKDF + AES-GCM) | `HELLO ECC` | Mã hóa/giải mã đúng; `Save Keys`/`Load Keys` khôi phục đúng cặp khóa thử nghiệm; dùng sai khóa riêng báo lỗi `AEADBadTagException: Tag mismatch`. |
| 3.4 | Hệ mã lai RSA + AES-256-GCM (`frm_RSA_AES`) | Văn bản dài > 1.000 ký tự | Mã hóa và giải mã đúng toàn văn; sửa 1 ký tự bản mã (`Sửa 1 ký tự bản mã`) bị AES-GCM phát hiện và từ chối giải mã. |
| 3.5 | Chữ ký số ECDSA (`SHA256withECDSA`, `frm_ECC_Signature`) | `THONG DIEP A` / `THONG DIEP B` | Ký bằng khóa riêng, xác minh `THONG DIEP A` đạt **`VALID`**; sửa thành `THONG DIEP B` hoặc đổi cặp khóa báo **`INVALID`**. |

### 3.3. Menu chính (`frm_Menu`) và kiểm tra lịch sử SQL Server

- Xây dựng `frm_Menu` (`EXIT_ON_CLOSE`) điều hướng tới toàn bộ các form chức năng (`frm_RSA`, `frm_ECC`, `frm_RSA_AES`, `frm_ECC_Signature`, `frm_KetNoiSQL`, `frm_LichSu`).
- Kết nối thành công `127.0.0.1:14333` tới database `LAB3_MaHoaKhoaCongKhai`, ghi nhận đầy đủ các thao tác thành công (`Thành công = Có`) và thất bại/INVALID (`Thành công = Không`) trên `frm_LichSu` và truy vấn thống kê trong SSMS.

## 4. Hướng dẫn chạy chương trình

1. **Khởi tạo cơ sở dữ liệu (chạy 1 lần trên SSMS):** Thực thi tệp `01_Tao_CSDL.sql` trên SQL Server Express (`SQLEXPRESS`, cổng TCP `14333`), tạo tài khoản SQL có quyền `SELECT, INSERT` trên `dbo.History`.
2. **Chạy chương trình Java (NetBeans):**
   - Mở project `LAB3_MaHoaKhoaCongKhai` trong Apache NetBeans.
   - Nhấn **`Shift+F11`** (Clean and Build), sau đó nhấn **`F6`** (Run Project) để mở `frm_Menu`.
   - Riêng **Bài 3.1 Console**: nhấp chuột phải tệp `RSADemo.java` → chọn **`Run File`** (`Shift+F6`).

## 5. Kết quả đánh giá

**Trạng thái: PASS**

Hoàn thành đầy đủ các bài thực hành RSA (Console & Swing), ECC/ECIES, hệ mã lai RSA + AES-GCM, chữ ký số ECDSA, lưu/truy vấn lịch sử trên SQL Server, menu chính và báo cáo kiểm thử của **Lab 3**.
