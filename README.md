# MÔ TẢ CHI TIẾT CÁC CHỨC NĂNG HỆ THỐNG

Tài liệu này mô tả chi tiết các chức năng của hệ thống GIVE-AID theo cấu trúc Input-Process-Output.

---

## **COMMON FUNCTIONS (Chức năng dùng chung)**

### **Tên chức năng 1: Đăng ký tài khoản (Register)**

| | |
|---|---|
| **Input** | • FullName (Họ tên): Chuỗi ký tự, bắt buộc<br>• Username (Tên đăng nhập): Chuỗi ký tự, bắt buộc, duy nhất<br>• Email (Email): Định dạng email hợp lệ, bắt buộc, duy nhất<br>• Password (Mật khẩu): Tối thiểu 6 ký tự, bắt buộc<br>• Phone (Số điện thoại): Tùy chọn<br>• Address (Địa chỉ): Tùy chọn |
| **Process** | 1. Kiểm tra request body không null<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Kiểm tra Email và Username đã tồn tại chưa (truy vấn database)<br>4. Nếu đã tồn tại → trả về lỗi "Email/Username already exists"<br>5. Nếu chưa tồn tại:<br>   - Hash mật khẩu bằng BCrypt<br>   - Tạo token xác thực email (32 ký tự ngẫu nhiên)<br>   - Hash token trước khi lưu vào database<br>   - Tạo User mới với EmailVerified = false<br>   - Lưu User vào database<br>6. Gửi email xác thực bất đồng bộ (không chặn response)<br>7. Trả về JWT token và thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Registration successful! Please check your email...", token: "JWT_TOKEN" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Email/Username already exists" }` hoặc `{ message: "Validation failed", errors: [...] }` |

---

### **Tên chức năng 2: Đăng nhập (Login)**

| | |
|---|---|
| **Input** | • UsernameOrEmail (Tên đăng nhập hoặc Email): Chuỗi ký tự, bắt buộc<br>• Password (Mật khẩu): Chuỗi ký tự, bắt buộc |
| **Process** | 1. Kiểm tra request body không null<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Xác định input là Email hay Username (kiểm tra ký tự "@")<br>4. Tìm User trong database theo Email hoặc Username<br>5. Nếu không tìm thấy → trả về lỗi "Invalid username/email or password"<br>6. Nếu tìm thấy:<br>   - Verify mật khẩu bằng BCrypt<br>   - Nếu mật khẩu sai → trả về lỗi "Invalid username/email or password"<br>   - Kiểm tra EmailVerified = true<br>   - Nếu chưa xác thực → trả về lỗi "Please verify your email..."<br>   - Nếu đã xác thực:<br>     - Tạo JWT token chứa thông tin User (Id, Username, Email, Role)<br>     - Trả về token và thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Login successful", token: "JWT_TOKEN" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Invalid username/email or password" }` hoặc `{ message: "Please verify your email..." }` |

---

### **Tên chức năng 3: Xác thực email (Verify Email)**

| | |
|---|---|
| **Input** | • Token (Token xác thực): Chuỗi ký tự 32 ký tự, bắt buộc (từ link trong email) |
| **Process** | 1. Kiểm tra token không null hoặc rỗng<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Hash token để so sánh với database<br>4. Tìm User có EmailVerificationToken khớp và chưa hết hạn<br>5. Nếu không tìm thấy hoặc hết hạn → trả về lỗi "Invalid or expired token"<br>6. Nếu tìm thấy:<br>   - Đặt EmailVerified = true<br>   - Xóa EmailVerificationToken và EmailVerificationTokenExpiry<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Email verified successfully" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Invalid or expired token" }` |

---

### **Tên chức năng 4: Quên mật khẩu (Forgot Password)**

| | |
|---|---|
| **Input** | • Email (Email): Định dạng email hợp lệ, bắt buộc |
| **Process** | 1. Kiểm tra Email không null hoặc rỗng<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Tìm User trong database theo Email<br>4. Nếu không tìm thấy → vẫn trả về thành công (bảo mật, không tiết lộ email có tồn tại)<br>5. Nếu tìm thấy:<br>   - Tạo token đặt lại mật khẩu (32 ký tự ngẫu nhiên)<br>   - Hash token trước khi lưu<br>   - Đặt PasswordResetToken và PasswordResetTokenExpiry (hết hạn sau 1 giờ)<br>   - Lưu vào database<br>   - Gửi email chứa link đặt lại mật khẩu bất đồng bộ<br>6. Luôn trả về thông báo thành công (bảo mật) |
| **Output** | • Luôn trả về: HTTP 200 OK với `{ message: "If email exists, password reset link has been sent to your email" }` |

---

### **Tên chức năng 5: Đặt lại mật khẩu (Reset Password)**

| | |
|---|---|
| **Input** | • Token (Token đặt lại mật khẩu): Chuỗi ký tự, bắt buộc (từ link trong email)<br>• NewPassword (Mật khẩu mới): Tối thiểu 6 ký tự, bắt buộc |
| **Process** | 1. Kiểm tra token và NewPassword không null hoặc rỗng<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Hash token để so sánh với database<br>4. Tìm User có PasswordResetToken khớp và chưa hết hạn<br>5. Nếu không tìm thấy hoặc hết hạn → trả về lỗi "Invalid or expired token"<br>6. Nếu tìm thấy:<br>   - Hash mật khẩu mới bằng BCrypt<br>   - Cập nhật PasswordHash<br>   - Xóa PasswordResetToken và PasswordResetTokenExpiry<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Password reset successfully" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Invalid or expired token" }` |

