THIẾT KẾ CƠ SỞ DỮ LIỆU PHÂN QUYỀN AN TOÀN
==========================================

CHÚ THÍCH
---------
PK = Khóa chính: mã định danh duy nhất của một bản ghi.
FK = Khóa ngoại: liên kết đến một bảng khác.
UK = Giá trị duy nhất: không được trùng lặp.
1:N = Một đối tượng có thể liên kết với nhiều đối tượng khác.
N:M = Nhiều đối tượng có thể liên kết với nhiều đối tượng khác.


1. BẢNG NGƯỜI DÙNG (users)
--------------------------

+--------------------------------+
|             users              |
+--------------------------------+
| PK  id              UUID       |
| UK  email           TEXT       |
|     password_hash   TEXT       |
|     status          TEXT       |
|     email_verified_at          |
|     created_at                 |
|     updated_at                 |
+--------------------------------+

Chức năng:
- Lưu thông tin tài khoản người dùng.
- Lưu mật khẩu dưới dạng mã băm an toàn,
  không lưu mật khẩu nguyên bản.
- Theo dõi trạng thái tài khoản:
  đang hoạt động, bị đình chỉ hoặc bị vô hiệu hóa.
- Ghi lại thời điểm tạo và cập nhật tài khoản.


2. BẢNG PHÂN VAI TRÒ (roles)
----------------------------

+--------------------------------+
|             roles              |
+--------------------------------+
| PK  id              BIGINT     |
| UK  name            TEXT       |
|     description     TEXT       |
+--------------------------------+

Chức năng:
- Xác định các nhóm quyền trong website.
- Ví dụ:
  + user: người dùng thông thường.
  + moderator: người kiểm duyệt nội dung.
  + admin: quản trị viên.


3. BẢNG GÁN VAI TRÒ CHO NGƯỜI DÙNG
----------------------------------

+-------------------+
|    user_roles     |
+-------------------+
| PK,FK user_id     |
| PK,FK role_id     |
|     granted_at    |
| FK  granted_by    |
+-------------------+

Chức năng:
- Xác định mỗi người dùng được giao vai trò nào.
- Một người có thể có nhiều vai trò.
- Một vai trò có thể được giao cho nhiều người.
- Ghi nhận thời điểm và người thực hiện việc cấp quyền.

Ví dụ:
- Người A có vai trò user.
- Người B có vai trò moderator.
- Người C có vai trò admin.

Lưu ý:
Việc cấp quyền quản trị phải được kiểm soát chặt chẽ.


4. BẢNG QUYỀN HẠN (permissions)
-------------------------------

+--------------------------------+
|          permissions           |
+--------------------------------+
| PK  id              BIGINT     |
| UK  name            TEXT       |
|     description     TEXT       |
+--------------------------------+

Chức năng:
- Xác định những hành động cụ thể được phép thực hiện.
- Ví dụ:
  + profile.read: xem hồ sơ được phép truy cập.
  + profile.update.self: sửa hồ sơ của chính mình.
  + content.create: tạo nội dung.
  + content.moderate: kiểm duyệt nội dung.
  + user.suspend: đình chỉ tài khoản.
  + role.assign: cấp hoặc thu hồi vai trò.
  + audit.read: xem nhật ký bảo mật.


5. BẢNG GÁN QUYỀN CHO VAI TRÒ
-----------------------------

+-------------------------+
|    role_permissions     |
+-------------------------+
| PK,FK role_id           |
| PK,FK permission_id     |
+-------------------------+

Chức năng:
- Xác định mỗi vai trò được phép làm những gì.

Ví dụ:
- Người dùng thông thường được đăng nội dung.
- Người kiểm duyệt được xử lý nội dung vi phạm.
- Quản trị viên được quản lý tài khoản theo phạm vi
  quyền hạn đã được cấp.

Không nên mặc định rằng mọi quản trị viên đều có
quyền truy cập tất cả dữ liệu nhạy cảm.


6. BẢNG PHIÊN ĐĂNG NHẬP (sessions)
----------------------------------

