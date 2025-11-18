# MÔ TẢ CHI TIẾT CÁC CHỨC NĂNG HỆ THỐNG

Tài liệu này mô tả chi tiết các chức năng của hệ thống GIVE-AID theo cấu trúc Input-Process-Output.

---

## **COMMON FUNCTIONS (Chức năng dùng chung)**

### **Tên chức năng 1: Đăng ký tài khoản (Register)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Họ tên, Tên đăng nhập, Email, Mật khẩu, Số điện thoại (tùy chọn), Địa chỉ (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra Email và Tên đăng nhập đã tồn tại trong database chưa<br>• Nếu đã tồn tại, hệ thống thông báo lỗi cho người dùng<br>• Nếu chưa tồn tại, hệ thống mã hóa mật khẩu và tạo token xác thực email<br>• Hệ thống tạo tài khoản mới với trạng thái email chưa xác thực<br>• Hệ thống lưu thông tin người dùng vào database<br>• Hệ thống gửi email xác thực cho người dùng (gửi bất đồng bộ, không chặn phản hồi)<br>• Hệ thống trả về token đăng nhập và thông báo thành công |
| **Output** | • Kết quả là thông báo thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 2: Đăng nhập (Login)**

| | |
|---|---|
| **Input** | • Người dùng nhập Tên đăng nhập hoặc Email và Mật khẩu |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống xác định người dùng nhập Email hay Tên đăng nhập<br>• Hệ thống tìm kiếm người dùng trong database theo Email hoặc Tên đăng nhập<br>• Nếu không tìm thấy, hệ thống thông báo lỗi đăng nhập<br>• Nếu tìm thấy, hệ thống xác minh mật khẩu<br>• Nếu mật khẩu sai, hệ thống thông báo lỗi đăng nhập<br>• Nếu mật khẩu đúng, hệ thống kiểm tra email đã được xác thực chưa<br>• Nếu email chưa xác thực, hệ thống yêu cầu người dùng xác thực email trước<br>• Nếu email đã xác thực, hệ thống tạo token đăng nhập và trả về cho người dùng |
| **Output** | • Kết quả là token đăng nhập và thông báo thành công, hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 3: Xác thực email (Verify Email)**

| | |
|---|---|
| **Input** | • Người dùng nhập token xác thực từ link trong email |
| **Process** | • Hệ thống kiểm tra token có hợp lệ không<br>• Hệ thống tìm kiếm người dùng có token xác thực khớp và chưa hết hạn<br>• Nếu không tìm thấy hoặc token hết hạn, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống đánh dấu email đã được xác thực<br>• Hệ thống xóa token xác thực và thời hạn token<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo xác thực thành công |
| **Output** | • Kết quả là thông báo xác thực thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 4: Quên mật khẩu (Forgot Password)**

| | |
|---|---|
| **Input** | • Người dùng nhập Email |
| **Process** | • Hệ thống kiểm tra Email có hợp lệ không<br>• Hệ thống tìm kiếm người dùng trong database theo Email<br>• Nếu tìm thấy, hệ thống tạo token đặt lại mật khẩu và đặt thời hạn hết hạn<br>• Hệ thống lưu token vào database<br>• Hệ thống gửi email chứa link đặt lại mật khẩu cho người dùng (gửi bất đồng bộ)<br>• Hệ thống luôn trả về thông báo thành công (bảo mật, không tiết lộ email có tồn tại hay không) |
| **Output** | • Kết quả là thông báo thành công cho người dùng (nếu email tồn tại, link đặt lại mật khẩu đã được gửi) |

---

### **Tên chức năng 5: Đặt lại mật khẩu (Reset Password)**