---

### **Tên chức năng 6: Thực hiện quyên góp (Donate)**

| | |
|---|---|
| **Input** | • Amount (Số tiền): Số thập phân > 0, bắt buộc<br>• Cause (Nguyên nhân): Chuỗi ký tự, bắt buộc<br>• FullName (Họ tên người quyên góp): Chuỗi ký tự, bắt buộc<br>• Email (Email người quyên góp): Định dạng email hợp lệ, bắt buộc<br>• Phone (Số điện thoại): Tùy chọn<br>• Address (Địa chỉ): Tùy chọn<br>• PaymentMethod (Phương thức thanh toán): Tùy chọn (mặc định "Card")<br>• ProgramId (ID chương trình): Số nguyên, tùy chọn (nếu quyên góp cho chương trình cụ thể)<br>• UserId (ID người dùng): Số nguyên, tùy chọn (null nếu chưa đăng nhập)<br>• Anonymous (Ẩn danh): Boolean, tùy chọn<br>• Newsletter (Đăng ký nhận tin): Boolean, tùy chọn |
| **Process** | 1. Kiểm tra DTO không null<br>2. Validate dữ liệu đầu vào (ModelState và kiểm tra thủ công)<br>3. Kiểm tra Amount > 0, Cause không rỗng, FullName không rỗng, Email không rỗng<br>4. Tạo TransactionReference duy nhất (format: "TRX-" + 12 ký tự ngẫu nhiên)<br>5. Xử lý DonorName: Nếu Anonymous = true → "Anonymous", ngược lại dùng FullName<br>6. Tạo đối tượng Donation với PaymentStatus = "Success"<br>7. Lưu Donation vào database<br>8. Nếu không phải Anonymous và có Email:<br>   - Gửi email xác nhận quyên góp bất đồng bộ (không chặn response)<br>   - Email chứa thông tin: Tên, Số tiền, Nguyên nhân, Mã giao dịch, Ngày giờ<br>9. Trả về thông tin Donation đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Donation (Id, Amount, CauseName, TransactionReference, DonorName, DonorEmail, CreatedAt, ...)<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Validation failed", errors: [...] }` hoặc `{ message: "Amount must be greater than 0" }` |

---

### **Tên chức năng 7: Xem danh sách chương trình (View Programs)**

| | |
|---|---|
| **Input** | Không có (GET request không có tham số) |
| **Process** | 1. Gọi ProgramService.GetAllAsync()<br>2. Truy vấn database lấy tất cả NgoProgram<br>3. Sắp xếp theo thứ tự (nếu có)<br>4. Trả về danh sách chương trình |
| **Output** | • Thành công: HTTP 200 OK với mảng các đối tượng Program (Id, Title, Description, StartDate, EndDate, Location, GoalAmount, CurrentAmount, ...) |

---

### **Tên chức năng 8: Xem thống kê chương trình (View Program Stats)**