+--------------------------------+
|            sessions            |
+--------------------------------+
| PK  id              UUID       |
| FK  user_id         UUID       |
| UK  token_hash      BYTEA      |
|     created_at                 |
|     expires_at                 |
|     revoked_at                 |
|     last_seen_at               |
+--------------------------------+

Chức năng:
- Theo dõi các phiên đăng nhập của người dùng.
- Xác định phiên nào còn hiệu lực.
- Cho phép thu hồi phiên đăng nhập khi cần thiết.

Ví dụ:
- Người dùng đăng nhập trên máy tính.
- Sau đó đăng nhập trên điện thoại.
- Hệ thống có thể quản lý hai phiên riêng biệt.

Bảo mật:
- Mã phiên phải được tạo bằng bộ sinh số ngẫu nhiên
  an toàn về mặt mật mã.
- Chỉ lưu giá trị băm của mã phiên, không lưu mã gốc.
- Khi người dùng đăng xuất hoặc phiên bị thu hồi,
  hệ thống phải từ chối sử dụng phiên đó.


7. BẢNG NHẬT KÝ BẢO MẬT (security_audit)
----------------------------------------

+--------------------------------+
|         security_audit         |
+--------------------------------+
| PK  id              BIGINT     |
| FK  actor_user_id   UUID       |
|     action          TEXT       |
|     resource_type   TEXT       |
|     resource_id     TEXT       |
|     outcome         TEXT       |
|     request_id      UUID       |
|     occurred_at     TIMESTAMP  |
+--------------------------------+

Chức năng:
- Ghi lại các hành động quan trọng trong hệ thống.
- Giúp điều tra khi có sự cố bảo mật.
- Hỗ trợ phát hiện hành vi bất thường.

Ví dụ:
- Ai đã cấp quyền quản trị?
- Ai đã đình chỉ tài khoản?
- Ai đã cố truy cập dữ liệu trái phép?
- Hành động xảy ra lúc nào và có thành công không?

Lưu ý:
Nhật ký cần được bảo vệ để kẻ tấn công không thể
dễ dàng sửa hoặc xóa dấu vết.


MỐI LIÊN KẾT GIỮA CÁC BẢNG
==========================

users N:M roles
    Thông qua bảng user_roles.

roles N:M permissions
    Thông qua bảng role_permissions.

users 1:N sessions
    Một người dùng có thể có nhiều phiên đăng nhập.

users 1:N security_audit
    Một người dùng có thể thực hiện nhiều hành động
    được ghi lại trong nhật ký.


QUY TRÌNH KIỂM TRA QUYỀN TRUY CẬP
=================================

       +----------------------+
       | Người dùng gửi yêu   |
       | cầu đến website      |
       +----------------------+
                  |
                  v
       +----------------------+
       | Kiểm tra phiên       |
       | đăng nhập            |
       +----------------------+
                  |
                  v
       +----------------------+
       | Tài khoản có đang    |
       | hoạt động không?     |
       +----------------------+
             |          |
            KHÔNG       CÓ
             |          |
             v          v
       +----------+  +------------------+
       | TỪ CHỐI  |  | Kiểm tra vai trò |
       | TRUY CẬP |  | của người dùng   |
       +----------+  +------------------+
                            |
                            v
                    +------------------+
                    | Kiểm tra quyền   |
                    | được cấp         |
                    +------------------+
                            |
                            v
                    +------------------+
                    | Kiểm tra quyền   |
                    | với dữ liệu cụ   |
                    | thể đang yêu cầu |
                    +------------------+
                         |        |
                       TỪ CHỐI   CHO PHÉP
                         |        |
                         v        v
                    +---------+ +----------------+
                    | HTTP 403| | Thực hiện      |
                    | Bị cấm  | | hành động      |
                    +---------+ +----------------+
                                       |
                                       v
                                +----------------+
                                | Ghi nhật ký    |
                                | bảo mật        |
                                +----------------+


CÁC LỚP BẢO VỆ HỆ THỐNG
=======================

+--------------------------------+
| NGƯỜI DÙNG / TRÌNH DUYỆT        |
| Nguồn gửi yêu cầu               |
+--------------------------------+
                |
                v
