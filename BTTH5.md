# Phần 1. Thiết kế kiến trúc: Sơ đồ luồng dữ liệu 3 lớp bảo vệ (MFA)

Để bảo vệ ví tiền điện tử — nơi chứa các tài sản vô danh, có tính thanh khoản cực cao và không thể hoàn tác giao dịch — hệ thống được thiết lập mô hình Xác thực Đa yếu tố (MFA) với 3 chốt chặn phối hợp chặt chẽ.

**Luồng dữ liệu phối hợp 3 lớp bảo vệ:**

1.  **Lớp 1: Điều bạn biết (Something you know) - Mật khẩu Master/Mã PIN**
    *   *Quy trình:* Người dùng nhập Tên đăng nhập và Mật khẩu. Trình duyệt/App gửi dữ liệu mã hóa lên Server để đối chiếu.
    *   *Kịch bản bị tấn công:* Hacker lấy được mật khẩu qua mã độc Infostealer hoặc Phishing. Chúng gõ đúng mật khẩu và vượt qua Lớp 1.
2.  **Lớp 2: Điều bạn có (Something you have) - Thuật toán TOTP (Authenticator App)**
    *   *Quy trình:* Ngay khi Lớp 1 hợp lệ, Server tạm giữ phiên (Pending Session) và yêu cầu nhập mã 6 số TOTP. Mã này liên tục thay đổi sau mỗi 30 giây.
    *   *Kịch bản chặn đứng:* Hacker bị chặn đứng hoàn toàn tại Lớp 2. Dù cầm trong tay mật khẩu đúng, chúng không sở hữu chiếc điện thoại vật lý chứa Khóa bí mật (Shared Secret) của nạn nhân nên không thể sinh ra mã 6 số chính xác.
3.  **Lớp 3: Yếu tố thuộc về bạn (Something you are) - Sinh trắc học (Biometrics)**
    *   *Quy trình:* Để mở được ứng dụng Authenticator (lấy mã TOTP) hoặc mở app Crypto Wallet trên điện thoại, hệ thống yêu cầu quét FaceID hoặc Dấu vân tay.
    *   *Kịch bản chặn đứng:* Kể cả khi hacker trộm cắp được điện thoại vật lý của nạn nhân (có Lớp 2), chúng vẫn bất lực vì không có khuôn mặt hoặc vân tay sinh học chính chủ để trích xuất mã.

**=> Kết luận kiến trúc:** Sự phối hợp tuần tự **[Mật khẩu] ➔ [Thiết bị sinh mã] ➔ [Sinh trắc học]** tạo ra cấu trúc bảo vệ bọc thép. Hacker bắt buộc phải đánh cắp và thỏa mãn đồng thời cả 3 lớp này trong một cửa sổ thời gian cực ngắn (30 giây) mới có thể mở khóa ví — một điều gần như bất khả thi trong thực tế.

---

# Phần 2. Xử lý bẫy (Edge Case): Lấy mã TOTP trên máy bay

**Tình huống:** Người dùng đang ở trên chuyến bay quốc tế, điện thoại bật chế độ máy bay (Airplane Mode), hoàn toàn không có Wi-Fi, 4G hay sóng viễn thông. Người dùng cần lấy mã OTP để đăng nhập ví và ký giao dịch (thông qua mạng vệ tinh của máy tính xách tay).

**Giải pháp và Nguyên lý hoạt động Offline của TOTP:**

*   **Bản chất thuật toán:** Thuật toán TOTP (Time-based One-Time Password) được thiết kế để hoạt động **100% ngoại tuyến (Offline)**. Ứng dụng Authenticator trên điện thoại tuyệt đối KHÔNG HỀ kết nối internet để xin mã từ Server.
*   **Cơ chế sinh mã cục bộ:** Ứng dụng Authenticator tự sinh ra mã 6 số ngay tại chip xử lý của điện thoại dựa trên công thức toán học băm (HMAC-SHA1), sử dụng đúng 2 biến số nguyên liệu:
    1.  **Khóa bí mật (Shared Secret Key):** Chuỗi ký tự đã được lưu chết vào bộ nhớ an toàn của điện thoại từ lúc bạn quét mã QR thiết lập lần đầu (khi còn ở dưới đất và có mạng).
    2.  **Thời gian hệ thống (System Timestamp):** Đồng hồ thời gian thực (Unix Time) đang chạy nội bộ trên điện thoại.
*   **Luồng xử lý vượt bẫy:** Mặc dù không có mạng, ứng dụng Authenticator vẫn lấy *Khóa bí mật* trộn với *Thời gian hiện tại của thiết bị* để tính ra kết quả 6 chữ số. Cùng lúc đó, Server của ví Crypto (dưới mặt đất) cũng tự động chạy y hệt phép toán đó. Vì cả hai bên đều dùng chung 1 Khóa bí mật và đồng bộ cùng 1 mốc thời gian thế giới, kết quả mã 6 số sinh ra sẽ trùng khớp hoàn hảo.
*   **Kết luận:** Người dùng trên máy bay chỉ cần mở điện thoại, đọc mã 6 số đang đếm lùi trên Authenticator và nhập vào máy tính. Quá trình xác thực 2FA sẽ thành công mỹ mãn mà không chịu bất kỳ cản trở nào về việc mất sóng viễn thông.