| | |
|---|---|
| **Input** | • ProgramId (ID chương trình): Số nguyên, bắt buộc (từ URL path `/api/program/{id}/stats`) |
| **Process** | 1. Lấy ProgramId từ URL path<br>2. Gọi ProgramService.GetByIdAsync(id) để lấy thông tin chương trình<br>3. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>4. Nếu tìm thấy:<br>   - Gọi ProgramService.GetTotalDonationsAsync(id) để tính tổng số tiền quyên góp<br>   - Gọi ProgramService.GetProgressPercentageAsync(id) để tính phần trăm hoàn thành<br>   - Gọi ProgramService.GetRegistrationCountAsync(id) để đếm số lượt đăng ký<br>   - Tính RemainingAmount = GoalAmount - TotalDonations (nếu có GoalAmount)<br>5. Trả về đối tượng thống kê |
| **Output** | • Thành công: HTTP 200 OK với `{ programId, goalAmount, totalDonations, progressPercentage, registrationCount, remainingAmount }`<br>• Thất bại: HTTP 404 Not Found nếu không tìm thấy chương trình |

---

### **Tên chức năng 9: Đăng ký tham gia chương trình (Register for Program)**

| | |
|---|---|
| **Input** | • ProgramId (ID chương trình): Số nguyên, bắt buộc (từ URL path `/api/program/register`)<br>• FullName (Họ tên): Chuỗi ký tự, bắt buộc<br>• Email (Email): Định dạng email hợp lệ, bắt buộc<br>• Phone (Số điện thoại): Tùy chọn |
| **Process** | 1. Validate dữ liệu đầu vào (ModelState)<br>2. Kiểm tra ProgramId có tồn tại trong database<br>3. Nếu không tồn tại → trả về lỗi "Program not found"<br>4. Nếu tồn tại:<br>   - Kiểm tra User đã đăng ký chương trình này chưa (nếu có UserId từ JWT token)<br>   - Nếu đã đăng ký → trả về lỗi "Already registered"<br>   - Nếu chưa đăng ký:<br>     - Tạo ProgramRegistration mới<br>     - Lưu vào database<br>     - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Successfully registered for program" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Program not found" }` hoặc `{ message: "Already registered" }` |

---

### **Tên chức năng 10: Gửi câu hỏi/Yêu cầu (Submit Query)**

| | |
|---|---|
| **Input** | • Subject (Tiêu đề): Chuỗi ký tự, bắt buộc<br>• Message (Nội dung): Chuỗi ký tự, bắt buộc<br>• Email (Email): Định dạng email hợp lệ, bắt buộc<br>• FullName (Họ tên): Tùy chọn<br>• UserId (ID người dùng): Số nguyên, tùy chọn (null nếu chưa đăng nhập) |
| **Process** | 1. Validate dữ liệu đầu vào (ModelState)<br>2. Tạo đối tượng Query mới<br>3. Lưu Query vào database<br>4. Trả về thông tin Query đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Query (Id, Subject, Message, Email, FullName, CreatedAt, ...) |

---

### **Tên chức năng 11: Xem hồ sơ cá nhân (View Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | Không có (GET request, lấy UserId từ JWT token trong Authorization header) |
| **Process** | 1. Kiểm tra JWT token hợp lệ (middleware [Authorize])<br>2. Lấy UserId từ JWT token (ClaimTypes.NameIdentifier)<br>3. Nếu không có UserId → trả về HTTP 401 Unauthorized<br>4. Gọi ProfileService.GetProfileAsync(userId)<br>5. Truy vấn database lấy thông tin UserProfile và User<br>6. Nếu không tìm thấy → trả về null (không phải lỗi)<br>7. Nếu tìm thấy → trả về thông tin profile |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Profile (FullName, Email, Phone, Address, DateOfBirth, Gender, ...) hoặc null nếu chưa có profile<br>• Thất bại: HTTP 401 Unauthorized nếu token không hợp lệ |

---

### **Tên chức năng 12: Cập nhật hồ sơ cá nhân (Update Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • FullName (Họ tên): Chuỗi ký tự, tùy chọn<br>• Phone (Số điện thoại): Chuỗi ký tự, tùy chọn<br>• Address (Địa chỉ): Chuỗi ký tự, tùy chọn<br>• DateOfBirth (Ngày sinh): DateTime, tùy chọn<br>• Gender (Giới tính): Chuỗi ký tự, tùy chọn<br>• UserId được lấy từ JWT token (không cần gửi trong request) |
| **Process** | 1. Kiểm tra JWT token hợp lệ (middleware [Authorize])<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy UserId từ JWT token<br>4. Gọi ProfileService.UpdateProfileAsync(userId, request)<br>5. Tìm UserProfile theo UserId<br>6. Nếu không tìm thấy → tạo UserProfile mới<br>7. Nếu tìm thấy → cập nhật các trường từ request<br>8. Lưu thay đổi vào database<br>9. Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Profile updated successfully" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Failed to update profile" }` hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 13: Đổi mật khẩu (Change Password)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • OldPassword (Mật khẩu cũ): Chuỗi ký tự, bắt buộc<br>• NewPassword (Mật khẩu mới): Tối thiểu 6 ký tự, bắt buộc<br>• UserId được lấy từ JWT token (không cần gửi trong request) |
| **Process** | 1. Kiểm tra JWT token hợp lệ (middleware [Authorize])<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy UserId từ JWT token<br>4. Gọi AuthService.ChangePasswordAsync(userId, request)<br>5. Tìm User theo UserId<br>6. Verify OldPassword với PasswordHash trong database (BCrypt)<br>7. Nếu mật khẩu cũ sai → trả về lỗi "Old password is incorrect"<br>8. Nếu đúng:<br>   - Hash NewPassword bằng BCrypt<br>   - Cập nhật PasswordHash<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Password changed successfully" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Old password is incorrect" }` hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 14: Xem lịch sử quyên góp (View Donation History)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | Không có (GET request, lấy UserId từ JWT token trong Authorization header) |
| **Process** | 1. Kiểm tra JWT token hợp lệ (middleware [Authorize])<br>2. Lấy UserId từ JWT token<br>3. Gọi DonationService.GetByUserIdAsync(userId)<br>4. Truy vấn database lấy tất cả Donation có UserId = userId<br>5. Sắp xếp theo CreatedAt giảm dần (mới nhất trước)<br>6. Trả về danh sách donations |
| **Output** | • Thành công: HTTP 200 OK với mảng các đối tượng Donation (Id, Amount, CauseName, TransactionReference, DonorName, DonorEmail, CreatedAt, PaymentStatus, ...) |

---

## **ADMIN FUNCTIONS (Chức năng quản trị)**

### **Tên chức năng 1: Quản lý người dùng - Xem danh sách (Get All Users)**

| | |
|---|---|
| **Input** | Không có (GET request, yêu cầu role "Admin" trong JWT token) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin" (middleware [Authorize(Roles = "Admin")])<br>2. Gọi UserService.GetAllAsync()<br>3. Truy vấn database lấy tất cả User<br>4. Trả về danh sách users (không bao gồm PasswordHash) |
| **Output** | • Thành công: HTTP 200 OK với mảng các đối tượng User (Id, Username, Email, FullName, Role, EmailVerified, CreatedAt, ...)<br>• Thất bại: HTTP 401 Unauthorized nếu không phải Admin |

---

### **Tên chức năng 2: Quản lý người dùng - Xem chi tiết (Get User by ID)**

| | |
|---|---|
| **Input** | • UserId (ID người dùng): Số nguyên, bắt buộc (từ URL path `/api/admin/users/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy UserId từ URL path<br>3. Gọi UserService.GetByIdAsync(id)<br>4. Truy vấn database tìm User theo Id<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy → trả về thông tin User |
| **Output** | • Thành công: HTTP 200 OK với đối tượng User<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 3: Quản lý người dùng - Cập nhật vai trò (Update User Role)**

