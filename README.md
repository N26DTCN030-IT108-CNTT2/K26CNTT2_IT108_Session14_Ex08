# K26CNTT2_IT108_Session14_Ex08

# 🔒 Báo cáo Bài tập: Kiểm Tra Lỗ Hổng Bảo Mật & Rò Rỉ Dữ Liệu Dự Án CampusPay
**Môn học:** Nhập Môn Công Nghệ Thông Tin (Buổi 15: Không gian cộng tác đám mây)  
**Học viên:** Lê Hồ Thanh Thảo (N26DTCN030)  
**Repository:** `K26CNTT2_IT108_Session14_Ex08`

---

## 1. Mục tiêu
- Phát hiện chính xác lỗi cấu hình quyền chia sẻ (`Permissions`) gây rò rỉ dữ liệu ngoài ý muốn trên không gian đám mây.
- Hiểu rõ sự khác biệt giữa các mức quyền (`Viewer`, `Commenter`, `Editor`) và các chế độ chia sẻ (`Restricted`, `Anyone with the link`).

---

## 2. Bối cảnh & Vấn đề
> **Tình huống:** Ứng dụng ví điện tử **"CampusPay"** vừa ra mắt bản Beta cho 200 sinh viên PTIT. Một tuần sau, thông tin nhạy cảm của người dùng (họ tên, SĐT, số dư tài khoản) bị phát tán công khai lên một nhóm Facebook. Nguồn rò rỉ được xác định xuất phát từ cấu hình sai lệch trên Google Drive của nhóm.

---

## 3. Yêu cầu đầu ra

### Phần 1: Phân tích lỗ hổng

#### 1. Xác định nguồn gốc trực tiếp gây rò rỉ dữ liệu khách hàng
* **Dòng log gây rò rỉ trực tiếp:** Dòng thứ nhất lúc **08:15 T2** (`campuspay_beta_users.xlsx` | Thành viên A | `Editor` | `Anyone with the link`).
* **Giải thích kỹ thuật:** 
  * Đây là sự vi phạm nghiêm trọng **"Quy tắc Cờ Đỏ Bảo mật"** (`Editor` kết hợp với `Anyone with the link`). 
  * Tổ hợp này biến tệp tin chứa thông tin cá nhân khách hàng thành một tài liệu công cộng mà bất kỳ ai trên Internet có đường dẫn đều xem được, tải được và thậm chí phát tán lên mạng xã hội.

#### 2. Ba giả thuyết về việc "Email người lạ" truy cập vào `campuspay_api_keys.txt` (Dòng 10:00 T3)
Mặc dù file này được cấu hình `Restricted` cho email B, người lạ vẫn có thể truy cập qua các kịch bản sau:
1. **Giả thuyết 1 (Chia sẻ tiếp / Reshare):** Email B (hoặc Trưởng nhóm) sau khi nhận quyền `Editor` hoặc quyền chia sẻ đã vô tình hoặc chủ ý tạo thêm một đường liên kết chia sẻ công khai (`Anyone with the link`) cho người khác hoặc chuyển tiếp tệp cho người ngoài.
2. **Giả thuyết 2 (Lộ tài khoản / Session Hijacking):** Tài khoản Google của Email B bị lộ mật khẩu hoặc bị chiếm quyền kiểm soát, cho phép kẻ lạ đăng nhập vào phiên làm việc và truy cập vào các tài nguyên được cấp phép riêng.
3. **Giả thuyết 3 (Lỗi phân quyền thư mục cha / Inherited Permissions):** Tệp `campuspay_api_keys.txt` được đặt bên trong một thư mục gốc đang bị cài đặt chế độ chia sẻ công khai (ví dụ: thư mục cha để `Anyone with the link`), dẫn đến việc quyền hạn bị kế thừa (`Inheritance`) xuống các file con bên trong.

---

### Phần 2: Sửa lỗi & Nghiệm thu

#### 3. Bảng cấu hình quyền chia sẻ đúng chuẩn (Secure Permissions)

| Tên file | Quyền đúng (`Role`) | Phạm vi đúng (`Scope`) | Giải thích lý do lựa chọn |
| :--- | :--- | :--- | :--- |
| **`campuspay_beta_users.xlsx`** | `Viewer` (hoặc `Restricted`) | `Restricted` (Chỉ thêm email nội bộ nhóm QA/Dev) | Chứa dữ liệu cá nhân nhạy cảm của khách hàng, tuyệt đối không bao giờ được để lộ ra ngoài hoặc cho phép chỉnh sửa công khai. |
| **`campuspay_api_keys.txt`** | `Viewer` | `Restricted` (Chỉ cấp cho Dev phụ trách hệ thống) | Chứa khóa API bảo mật tối quan trọng, cần cô lập hoàn toàn khỏi những thành viên không có nhiệm vụ. |
| **`slide_pitch_v8.pptx`** | `Commenter` | `Restricted` (Hoặc Share nội bộ trường/nhóm) | Tài liệu thuyết trình cần lấy ý kiến đóng góp, chỉ nên cấp quyền nhận xét để tránh bị xóa hoặc sửa nội dung slide thuyết trình. |

#### 4. Xử lý "Bẫy dữ liệu" & Vô hiệu hóa liên kết cũ
* **Tính năng Google Drive giúp vô hiệu hóa link cũ nhanh nhất:**  
  * **"Change link access" / "Reset Link"** hoặc chuyển đổi chế độ chia sẻ từ *Anyone with the link* về lại ***Restricted***. Hành động này lập tức làm mất hiệu lực toàn bộ các đường dẫn cũ đang trôi nổi trên không gian mạng.
* **Khi nào nên dùng tính năng "Set Expiration Date" (Đặt thời hạn truy cập)?**
  * Nên sử dụng khi làm việc với các **đối tác bên thứ ba, cộng tác viên ngắn hạn, hoặc khi gửi tài liệu đánh giá thử nghiệm (Beta testing)** trong một khoảng thời gian giới hạn (ví dụ: hết tuần pitch). 
  * Khi hết hạn, hệ thống tự động thu hồi quyền mà không cần thủ công nhớ lại để xóa, giúp bịt kín các lỗ hổng quên thu hồi quyền sau khi kết thúc công việc.