| | |
|---|---|
| **Input** | • Người dùng nhập token đặt lại mật khẩu từ link trong email và mật khẩu mới |
| **Process** | • Hệ thống kiểm tra token và mật khẩu mới có hợp lệ không<br>• Hệ thống tìm kiếm người dùng có token đặt lại mật khẩu khớp và chưa hết hạn<br>• Nếu không tìm thấy hoặc token hết hạn, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống mã hóa mật khẩu mới<br>• Hệ thống cập nhật mật khẩu mới vào database<br>• Hệ thống xóa token đặt lại mật khẩu và thời hạn token<br>• Hệ thống thông báo đặt lại mật khẩu thành công |
| **Output** | • Kết quả là thông báo đặt lại mật khẩu thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 6: Thực hiện quyên góp (Donate)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin quyên góp: Số tiền, Nguyên nhân, Họ tên, Email, Số điện thoại (tùy chọn), Địa chỉ (tùy chọn), Phương thức thanh toán (tùy chọn), Chọn chương trình (tùy chọn), Chọn ẩn danh (tùy chọn), Đăng ký nhận tin (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra số tiền lớn hơn 0, nguyên nhân không rỗng, họ tên không rỗng, email không rỗng<br>• Hệ thống tạo mã giao dịch duy nhất cho quyên góp<br>• Nếu người dùng chọn ẩn danh, hệ thống đặt tên người quyên góp là "Anonymous"<br>• Hệ thống tạo bản ghi quyên góp với trạng thái thanh toán thành công<br>• Hệ thống lưu thông tin quyên góp vào database<br>• Nếu không phải ẩn danh và có email, hệ thống gửi email xác nhận quyên góp cho người dùng (gửi bất đồng bộ)<br>• Hệ thống trả về thông tin quyên góp đã tạo |
| **Output** | • Kết quả là thông tin quyên góp đã tạo hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 7: Xem danh sách chương trình (View Programs)**

| | |
|---|---|
| **Input** | • Người dùng truy cập trang danh sách chương trình |
| **Process** | • Hệ thống truy vấn database lấy tất cả chương trình<br>• Hệ thống sắp xếp danh sách chương trình<br>• Hệ thống hiển thị danh sách chương trình cho người dùng |
| **Output** | • Kết quả là danh sách các chương trình hiển thị cho người dùng |

---

### **Tên chức năng 8: Xem thống kê chương trình (View Program Stats)**

| | |
|---|---|
| **Input** | • Người dùng chọn chương trình muốn xem thống kê |
| **Process** | • Hệ thống lấy thông tin chương trình từ database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống tính tổng số tiền quyên góp cho chương trình<br>• Hệ thống tính phần trăm hoàn thành mục tiêu<br>• Hệ thống đếm số lượt đăng ký tham gia chương trình<br>• Hệ thống tính số tiền còn lại cần quyên góp (nếu có mục tiêu)<br>• Hệ thống hiển thị thống kê cho người dùng |
| **Output** | • Kết quả là thông tin thống kê chương trình hiển thị cho người dùng, hoặc thông báo lỗi nếu không tìm thấy chương trình |

---

### **Tên chức năng 9: Đăng ký tham gia chương trình (Register for Program)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Họ tên, Email, Số điện thoại (tùy chọn) và chọn chương trình muốn đăng ký |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra chương trình có tồn tại trong database không<br>• Nếu không tồn tại, hệ thống thông báo lỗi<br>• Nếu tồn tại, hệ thống kiểm tra người dùng đã đăng ký chương trình này chưa (nếu đã đăng nhập)<br>• Nếu đã đăng ký, hệ thống thông báo lỗi<br>• Nếu chưa đăng ký, hệ thống tạo bản ghi đăng ký mới<br>• Hệ thống lưu thông tin đăng ký vào database<br>• Hệ thống thông báo đăng ký thành công |
| **Output** | • Kết quả là thông báo đăng ký thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 10: Gửi câu hỏi/Yêu cầu (Submit Query)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Tiêu đề, Nội dung câu hỏi, Email, Họ tên (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo bản ghi câu hỏi mới<br>• Hệ thống lưu thông tin câu hỏi vào database<br>• Hệ thống trả về thông tin câu hỏi đã tạo |
| **Output** | • Kết quả là thông tin câu hỏi đã được gửi thành công |

---

### **Tên chức năng 11: Xem hồ sơ cá nhân (View Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập truy cập trang hồ sơ cá nhân |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Nếu token không hợp lệ, hệ thống từ chối truy cập<br>• Hệ thống truy vấn database lấy thông tin hồ sơ cá nhân của người dùng<br>• Nếu chưa có hồ sơ, hệ thống trả về thông tin rỗng<br>• Nếu có hồ sơ, hệ thống hiển thị thông tin hồ sơ cho người dùng |
| **Output** | • Kết quả là thông tin hồ sơ cá nhân hiển thị cho người dùng, hoặc thông báo lỗi nếu không có quyền truy cập |

---

### **Tên chức năng 12: Cập nhật hồ sơ cá nhân (Update Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập nhập thông tin cần cập nhật: Họ tên, Số điện thoại, Địa chỉ, Ngày sinh, Giới tính (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống tìm kiếm hồ sơ cá nhân của người dùng trong database<br>• Nếu chưa có hồ sơ, hệ thống tạo hồ sơ mới<br>• Nếu đã có hồ sơ, hệ thống cập nhật các trường thông tin từ dữ liệu người dùng nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 13: Đổi mật khẩu (Change Password)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập nhập mật khẩu cũ và mật khẩu mới |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống tìm kiếm người dùng trong database<br>• Hệ thống xác minh mật khẩu cũ có đúng không<br>• Nếu mật khẩu cũ sai, hệ thống thông báo lỗi<br>• Nếu mật khẩu cũ đúng, hệ thống mã hóa mật khẩu mới<br>• Hệ thống cập nhật mật khẩu mới vào database<br>• Hệ thống thông báo đổi mật khẩu thành công |
| **Output** | • Kết quả là thông báo đổi mật khẩu thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 14: Xem lịch sử quyên góp (View Donation History)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập truy cập trang lịch sử quyên góp |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống truy vấn database lấy tất cả quyên góp của người dùng<br>• Hệ thống sắp xếp danh sách quyên góp theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách quyên góp cho người dùng |
| **Output** | • Kết quả là danh sách các quyên góp của người dùng hiển thị trên trang |

---

## **ADMIN FUNCTIONS (Chức năng quản trị)**

### **Tên chức năng 1: Quản lý người dùng - Xem danh sách (Get All Users)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý người dùng |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Nếu không có quyền, hệ thống từ chối truy cập<br>• Hệ thống truy vấn database lấy tất cả người dùng<br>• Hệ thống hiển thị danh sách người dùng cho quản trị viên (không hiển thị mật khẩu) |
| **Output** | • Kết quả là danh sách tất cả người dùng hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 2: Quản lý người dùng - Xem chi tiết (Get User by ID)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng muốn xem chi tiết |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID người dùng<br>• Hệ thống truy vấn database tìm kiếm người dùng theo ID<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống hiển thị thông tin chi tiết người dùng cho quản trị viên |
| **Output** | • Kết quả là thông tin chi tiết người dùng hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không tìm thấy hoặc không có quyền |

---

### **Tên chức năng 3: Quản lý người dùng - Cập nhật vai trò (Update User Role)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng và chọn vai trò mới (User hoặc Admin) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra vai trò có hợp lệ không<br>• Hệ thống lấy ID người dùng và vai trò mới<br>• Hệ thống tìm kiếm người dùng trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật vai trò mới<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật vai trò thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 4: Quản lý người dùng - Xóa người dùng (Delete User)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID người dùng<br>• Hệ thống tìm kiếm người dùng trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa người dùng khỏi database (có thể xóa kèm các bản ghi liên quan như quyên góp, câu hỏi)<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 5: Quản lý quyên góp - Xem tất cả (Get All Donations)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý quyên góp |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống truy vấn database lấy tất cả quyên góp (kèm thông tin người dùng nếu có)<br>• Hệ thống sắp xếp danh sách quyên góp theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách quyên góp cho quản trị viên |
| **Output** | • Kết quả là danh sách tất cả quyên góp hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 6: Quản lý quyên góp - Xem chi tiết (Get Donation by ID)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn quyên góp muốn xem chi tiết |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID quyên góp<br>• Hệ thống truy vấn database tìm kiếm quyên góp theo ID (kèm thông tin người dùng)<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống hiển thị thông tin chi tiết quyên góp cho quản trị viên |
| **Output** | • Kết quả là thông tin chi tiết quyên góp hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không tìm thấy hoặc không có quyền |

---

### **Tên chức năng 7: Quản lý chương trình - Tạo mới (Create Program)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin chương trình: Tiêu đề, Mô tả, Ngày bắt đầu (tùy chọn), Ngày kết thúc (tùy chọn), Địa điểm (tùy chọn), Mục tiêu số tiền (tùy chọn), Chọn tổ chức NGO (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo chương trình mới<br>• Hệ thống lưu thông tin chương trình vào database<br>• Hệ thống trả về thông tin chương trình đã tạo |
| **Output** | • Kết quả là thông tin chương trình đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 8: Quản lý chương trình - Cập nhật (Update Program)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn chương trình và nhập thông tin cần cập nhật: Tiêu đề, Mô tả, Ngày bắt đầu, Ngày kết thúc, Địa điểm, Mục tiêu số tiền, Tổ chức NGO (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID chương trình<br>• Hệ thống tìm kiếm chương trình trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 9: Quản lý chương trình - Xóa (Delete Program)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn chương trình muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID chương trình<br>• Hệ thống tìm kiếm chương trình trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa chương trình khỏi database (có thể xóa kèm các bản ghi liên quan như đăng ký tham gia, quyên góp, ảnh gallery)<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 10: Quản lý NGO - Tạo mới (Create NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin tổ chức NGO: Tên tổ chức, Mô tả (tùy chọn), URL logo (tùy chọn), Website (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo tổ chức NGO mới<br>• Hệ thống lưu thông tin tổ chức NGO vào database<br>• Hệ thống trả về thông tin tổ chức NGO đã tạo |
| **Output** | • Kết quả là thông tin tổ chức NGO đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 11: Quản lý NGO - Cập nhật (Update NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn tổ chức NGO và nhập thông tin cần cập nhật: Tên tổ chức, Mô tả, URL logo, Website (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID tổ chức NGO<br>• Hệ thống tìm kiếm tổ chức NGO trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 12: Quản lý NGO - Xóa (Delete NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn tổ chức NGO muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID tổ chức NGO<br>• Hệ thống tìm kiếm tổ chức NGO trong database<br>• Hệ thống kiểm tra tổ chức NGO có đang được sử dụng bởi chương trình nào không<br>• Nếu đang được sử dụng, hệ thống thông báo lỗi và không cho phép xóa<br>• Nếu không được sử dụng, hệ thống xóa tổ chức NGO khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 13: Quản lý Gallery - Thêm ảnh (Create Gallery Item)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn file ảnh hoặc nhập URL ảnh, nhập chú thích (tùy chọn), chọn chương trình liên quan (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra phải có file ảnh hoặc URL ảnh (ít nhất một trong hai)<br>• Nếu có file ảnh, hệ thống kiểm tra loại file (chỉ cho phép ảnh), kiểm tra kích thước file (tối đa 5MB)<br>• Nếu file hợp lệ, hệ thống lưu file vào thư mục và tạo URL ảnh<br>• Nếu có URL ảnh, hệ thống sử dụng URL trực tiếp<br>• Hệ thống tạo bản ghi ảnh mới<br>• Hệ thống lưu thông tin ảnh vào database<br>• Hệ thống trả về thông tin ảnh đã tạo |
| **Output** | • Kết quả là thông tin ảnh đã tạo hoặc thông báo lỗi cho quản trị viên (lỗi loại file, kích thước file, hoặc không có quyền) |

---

### **Tên chức năng 14: Quản lý Gallery - Xóa ảnh (Delete Gallery Item)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn ảnh muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID ảnh<br>• Hệ thống tìm kiếm ảnh trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa file ảnh khỏi thư mục (nếu là file upload)<br>• Hệ thống xóa bản ghi ảnh khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 15: Quản lý Đối tác - Tạo mới (Create Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin đối tác: Tên đối tác, URL logo (tùy chọn), Website (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo đối tác mới<br>• Hệ thống lưu thông tin đối tác vào database<br>• Hệ thống trả về thông tin đối tác đã tạo |
| **Output** | • Kết quả là thông tin đối tác đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 16: Quản lý Đối tác - Cập nhật (Update Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn đối tác và nhập thông tin cần cập nhật: Tên đối tác, URL logo, Website (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID đối tác<br>• Hệ thống tìm kiếm đối tác trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 17: Quản lý Đối tác - Xóa (Delete Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn đối tác muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID đối tác<br>• Hệ thống tìm kiếm đối tác trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa đối tác khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 18: Quản lý Nội dung Giới thiệu - Tạo mới (Create About Section)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin phần nội dung: Khóa (duy nhất), Tiêu đề, Nội dung, Dữ liệu bổ sung (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo phần nội dung mới<br>• Hệ thống lưu thông tin phần nội dung vào database<br>• Hệ thống trả về thông tin phần nội dung đã tạo |
| **Output** | • Kết quả là thông tin phần nội dung đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 19: Quản lý Nội dung Giới thiệu - Cập nhật (Update About Section)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn phần nội dung và nhập thông tin cần cập nhật: Tiêu đề, Nội dung, Dữ liệu bổ sung (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID phần nội dung<br>• Hệ thống tìm kiếm phần nội dung trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 20: Quản lý Câu hỏi - Xem tất cả (Get All Queries)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý câu hỏi |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống truy vấn database lấy tất cả câu hỏi (kèm thông tin người dùng nếu có)<br>• Hệ thống sắp xếp danh sách câu hỏi theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách câu hỏi cho quản trị viên |
| **Output** | • Kết quả là danh sách tất cả câu hỏi hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 21: Quản lý Câu hỏi - Trả lời (Reply to Query)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn câu hỏi và nhập nội dung trả lời |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra nội dung trả lời có hợp lệ không<br>• Hệ thống lấy ID câu hỏi<br>• Hệ thống tìm kiếm câu hỏi trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật nội dung trả lời và ngày trả lời<br>• Hệ thống lưu thay đổi vào database<br>• Nếu câu hỏi có người dùng và người dùng có email, hệ thống gửi email phản hồi cho người dùng (gửi bất đồng bộ)<br>• Hệ thống thông báo trả lời thành công |
| **Output** | • Kết quả là thông báo trả lời thành công hoặc thông báo lỗi cho quản trị viên |

---

## **Ghi chú**

- Tất cả các chức năng yêu cầu đăng nhập đều sử dụng token đăng nhập trong header
- Các chức năng Admin yêu cầu người dùng có vai trò Admin
- Email được gửi bất đồng bộ để không làm chậm phản hồi của hệ thống
- Tất cả mật khẩu được mã hóa trước khi lưu vào database
- Token xác thực email và đặt lại mật khẩu có thời hạn sử dụng
# MÔ TẢ CHI TIẾT CÁC CHỨC NĂNG HỆ THỐNG

Tài liệu này mô tả chi tiết các chức năng của hệ thống GIVE-AID theo cấu trúc Input-Process-Output.

---

## **COMMON FUNCTIONS (Chức năng dùng chung)**

### **Tên chức năng 1: Đăng ký tài khoản (Register)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Họ tên, Tên đăng nhập, Email, Mật khẩu, Số điện thoại (tùy chọn), Địa chỉ (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra Email và Tên đăng nhập đã tồn tại trong database chưa<br>• Nếu đã tồn tại, hệ thống thông báo lỗi cho người dùng<br>• Nếu chưa tồn tại, hệ thống mã hóa mật khẩu và tạo token xác thực email<br>• Hệ thống tạo tài khoản mới với trạng thái email chưa xác thực<br>• Hệ thống lưu thông tin người dùng vào database<br>• Hệ thống gửi email xác thực cho người dùng (gửi bất đồng bộ, không chặn phản hồi)<br>• Hệ thống trả về token đăng nhập và thông báo thành công |
| **Output** | • Kết quả là thông báo thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 2: Đăng nhập (Login)**

| | |
|---|---|
| **Input** | • Người dùng nhập Tên đăng nhập hoặc Email và Mật khẩu |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống xác định người dùng nhập Email hay Tên đăng nhập<br>• Hệ thống tìm kiếm người dùng trong database theo Email hoặc Tên đăng nhập<br>• Nếu không tìm thấy, hệ thống thông báo lỗi đăng nhập<br>• Nếu tìm thấy, hệ thống xác minh mật khẩu<br>• Nếu mật khẩu sai, hệ thống thông báo lỗi đăng nhập<br>• Nếu mật khẩu đúng, hệ thống kiểm tra email đã được xác thực chưa<br>• Nếu email chưa xác thực, hệ thống yêu cầu người dùng xác thực email trước<br>• Nếu email đã xác thực, hệ thống tạo token đăng nhập và trả về cho người dùng |
| **Output** | • Kết quả là token đăng nhập và thông báo thành công, hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 3: Xác thực email (Verify Email)**

| | |
|---|---|
| **Input** | • Người dùng nhập token xác thực từ link trong email |
| **Process** | • Hệ thống kiểm tra token có hợp lệ không<br>• Hệ thống tìm kiếm người dùng có token xác thực khớp và chưa hết hạn<br>• Nếu không tìm thấy hoặc token hết hạn, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống đánh dấu email đã được xác thực<br>• Hệ thống xóa token xác thực và thời hạn token<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo xác thực thành công |
| **Output** | • Kết quả là thông báo xác thực thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 4: Quên mật khẩu (Forgot Password)**

| | |
|---|---|
| **Input** | • Người dùng nhập Email |
| **Process** | • Hệ thống kiểm tra Email có hợp lệ không<br>• Hệ thống tìm kiếm người dùng trong database theo Email<br>• Nếu tìm thấy, hệ thống tạo token đặt lại mật khẩu và đặt thời hạn hết hạn<br>• Hệ thống lưu token vào database<br>• Hệ thống gửi email chứa link đặt lại mật khẩu cho người dùng (gửi bất đồng bộ)<br>• Hệ thống luôn trả về thông báo thành công (bảo mật, không tiết lộ email có tồn tại hay không) |
| **Output** | • Kết quả là thông báo thành công cho người dùng (nếu email tồn tại, link đặt lại mật khẩu đã được gửi) |

---

### **Tên chức năng 5: Đặt lại mật khẩu (Reset Password)**

| | |
|---|---|
| **Input** | • Người dùng nhập token đặt lại mật khẩu từ link trong email và mật khẩu mới |
| **Process** | • Hệ thống kiểm tra token và mật khẩu mới có hợp lệ không<br>• Hệ thống tìm kiếm người dùng có token đặt lại mật khẩu khớp và chưa hết hạn<br>• Nếu không tìm thấy hoặc token hết hạn, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống mã hóa mật khẩu mới<br>• Hệ thống cập nhật mật khẩu mới vào database<br>• Hệ thống xóa token đặt lại mật khẩu và thời hạn token<br>• Hệ thống thông báo đặt lại mật khẩu thành công |
| **Output** | • Kết quả là thông báo đặt lại mật khẩu thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 6: Thực hiện quyên góp (Donate)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin quyên góp: Số tiền, Nguyên nhân, Họ tên, Email, Số điện thoại (tùy chọn), Địa chỉ (tùy chọn), Phương thức thanh toán (tùy chọn), Chọn chương trình (tùy chọn), Chọn ẩn danh (tùy chọn), Đăng ký nhận tin (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra số tiền lớn hơn 0, nguyên nhân không rỗng, họ tên không rỗng, email không rỗng<br>• Hệ thống tạo mã giao dịch duy nhất cho quyên góp<br>• Nếu người dùng chọn ẩn danh, hệ thống đặt tên người quyên góp là "Anonymous"<br>• Hệ thống tạo bản ghi quyên góp với trạng thái thanh toán thành công<br>• Hệ thống lưu thông tin quyên góp vào database<br>• Nếu không phải ẩn danh và có email, hệ thống gửi email xác nhận quyên góp cho người dùng (gửi bất đồng bộ)<br>• Hệ thống trả về thông tin quyên góp đã tạo |
| **Output** | • Kết quả là thông tin quyên góp đã tạo hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 7: Xem danh sách chương trình (View Programs)**

| | |
|---|---|
| **Input** | • Người dùng truy cập trang danh sách chương trình |
| **Process** | • Hệ thống truy vấn database lấy tất cả chương trình<br>• Hệ thống sắp xếp danh sách chương trình<br>• Hệ thống hiển thị danh sách chương trình cho người dùng |
| **Output** | • Kết quả là danh sách các chương trình hiển thị cho người dùng |

---

### **Tên chức năng 8: Xem thống kê chương trình (View Program Stats)**

| | |
|---|---|
| **Input** | • Người dùng chọn chương trình muốn xem thống kê |
| **Process** | • Hệ thống lấy thông tin chương trình từ database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống tính tổng số tiền quyên góp cho chương trình<br>• Hệ thống tính phần trăm hoàn thành mục tiêu<br>• Hệ thống đếm số lượt đăng ký tham gia chương trình<br>• Hệ thống tính số tiền còn lại cần quyên góp (nếu có mục tiêu)<br>• Hệ thống hiển thị thống kê cho người dùng |
| **Output** | • Kết quả là thông tin thống kê chương trình hiển thị cho người dùng, hoặc thông báo lỗi nếu không tìm thấy chương trình |

---

### **Tên chức năng 9: Đăng ký tham gia chương trình (Register for Program)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Họ tên, Email, Số điện thoại (tùy chọn) và chọn chương trình muốn đăng ký |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống kiểm tra chương trình có tồn tại trong database không<br>• Nếu không tồn tại, hệ thống thông báo lỗi<br>• Nếu tồn tại, hệ thống kiểm tra người dùng đã đăng ký chương trình này chưa (nếu đã đăng nhập)<br>• Nếu đã đăng ký, hệ thống thông báo lỗi<br>• Nếu chưa đăng ký, hệ thống tạo bản ghi đăng ký mới<br>• Hệ thống lưu thông tin đăng ký vào database<br>• Hệ thống thông báo đăng ký thành công |
| **Output** | • Kết quả là thông báo đăng ký thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 10: Gửi câu hỏi/Yêu cầu (Submit Query)**

| | |
|---|---|
| **Input** | • Người dùng nhập thông tin: Tiêu đề, Nội dung câu hỏi, Email, Họ tên (tùy chọn) |
| **Process** | • Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo bản ghi câu hỏi mới<br>• Hệ thống lưu thông tin câu hỏi vào database<br>• Hệ thống trả về thông tin câu hỏi đã tạo |
| **Output** | • Kết quả là thông tin câu hỏi đã được gửi thành công |

---

### **Tên chức năng 11: Xem hồ sơ cá nhân (View Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập truy cập trang hồ sơ cá nhân |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Nếu token không hợp lệ, hệ thống từ chối truy cập<br>• Hệ thống truy vấn database lấy thông tin hồ sơ cá nhân của người dùng<br>• Nếu chưa có hồ sơ, hệ thống trả về thông tin rỗng<br>• Nếu có hồ sơ, hệ thống hiển thị thông tin hồ sơ cho người dùng |
| **Output** | • Kết quả là thông tin hồ sơ cá nhân hiển thị cho người dùng, hoặc thông báo lỗi nếu không có quyền truy cập |

---

### **Tên chức năng 12: Cập nhật hồ sơ cá nhân (Update Profile)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập nhập thông tin cần cập nhật: Họ tên, Số điện thoại, Địa chỉ, Ngày sinh, Giới tính (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống tìm kiếm hồ sơ cá nhân của người dùng trong database<br>• Nếu chưa có hồ sơ, hệ thống tạo hồ sơ mới<br>• Nếu đã có hồ sơ, hệ thống cập nhật các trường thông tin từ dữ liệu người dùng nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 13: Đổi mật khẩu (Change Password)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập nhập mật khẩu cũ và mật khẩu mới |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống tìm kiếm người dùng trong database<br>• Hệ thống xác minh mật khẩu cũ có đúng không<br>• Nếu mật khẩu cũ sai, hệ thống thông báo lỗi<br>• Nếu mật khẩu cũ đúng, hệ thống mã hóa mật khẩu mới<br>• Hệ thống cập nhật mật khẩu mới vào database<br>• Hệ thống thông báo đổi mật khẩu thành công |
| **Output** | • Kết quả là thông báo đổi mật khẩu thành công hoặc thông báo lỗi cho người dùng |

---

### **Tên chức năng 14: Xem lịch sử quyên góp (View Donation History)** - Chỉ Authenticated User

| | |
|---|---|
| **Input** | • Người dùng đã đăng nhập truy cập trang lịch sử quyên góp |
| **Process** | • Hệ thống kiểm tra token đăng nhập có hợp lệ không<br>• Hệ thống lấy thông tin người dùng từ token<br>• Hệ thống truy vấn database lấy tất cả quyên góp của người dùng<br>• Hệ thống sắp xếp danh sách quyên góp theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách quyên góp cho người dùng |
| **Output** | • Kết quả là danh sách các quyên góp của người dùng hiển thị trên trang |

---

## **ADMIN FUNCTIONS (Chức năng quản trị)**

### **Tên chức năng 1: Quản lý người dùng - Xem danh sách (Get All Users)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý người dùng |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Nếu không có quyền, hệ thống từ chối truy cập<br>• Hệ thống truy vấn database lấy tất cả người dùng<br>• Hệ thống hiển thị danh sách người dùng cho quản trị viên (không hiển thị mật khẩu) |
| **Output** | • Kết quả là danh sách tất cả người dùng hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 2: Quản lý người dùng - Xem chi tiết (Get User by ID)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng muốn xem chi tiết |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID người dùng<br>• Hệ thống truy vấn database tìm kiếm người dùng theo ID<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống hiển thị thông tin chi tiết người dùng cho quản trị viên |
| **Output** | • Kết quả là thông tin chi tiết người dùng hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không tìm thấy hoặc không có quyền |

---

### **Tên chức năng 3: Quản lý người dùng - Cập nhật vai trò (Update User Role)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng và chọn vai trò mới (User hoặc Admin) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra vai trò có hợp lệ không<br>• Hệ thống lấy ID người dùng và vai trò mới<br>• Hệ thống tìm kiếm người dùng trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật vai trò mới<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật vai trò thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 4: Quản lý người dùng - Xóa người dùng (Delete User)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn người dùng muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID người dùng<br>• Hệ thống tìm kiếm người dùng trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa người dùng khỏi database (có thể xóa kèm các bản ghi liên quan như quyên góp, câu hỏi)<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 5: Quản lý quyên góp - Xem tất cả (Get All Donations)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý quyên góp |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống truy vấn database lấy tất cả quyên góp (kèm thông tin người dùng nếu có)<br>• Hệ thống sắp xếp danh sách quyên góp theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách quyên góp cho quản trị viên |
| **Output** | • Kết quả là danh sách tất cả quyên góp hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 6: Quản lý quyên góp - Xem chi tiết (Get Donation by ID)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn quyên góp muốn xem chi tiết |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID quyên góp<br>• Hệ thống truy vấn database tìm kiếm quyên góp theo ID (kèm thông tin người dùng)<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống hiển thị thông tin chi tiết quyên góp cho quản trị viên |
| **Output** | • Kết quả là thông tin chi tiết quyên góp hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không tìm thấy hoặc không có quyền |

---

### **Tên chức năng 7: Quản lý chương trình - Tạo mới (Create Program)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin chương trình: Tiêu đề, Mô tả, Ngày bắt đầu (tùy chọn), Ngày kết thúc (tùy chọn), Địa điểm (tùy chọn), Mục tiêu số tiền (tùy chọn), Chọn tổ chức NGO (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo chương trình mới<br>• Hệ thống lưu thông tin chương trình vào database<br>• Hệ thống trả về thông tin chương trình đã tạo |
| **Output** | • Kết quả là thông tin chương trình đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 8: Quản lý chương trình - Cập nhật (Update Program)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn chương trình và nhập thông tin cần cập nhật: Tiêu đề, Mô tả, Ngày bắt đầu, Ngày kết thúc, Địa điểm, Mục tiêu số tiền, Tổ chức NGO (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID chương trình<br>• Hệ thống tìm kiếm chương trình trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 9: Quản lý chương trình - Xóa (Delete Program)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn chương trình muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID chương trình<br>• Hệ thống tìm kiếm chương trình trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa chương trình khỏi database (có thể xóa kèm các bản ghi liên quan như đăng ký tham gia, quyên góp, ảnh gallery)<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 10: Quản lý NGO - Tạo mới (Create NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin tổ chức NGO: Tên tổ chức, Mô tả (tùy chọn), URL logo (tùy chọn), Website (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo tổ chức NGO mới<br>• Hệ thống lưu thông tin tổ chức NGO vào database<br>• Hệ thống trả về thông tin tổ chức NGO đã tạo |
| **Output** | • Kết quả là thông tin tổ chức NGO đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 11: Quản lý NGO - Cập nhật (Update NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn tổ chức NGO và nhập thông tin cần cập nhật: Tên tổ chức, Mô tả, URL logo, Website (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID tổ chức NGO<br>• Hệ thống tìm kiếm tổ chức NGO trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 12: Quản lý NGO - Xóa (Delete NGO)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn tổ chức NGO muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID tổ chức NGO<br>• Hệ thống tìm kiếm tổ chức NGO trong database<br>• Hệ thống kiểm tra tổ chức NGO có đang được sử dụng bởi chương trình nào không<br>• Nếu đang được sử dụng, hệ thống thông báo lỗi và không cho phép xóa<br>• Nếu không được sử dụng, hệ thống xóa tổ chức NGO khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 13: Quản lý Gallery - Thêm ảnh (Create Gallery Item)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn file ảnh hoặc nhập URL ảnh, nhập chú thích (tùy chọn), chọn chương trình liên quan (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra phải có file ảnh hoặc URL ảnh (ít nhất một trong hai)<br>• Nếu có file ảnh, hệ thống kiểm tra loại file (chỉ cho phép ảnh), kiểm tra kích thước file (tối đa 5MB)<br>• Nếu file hợp lệ, hệ thống lưu file vào thư mục và tạo URL ảnh<br>• Nếu có URL ảnh, hệ thống sử dụng URL trực tiếp<br>• Hệ thống tạo bản ghi ảnh mới<br>• Hệ thống lưu thông tin ảnh vào database<br>• Hệ thống trả về thông tin ảnh đã tạo |
| **Output** | • Kết quả là thông tin ảnh đã tạo hoặc thông báo lỗi cho quản trị viên (lỗi loại file, kích thước file, hoặc không có quyền) |

---

### **Tên chức năng 14: Quản lý Gallery - Xóa ảnh (Delete Gallery Item)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn ảnh muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID ảnh<br>• Hệ thống tìm kiếm ảnh trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa file ảnh khỏi thư mục (nếu là file upload)<br>• Hệ thống xóa bản ghi ảnh khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 15: Quản lý Đối tác - Tạo mới (Create Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin đối tác: Tên đối tác, URL logo (tùy chọn), Website (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo đối tác mới<br>• Hệ thống lưu thông tin đối tác vào database<br>• Hệ thống trả về thông tin đối tác đã tạo |
| **Output** | • Kết quả là thông tin đối tác đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 16: Quản lý Đối tác - Cập nhật (Update Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn đối tác và nhập thông tin cần cập nhật: Tên đối tác, URL logo, Website (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID đối tác<br>• Hệ thống tìm kiếm đối tác trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 17: Quản lý Đối tác - Xóa (Delete Partner)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn đối tác muốn xóa và xác nhận xóa |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống lấy ID đối tác<br>• Hệ thống tìm kiếm đối tác trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống xóa đối tác khỏi database<br>• Hệ thống thông báo xóa thành công |
| **Output** | • Kết quả là thông báo xóa thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 18: Quản lý Nội dung Giới thiệu - Tạo mới (Create About Section)**

| | |
|---|---|
| **Input** | • Quản trị viên nhập thông tin phần nội dung: Khóa (duy nhất), Tiêu đề, Nội dung, Dữ liệu bổ sung (tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống tạo phần nội dung mới<br>• Hệ thống lưu thông tin phần nội dung vào database<br>• Hệ thống trả về thông tin phần nội dung đã tạo |
| **Output** | • Kết quả là thông tin phần nội dung đã tạo hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 19: Quản lý Nội dung Giới thiệu - Cập nhật (Update About Section)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn phần nội dung và nhập thông tin cần cập nhật: Tiêu đề, Nội dung, Dữ liệu bổ sung (tất cả đều tùy chọn) |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra dữ liệu đầu vào có hợp lệ không<br>• Hệ thống lấy ID phần nội dung<br>• Hệ thống tìm kiếm phần nội dung trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật các trường thông tin từ dữ liệu quản trị viên nhập<br>• Hệ thống lưu thay đổi vào database<br>• Hệ thống thông báo cập nhật thành công |
| **Output** | • Kết quả là thông báo cập nhật thành công hoặc thông báo lỗi cho quản trị viên |

---

### **Tên chức năng 20: Quản lý Câu hỏi - Xem tất cả (Get All Queries)**

| | |
|---|---|
| **Input** | • Quản trị viên truy cập trang quản lý câu hỏi |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống truy vấn database lấy tất cả câu hỏi (kèm thông tin người dùng nếu có)<br>• Hệ thống sắp xếp danh sách câu hỏi theo thời gian (mới nhất trước)<br>• Hệ thống hiển thị danh sách câu hỏi cho quản trị viên |
| **Output** | • Kết quả là danh sách tất cả câu hỏi hiển thị cho quản trị viên, hoặc thông báo lỗi nếu không có quyền |

---

### **Tên chức năng 21: Quản lý Câu hỏi - Trả lời (Reply to Query)**

| | |
|---|---|
| **Input** | • Quản trị viên chọn câu hỏi và nhập nội dung trả lời |
| **Process** | • Hệ thống kiểm tra quản trị viên đã đăng nhập và có quyền Admin không<br>• Hệ thống kiểm tra nội dung trả lời có hợp lệ không<br>• Hệ thống lấy ID câu hỏi<br>• Hệ thống tìm kiếm câu hỏi trong database<br>• Nếu không tìm thấy, hệ thống thông báo lỗi<br>• Nếu tìm thấy, hệ thống cập nhật nội dung trả lời và ngày trả lời<br>• Hệ thống lưu thay đổi vào database<br>• Nếu câu hỏi có người dùng và người dùng có email, hệ thống gửi email phản hồi cho người dùng (gửi bất đồng bộ)<br>• Hệ thống thông báo trả lời thành công |
| **Output** | • Kết quả là thông báo trả lời thành công hoặc thông báo lỗi cho quản trị viên |

---

## **Ghi chú**

- Tất cả các chức năng yêu cầu đăng nhập đều sử dụng token đăng nhập trong header
- Các chức năng Admin yêu cầu người dùng có vai trò Admin
- Email được gửi bất đồng bộ để không làm chậm phản hồi của hệ thống
- Tất cả mật khẩu được mã hóa trước khi lưu vào database
- Token xác thực email và đặt lại mật khẩu có thời hạn sử dụng