| | |
|---|---|
| **Input** | • UserId (ID người dùng): Số nguyên, bắt buộc (từ URL path `/api/admin/users/{id}/role`)<br>• Role (Vai trò): Chuỗi ký tự ("User" hoặc "Admin"), bắt buộc (từ request body) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate Role không null hoặc rỗng<br>3. Lấy UserId từ URL path<br>4. Gọi UserService.UpdateRoleAsync(id, role)<br>5. Tìm User theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật Role<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Role updated" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Role is required" }`, HTTP 404 Not Found, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 4: Quản lý người dùng - Xóa người dùng (Delete User)**

| | |
|---|---|
| **Input** | • UserId (ID người dùng): Số nguyên, bắt buộc (từ URL path `/api/admin/users/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy UserId từ URL path<br>3. Gọi UserService.DeleteAsync(id)<br>4. Tìm User theo Id<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy:<br>   - Xóa User khỏi database (có thể xóa cascade các bản ghi liên quan như Donation, Query, ...)<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "User deleted" }`<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 5: Quản lý quyên góp - Xem tất cả (Get All Donations)**

| | |
|---|---|
| **Input** | Không có (GET request, yêu cầu role "Admin") |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Gọi DonationService.GetAllAsync()<br>3. Truy vấn database lấy tất cả Donation (kèm thông tin User nếu có)<br>4. Sắp xếp theo CreatedAt giảm dần<br>5. Trả về danh sách donations |
| **Output** | • Thành công: HTTP 200 OK với mảng các đối tượng Donation (bao gồm thông tin User)<br>• Thất bại: HTTP 401 Unauthorized |

---

### **Tên chức năng 6: Quản lý quyên góp - Xem chi tiết (Get Donation by ID)**

| | |
|---|---|
| **Input** | • DonationId (ID quyên góp): Số nguyên, bắt buộc (từ URL path `/api/admin/donations/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy DonationId từ URL path<br>3. Gọi DonationService.GetByIdAsync(id)<br>4. Truy vấn database tìm Donation theo Id (kèm thông tin User)<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy → trả về thông tin Donation |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Donation<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 7: Quản lý chương trình - Tạo mới (Create Program)**