+--------------------------------+
| MÁY CHỦ ỨNG DỤNG                |
| - Xác minh danh tính            |
| - Kiểm tra dữ liệu đầu vào      |
| - Kiểm tra quyền truy cập       |
| - Ngăn truy vấn SQL nguy hiểm   |
| - Kiểm tra quyền sở hữu dữ liệu |
+--------------------------------+
                |
                v
+--------------------------------+
| CƠ SỞ DỮ LIỆU                  |
| - Tài khoản                     |
| - Vai trò                       |
| - Quyền hạn                     |
| - Phiên đăng nhập               |
| - Nhật ký bảo mật               |
+--------------------------------+
                |
                v
+--------------------------------+
| BẢN SAO LƯU ĐƯỢC BẢO VỆ         |
| - Mã hóa dữ liệu                |
| - Tài khoản truy cập riêng      |
| - Hạn chế người được phép dùng  |
| - Kiểm tra khả năng khôi phục   |
+--------------------------------+


12 NGUYÊN TẮC BẢO MẬT QUAN TRỌNG
================================

1. KHÔNG LƯU MẬT KHẨU NGUYÊN BẢN
   Sử dụng thuật toán băm mật khẩu Argon2id
   với cấu hình phù hợp.

2. BẢO VỆ PHIÊN ĐĂNG NHẬP
   Tạo mã phiên ngẫu nhiên an toàn.
   Chỉ lưu giá trị băm của mã phiên.

3. KHÔNG TIN DỮ LIỆU DO TRÌNH DUYỆT TỰ GỬI
   Không tự động tin mã người dùng, vai trò hoặc
   quyền hạn do phía trình duyệt khai báo.

4. KIỂM TRA QUYỀN TRONG MỌI YÊU CẦU NHẠY CẢM
   Không chỉ kiểm tra khi đăng nhập một lần.

5. KIỂM TRA QUYỀN SỞ HỮU DỮ LIỆU
   Người dùng không được mặc nhiên xem hoặc sửa
   dữ liệu riêng tư của người khác.

6. MẶC ĐỊNH TỪ CHỐI
   Nếu không có quyền rõ ràng thì không cho truy cập.

7. GIỚI HẠN QUYỀN CỦA TÀI KHOẢN CƠ SỞ DỮ LIỆU
   Không sử dụng tài khoản quản trị tối cao cho
   hoạt động thông thường của website.

8. BẢO VỆ NHẬT KÝ BẢO MẬT
   Ghi lại việc cấp quyền, thu hồi quyền và hành động
   nhạy cảm. Hạn chế quyền sửa hoặc xóa nhật ký.

9. BẬT XÁC THỰC ĐA YẾU TỐ
   Đặc biệt đối với quản trị viên và thao tác nhạy cảm.

10. MÃ HÓA BẢN SAO LƯU
    Sử dụng thông tin xác thực riêng và hạn chế quyền
    truy cập bản sao lưu.

11. THU HỒI PHIÊN KHI CÓ SỰ CỐ
    Vô hiệu hóa phiên khi tài khoản bị đình chỉ,
    thông tin đăng nhập bị lộ hoặc quyền bị thu hồi.

12. KIỂM TRA TOÀN VẸN DỮ LIỆU
    Sử dụng các ràng buộc của cơ sở dữ liệu và giao dịch
    để tránh lỗi khi thay đổi quyền truy cập.


MỤC TIÊU CUỐI CÙNG
==================

Nếu kẻ tấn công xâm nhập được một phần hệ thống,
họ không nên tự động có quyền truy cập toàn bộ dữ liệu.

Cần kết hợp:
- Thiết kế cơ sở dữ liệu hợp lý.
- Phân quyền tối thiểu.
- Xác minh quyền ở phía máy chủ.
- Bảo vệ thông tin đăng nhập và phiên.
- Nhật ký bảo mật khó bị can thiệp.
- Sao lưu an toàn và quy trình khôi phục.

LƯU Ý:
Sơ đồ cơ sở dữ liệu chỉ là một phần của giải pháp.
Bảo mật thực tế còn phụ thuộc vào mã nguồn ứng dụng,
cấu hình máy chủ, quyền cơ sở dữ liệu và quy trình
vận hành hệ thống.