| | |
|---|---|
| **Input** | • Title (Tiêu đề): Chuỗi ký tự, bắt buộc<br>• Description (Mô tả): Chuỗi ký tự, bắt buộc<br>• StartDate (Ngày bắt đầu): DateTime, tùy chọn<br>• EndDate (Ngày kết thúc): DateTime, tùy chọn<br>• Location (Địa điểm): Chuỗi ký tự, tùy chọn<br>• GoalAmount (Mục tiêu số tiền): Decimal, tùy chọn<br>• NGOId (ID tổ chức NGO): Số nguyên, tùy chọn |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Gọi ProgramService.CreateAsync(model)<br>4. Tạo đối tượng NgoProgram mới<br>5. Lưu vào database<br>6. Trả về thông tin Program đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Program (Id, Title, Description, ...)<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Validation failed", errors: [...] }` hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 8: Quản lý chương trình - Cập nhật (Update Program)**

| | |
|---|---|
| **Input** | • ProgramId (ID chương trình): Số nguyên, bắt buộc (từ URL path `/api/program/{id}`)<br>• Title, Description, StartDate, EndDate, Location, GoalAmount, NGOId (các trường cần cập nhật, tùy chọn) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy ProgramId từ URL path<br>4. Gọi ProgramService.UpdateAsync(id, model)<br>5. Tìm NgoProgram theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật các trường từ request<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Program updated successfully" }`<br>• Thất bại: HTTP 404 Not Found, HTTP 400 Bad Request, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 9: Quản lý chương trình - Xóa (Delete Program)**

| | |
|---|---|
| **Input** | • ProgramId (ID chương trình): Số nguyên, bắt buộc (từ URL path `/api/program/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy ProgramId từ URL path<br>3. Gọi ProgramService.DeleteAsync(id)<br>4. Tìm NgoProgram theo Id<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy:<br>   - Xóa NgoProgram khỏi database (có thể xóa cascade các bản ghi liên quan như ProgramRegistration, Donation, Gallery, ...)<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Program deleted successfully" }`<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 10: Quản lý NGO - Tạo mới (Create NGO)**

| | |
|---|---|
| **Input** | • Name (Tên tổ chức): Chuỗi ký tự, bắt buộc<br>• Description (Mô tả): Chuỗi ký tự, tùy chọn<br>• LogoUrl (URL logo): Chuỗi ký tự, tùy chọn<br>• Website (Website): Chuỗi ký tự, tùy chọn |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Gọi NGOService.CreateAsync(model)<br>4. Tạo đối tượng NGO mới<br>5. Lưu vào database<br>6. Trả về thông tin NGO đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng NGO (Id, Name, Description, LogoUrl, Website, ...)<br>• Thất bại: HTTP 400 Bad Request hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 11: Quản lý NGO - Cập nhật (Update NGO)**

| | |
|---|---|
| **Input** | • NGOId (ID tổ chức): Số nguyên, bắt buộc (từ URL path `/api/ngo/{id}`)<br>• Name, Description, LogoUrl, Website (các trường cần cập nhật, tùy chọn) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy NGOId từ URL path<br>4. Gọi NGOService.UpdateAsync(id, model)<br>5. Tìm NGO theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật các trường từ request<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "NGO updated successfully" }`<br>• Thất bại: HTTP 404 Not Found, HTTP 400 Bad Request, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 12: Quản lý NGO - Xóa (Delete NGO)**

| | |
|---|---|
| **Input** | • NGOId (ID tổ chức): Số nguyên, bắt buộc (từ URL path `/api/ngo/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy NGOId từ URL path<br>3. Gọi NGOService.DeleteAsync(id)<br>4. Tìm NGO theo Id<br>5. Kiểm tra NGO có đang được sử dụng bởi Program nào không<br>6. Nếu đang được sử dụng → trả về lỗi "Cannot delete NGO that has associated programs"<br>7. Nếu không được sử dụng:<br>   - Xóa NGO khỏi database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "NGO deleted successfully" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Cannot delete NGO that has associated programs" }`, HTTP 404 Not Found, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 13: Quản lý Gallery - Thêm ảnh (Create Gallery Item)**

| | |
|---|---|
| **Input** | • File (File ảnh): Multipart form data, tùy chọn (nếu không có thì phải có ImageUrl)<br>• ImageUrl (URL ảnh): Chuỗi ký tự, tùy chọn (nếu không có thì phải có File)<br>• Caption (Chú thích): Chuỗi ký tự, tùy chọn<br>• ProgramId (ID chương trình): Số nguyên, tùy chọn |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Kiểm tra phải có File hoặc ImageUrl (ít nhất một trong hai)<br>3. Nếu có File:<br>   - Validate file type (chỉ cho phép .jpg, .jpeg, .png, .gif, .webp)<br>   - Validate file size (tối đa 5MB)<br>   - Lưu file vào thư mục `wwwroot/uploads/gallery/`<br>   - Tạo ImageUrl từ đường dẫn file đã lưu<br>4. Nếu có ImageUrl:<br>   - Sử dụng ImageUrl trực tiếp<br>5. Tạo đối tượng Gallery mới<br>6. Lưu vào database<br>7. Trả về thông tin Gallery đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Gallery (Id, ImageUrl, Caption, ProgramId, CreatedAt, ...)<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Invalid file type" }` hoặc `{ message: "File size exceeds 5MB limit" }` hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 14: Quản lý Gallery - Xóa ảnh (Delete Gallery Item)**

| | |
|---|---|
| **Input** | • GalleryId (ID ảnh): Số nguyên, bắt buộc (từ URL path `/api/gallery/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy GalleryId từ URL path<br>3. Gọi GalleryService.DeleteAsync(id)<br>4. Tìm Gallery theo Id<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy:<br>   - Xóa file ảnh khỏi thư mục (nếu là file upload)<br>   - Xóa Gallery khỏi database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Gallery item deleted successfully" }`<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 15: Quản lý Đối tác - Tạo mới (Create Partner)**

| | |
|---|---|
| **Input** | • Name (Tên đối tác): Chuỗi ký tự, bắt buộc<br>• LogoUrl (URL logo): Chuỗi ký tự, tùy chọn<br>• Website (Website): Chuỗi ký tự, tùy chọn |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Gọi PartnerService.CreateAsync(model)<br>4. Tạo đối tượng Partner mới<br>5. Lưu vào database<br>6. Trả về thông tin Partner đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng Partner (Id, Name, LogoUrl, Website, CreatedAt, ...)<br>• Thất bại: HTTP 400 Bad Request hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 16: Quản lý Đối tác - Cập nhật (Update Partner)**

| | |
|---|---|
| **Input** | • PartnerId (ID đối tác): Số nguyên, bắt buộc (từ URL path `/api/partner/{id}`)<br>• Name, LogoUrl, Website (các trường cần cập nhật, tùy chọn) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy PartnerId từ URL path<br>4. Gọi PartnerService.UpdateAsync(id, model)<br>5. Tìm Partner theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật các trường từ request<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Partner updated successfully" }`<br>• Thất bại: HTTP 404 Not Found, HTTP 400 Bad Request, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 17: Quản lý Đối tác - Xóa (Delete Partner)**

| | |
|---|---|
| **Input** | • PartnerId (ID đối tác): Số nguyên, bắt buộc (từ URL path `/api/partner/{id}`) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Lấy PartnerId từ URL path<br>3. Gọi PartnerService.DeleteAsync(id)<br>4. Tìm Partner theo Id<br>5. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>6. Nếu tìm thấy:<br>   - Xóa Partner khỏi database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Partner deleted successfully" }`<br>• Thất bại: HTTP 404 Not Found hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 18: Quản lý Nội dung Giới thiệu - Tạo mới (Create About Section)**

| | |
|---|---|
| **Input** | • Key (Khóa): Chuỗi ký tự, bắt buộc, duy nhất (ví dụ: "mission", "vision", "history")<br>• Title (Tiêu đề): Chuỗi ký tự, bắt buộc<br>• Content (Nội dung): Chuỗi ký tự, bắt buộc<br>• ExtraJson (Dữ liệu bổ sung): JSON string, tùy chọn |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Gọi AboutService.CreateAsync(model)<br>4. Tạo đối tượng AboutSection mới<br>5. Lưu vào database<br>6. Trả về thông tin AboutSection đã tạo |
| **Output** | • Thành công: HTTP 200 OK với đối tượng AboutSection (Id, Key, Title, Content, ExtraJson, CreatedAt, ...)<br>• Thất bại: HTTP 400 Bad Request hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 19: Quản lý Nội dung Giới thiệu - Cập nhật (Update About Section)**

| | |
|---|---|
| **Input** | • AboutSectionId (ID phần nội dung): Số nguyên, bắt buộc (từ URL path `/api/about/{id}`)<br>• Title, Content, ExtraJson (các trường cần cập nhật, tùy chọn) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate dữ liệu đầu vào (ModelState)<br>3. Lấy AboutSectionId từ URL path<br>4. Gọi AboutService.UpdateAsync(id, model)<br>5. Tìm AboutSection theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật các trường từ request<br>   - Lưu thay đổi vào database<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "About section updated successfully" }`<br>• Thất bại: HTTP 404 Not Found, HTTP 400 Bad Request, hoặc HTTP 401 Unauthorized |

---

### **Tên chức năng 20: Quản lý Câu hỏi - Xem tất cả (Get All Queries)**

| | |
|---|---|
| **Input** | Không có (GET request, yêu cầu role "Admin") |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Gọi QueryService.GetAllAsync()<br>3. Truy vấn database lấy tất cả Query (kèm thông tin User nếu có)<br>4. Sắp xếp theo CreatedAt giảm dần<br>5. Trả về danh sách queries |
| **Output** | • Thành công: HTTP 200 OK với mảng các đối tượng Query (Id, Subject, Message, Email, FullName, Reply, CreatedAt, ...)<br>• Thất bại: HTTP 401 Unauthorized |

---

### **Tên chức năng 21: Quản lý Câu hỏi - Trả lời (Reply to Query)**

| | |
|---|---|
| **Input** | • QueryId (ID câu hỏi): Số nguyên, bắt buộc (từ URL path `/api/admin/queries/{id}/reply`)<br>• Reply (Nội dung trả lời): Chuỗi ký tự, bắt buộc (từ request body) |
| **Process** | 1. Kiểm tra JWT token hợp lệ và role = "Admin"<br>2. Validate Reply không null hoặc rỗng<br>3. Lấy QueryId từ URL path<br>4. Gọi QueryService.ReplyAsync(id, reply)<br>5. Tìm Query theo Id<br>6. Nếu không tìm thấy → trả về HTTP 404 Not Found<br>7. Nếu tìm thấy:<br>   - Cập nhật Reply và ReplyDate<br>   - Lưu thay đổi vào database<br>   - Nếu Query có User và User có Email:<br>     - Gửi email phản hồi bất đồng bộ cho người dùng<br>     - Email chứa Subject, Reply, và tên người dùng<br>   - Trả về thông báo thành công |
| **Output** | • Thành công: HTTP 200 OK với `{ message: "Replied" }`<br>• Thất bại: HTTP 400 Bad Request với `{ message: "Reply is required" }`, HTTP 404 Not Found, hoặc HTTP 401 Unauthorized |

---

## **Ghi chú**

- Tất cả các chức năng yêu cầu đăng nhập đều sử dụng JWT token trong header `Authorization: Bearer {token}`
- Các chức năng Admin yêu cầu role = "Admin" trong JWT token
- Email được gửi bất đồng bộ (fire-and-forget) để không chặn response
- Tất cả mật khẩu được hash bằng BCrypt trước khi lưu vào database
- Token xác thực email và đặt lại mật khẩu có thời hạn (24 giờ và 1 giờ tương ứng)

