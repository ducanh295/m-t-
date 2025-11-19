# Give-AID - Tài Liệu Kỹ Thuật Cho Developers

> **Đối tượng**: Developers mới vào team  
> **Cập nhật lần cuối**: Tháng 11/2024  
> **Ngôn ngữ**: Tiếng Việt (với thuật ngữ tiếng Anh)

---

## Mục Lục

- [1. Tổng Quan Kiến Trúc Hệ Thống](#1-tổng-quan-kiến-trúc-hệ-thống)
- [2. Cấu Trúc Thư Mục & File Quan Trọng](#2-cấu-trúc-thư-mục--file-quan-trọng)
- [3. Backend Deep Dive](#3-backend-deep-dive)
- [4. Frontend Deep Dive](#4-frontend-deep-dive)
- [5. Database Schema & Migrations](#5-database-schema--migrations)
- [6. Authentication & Authorization Flow](#6-authentication--authorization-flow)
- [7. Luồng Xử Lý Các Tính Năng Chính](#7-luồng-xử-lý-các-tính-năng-chính)
- [8. Email System](#8-email-system)
- [9. Hướng Dẫn Debugging](#9-hướng-dẫn-debugging)
- [10. Best Practices & Onboarding Checklist](#10-best-practices--onboarding-checklist)

---

## 1. Tổng Quan Kiến Trúc Hệ Thống

### 1.1 Sơ Đồ Kiến Trúc Tổng Quan

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         React Application (Port 5173)                    │  │
│  │  - React 18.3 với Hooks                                  │  │
│  │  - React Router DOM 7.0                                  │  │
│  │  - Axios 1.7 (HTTP Client)                               │  │
│  │  - Bootstrap 5 (UI Framework)                            │  │
│  │  - localStorage (JWT Token Storage)                      │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS (JSON + JWT Token)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         API LAYER                               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │      ASP.NET Core 8.0 Web API (Port 5230)                │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │  Middleware Pipeline:                              │  │  │
│  │  │  1. CORS (Allow Frontend)                          │  │  │
│  │  │  2. Authentication (JWT Bearer)                    │  │  │
│  │  │  3. Authorization (Role-based)                     │  │  │
│  │  │  4. Routing → Controllers                          │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │  Controllers → Services → DbContext               │  │  │
│  │  │  - AuthController                                  │  │  │
│  │  │  - DonationController                              │  │  │
│  │  │  - ProgramController                               │  │  │
│  │  │  - AdminController                                 │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Entity Framework Core 9.0
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATABASE LAYER                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         SQL Server Database                              │  │
│  │  - Users, Donations, Programs, NGOs, ...                 │  │
│  │  - Relationships & Constraints                           │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ SMTP (Fire-and-forget)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICES                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  - SMTP Server (Gmail/Outlook)                          │  │
│  │    → Send verification emails                            │  │
│  │    → Send donation receipts                              │  │
│  │    → Send password reset links                           │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Luồng Request Chi Tiết (Sequence Diagram)

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│   User   │      │ Frontend │      │  Backend │      │Database  │      │   SMTP   │
└────┬─────┘      └────┬─────┘      └────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                  │                  │                  │
     │ (1) Click       │                  │                  │                  │
     │  "Donate"       │                  │                  │                  │
     ├────────────────>│                  │                  │                  │
     │                 │                  │                  │                  │
     │                 │ (2) Form Submit  │                  │                  │
     │                 │  (Validate)      │                  │                  │
     │                 │                  │                  │                  │
     │                 │ (3) POST /api/   │                  │                  │
     │                 │  donation        │                  │                  │
     │                 │  + JWT Token     │                  │                  │
     │                 ├─────────────────>│                  │                  │
     │                 │                  │                  │                  │
     │                 │                  │ (4) Validate JWT │                  │
     │                 │                  │  (Check token)   │                  │
     │                 │                  ├─────────────────>│                  │
     │                 │                  │  (Verify user)   │                  │
     │                 │                  │<─────────────────┤                  │
     │                 │                  │                  │                  │
     │                 │                  │ (5) Validate DTO │                  │
     │                 │                  │  (ModelState)    │                  │
     │                 │                  │                  │                  │
     │                 │                  │ (6) Business     │                  │
     │                 │                  │  Logic           │                  │
     │                 │                  │  (DonationService│                  │
     │                 │                  │   .CreateAsync)  │                  │
     │                 │                  │                  │                  │
     │                 │                  │ (7) Save to DB   │                  │
     │                 │                  ├─────────────────>│                  │
     │                 │                  │  INSERT INTO     │                  │
     │                 │                  │  Donations       │                  │
     │                 │                  │<─────────────────┤                  │
     │                 │                  │  (Saved, ID=42)  │                  │
     │                 │                  │                  │                  │
     │                 │                  │ (8) Send Email   │                  │
     │                 │                  │  (Fire & Forget) │                  │
     │                 │                  ├────────────────────────────────────>│
     │                 │                  │  (Email sent)    │                  │
     │                 │                  │                  │                  │
     │                 │                  │ (9) Return 200   │                  │
     │                 │                  │  OK + Donation   │                  │
     │                 │<─────────────────┤                  │                  │
     │                 │  JSON Response   │                  │                  │
     │                 │                  │                  │                  │
     │                 │ (10) Update UI   │                  │                  │
     │                 │  (Show toast,    │                  │                  │
     │                 │   navigate)      │                  │                  │
     │<────────────────┤                  │                  │                  │
     │ Success Message │                  │                  │                  │
```

### 1.3 Technology Stack

**Backend:**
- **Framework**: ASP.NET Core 8.0
- **Language**: C# 12.0
- **ORM**: Entity Framework Core 9.0
- **Database**: SQL Server (LocalDB / SQL Server Express / Full SQL Server)
- **Authentication**: JWT Bearer Tokens (HS256)
- **Email**: SmtpClient (System.Net.Mail)
- **Documentation**: Swagger/OpenAPI (Swashbuckle)

**Frontend:**
- **Framework**: React 18.3 (Functional Components + Hooks)
- **Build Tool**: Vite 6.0
- **Router**: React Router DOM 7.0
- **HTTP Client**: Axios 1.7
- **UI Framework**: Bootstrap 5.3
- **Icons**: Font Awesome 6.0
- **Animations**: AOS (Animate On Scroll) 3.0
- **Notifications**: 
  - React Toastify 11.0 (Toast notifications)
  - SweetAlert2 11.15 (Modal dialogs)

**Development Tools:**
- **IDE**: Visual Studio 2022 / VS Code
- **Version Control**: Git
- **API Testing**: Swagger UI, Postman
- **Database Management**: SQL Server Management Studio (SSMS)

---

## 2. Cấu Trúc Thư Mục & File Quan Trọng

### 2.1 Backend Structure (Chi Tiết)

```
Backend/
│
├── Program.cs                                    # Dòng 1-117
│   └── Entry point: DI container, middleware, startup
│
├── appsettings.json                              # Cấu hình mặc định (commit vào Git)
├── appsettings.example.json                      # Template cấu hình (commit vào Git)
├── appsettings.Development.json                  # Cấu hình cá nhân (GITIGNORED)
│
├── Controllers/                                  # API Endpoints (HTTP handlers)
│   ├── AuthController.cs                        # Dòng 1-205
│   │   └── /api/auth/* endpoints
│   ├── DonationController.cs                    # Dòng 1-130
│   │   └── /api/donation/* endpoints
│   ├── ProgramController.cs                     # Dòng 1-xxx
│   │   └── /api/program/* endpoints
│   ├── AdminController.cs                       # Dòng 1-xxx
│   │   └── /api/admin/* endpoints (Admin only)
│   ├── ProfileController.cs                     # Dòng 1-xxx
│   │   └── /api/profile/* endpoints (User only)
│   ├── NGOController.cs                         # Dòng 1-xxx
│   │   └── /api/ngo/* endpoints
│   ├── PartnerController.cs                     # Dòng 1-xxx
│   │   └── /api/partner/* endpoints
│   ├── GalleryController.cs                     # Dòng 1-xxx
│   │   └── /api/gallery/* endpoints
│   ├── QueryController.cs                       # Dòng 1-xxx
│   │   └── /api/query/* endpoints
│   ├── AboutController.cs                       # Dòng 1-xxx
│   │   └── /api/about/* endpoints
│   └── InvitationController.cs                  # Dòng 1-xxx
│       └── /api/invitation/* endpoints
│
├── Services/                                     # Business Logic Layer
│   ├── AuthService.cs                           # Dòng 1-527
│   │   └── Registration, login, email verification, password reset
│   ├── DonationService.cs                       # Dòng 1-190
│   │   └── Donation processing, email confirmation
│   ├── ProgramService.cs                        # Dòng 1-xxx
│   │   └── Program management, stats, registration
│   ├── EmailService.cs                          # Dòng 1-xxx
│   │   └── SMTP email sending
│   ├── UserService.cs                           # Dòng 1-xxx
│   │   └── User CRUD operations
│   ├── NGOService.cs                            # Dòng 1-xxx
│   ├── PartnerService.cs                        # Dòng 1-xxx
│   ├── GalleryService.cs                        # Dòng 1-xxx
│   ├── QueryService.cs                          # Dòng 1-xxx
│   ├── AboutService.cs                          # Dòng 1-xxx
│   ├── ProfileService.cs                        # Dòng 1-xxx
│   └── InvitationService.cs                     # Dòng 1-xxx
│
├── Models/                                       # Database Entities (EF Core)
│   ├── User.cs                                  # Dòng 1-45
│   │   └── Users table entity
│   ├── Donation.cs                              # Dòng 1-xxx
│   │   └── Donations table entity
│   ├── NgoProgram.cs                            # Dòng 1-xxx
│   │   └── NgoPrograms table entity
│   ├── NGO.cs                                   # Dòng 1-xxx
│   ├── Partner.cs                               # Dòng 1-xxx
│   ├── Gallery.cs                               # Dòng 1-xxx
│   ├── Query.cs                                 # Dòng 1-xxx
│   ├── ProgramRegistration.cs                   # Dòng 1-xxx
│   ├── Invitation.cs                            # Dòng 1-xxx
│   ├── Cause.cs                                 # Dòng 1-xxx
│   ├── AboutSection.cs                          # Dòng 1-xxx
│   └── UserProfile.cs                           # Dòng 1-xxx
│
├── DTOs/                                         # Data Transfer Objects (API contracts)
│   ├── DonationDTO.cs                           # Dòng 1-xxx
│   │   └── Request/Response cho donation
│   ├── LoginRequest.cs                          # Dòng 1-xxx
│   │   └── Login payload
│   ├── RegisterRequest.cs                       # Dòng 1-xxx
│   │   └── Registration payload
│   └── ... (các DTO khác)
│
├── Data/
│   └── GiveAidContext.cs                        # Dòng 1-54
│       └── EF Core DbContext, entity configuration
│
├── Helpers/                                      # Utility Classes
│   ├── JwtHelper.cs                             # Dòng 1-xxx
│   │   └── JWT token generation & validation
│   ├── PasswordHasher.cs                        # Dòng 1-xxx
│   │   └── BCrypt password hashing
│   ├── EmailTemplate.cs                         # Dòng 1-302
│   │   └── HTML email templates
│   ├── DataSeeder.cs                            # Dòng 1-309
│   │   └── Auto-seed admin user & initial data
│   └── PaymentValidator.cs                      # Dòng 1-xxx
│       └── Luhn algorithm for card validation
│
├── Migrations/                                   # EF Core Database Migrations
│   ├── 20251021161050_InitialCreate.cs
│   │   └── Tạo tất cả tables ban đầu
│   ├── 20251031044158_AddUsernameToUser.cs
│   ├── 20251031092945_AddEmailVerificationToUser.cs
│   ├── 20251101120000_AddPasswordResetToUser.cs
│   ├── 20251103131053_AddProgramRegistration.cs
│   └── 20251105155757_AddProgramGoalAndDonationLink.cs
│
└── Scripts/                                      # SQL Scripts (optional)
    └── README_CREATE_ADMIN.md
```

### 2.2 Top 10 Files Backend Quan Trọng Nhất

**Thứ tự ưu tiên đọc cho dev mới:**

1. **`Program.cs` (Dòng 1-117)**
   - Entry point, DI container setup, middleware pipeline
   - JWT authentication config, CORS config
   - Data seeding on startup

2. **`Controllers/AuthController.cs` (Dòng 1-205)**
   - Tất cả endpoints liên quan authentication
   - Register, login, verify email, password reset

3. **`Services/AuthService.cs` (Dòng 1-527)**
   - Business logic cho authentication
   - Password hashing, token generation, email verification

4. **`Controllers/DonationController.cs` (Dòng 1-130)**
   - Donation endpoints
   - Create donation, get user donations, admin get all

5. **`Services/DonationService.cs` (Dòng 1-190)**
   - Donation processing logic
   - Email confirmation sending

6. **`Data/GiveAidContext.cs` (Dòng 1-54)**
   - EF Core DbContext
   - Entity relationships, constraints

7. **`Helpers/JwtHelper.cs`**
   - JWT token generation & validation
   - Claims mapping

8. **`Helpers/DataSeeder.cs` (Dòng 1-309)**
   - Auto-create admin user
   - Seed NGOs and Programs

9. **`Models/User.cs` (Dòng 1-45)**
   - User entity structure
   - Properties, relationships

10. **`Models/Donation.cs`**
    - Donation entity structure
    - Relationships với User và Program

### 2.3 Frontend Structure (Chi Tiết)

```
FRONTEND/
│
├── index.html                                    # HTML template
├── package.json                                  # Dòng 1-35: Dependencies & scripts
├── vite.config.js                               # Vite configuration
│
└── src/
    ├── main.jsx                                 # Entry point: React app mounting
    │
    ├── App.jsx                                  # Dòng 1-120
    │   └── Routes, providers, global setup
    │
    ├── components/
    │   ├── layout/
    │   │   ├── Layout.jsx                       # Main layout wrapper (Navbar + Footer)
    │   │   ├── Navbar.jsx                       # Dòng 1-146: Navigation bar với auth state
    │   │   └── Footer.jsx                       # Footer component
    │   │
    │   ├── admin/
    │   │   ├── AdminLayout.jsx                  # Admin panel layout
    │   │   └── AdminRoute.jsx                   # Admin route guard (check role)
    │   │
    │   └── ScrollToTop.jsx                      # Scroll restoration on route change
    │
    ├── pages/
    │   ├── HomePage.jsx                         # Landing page
    │   ├── DonatePage.jsx                       # Dòng 1-767
    │   │   └── Donation form + program selection
    │   ├── LoginPage.jsx                        # Dòng 1-xxx
    │   │   └── Login form
    │   ├── RegisterPage.jsx                     # Dòng 1-xxx
    │   │   └── Registration form
    │   ├── DonationHistoryPage.jsx              # Dòng 1-352
    │   │   └── User donation history
    │   ├── ProgramsPage.jsx                     # Dòng 1-xxx
    │   │   └── Programs listing + registration
    │   ├── NGOsPage.jsx                         # Dòng 1-xxx
    │   ├── PartnersPage.jsx                     # Dòng 1-xxx
    │   ├── GalleryPage.jsx                      # Dòng 1-xxx
    │   ├── AboutPage.jsx                        # Dòng 1-xxx
    │   ├── ContactPage.jsx                      # Dòng 1-xxx
    │   ├── HelpPage.jsx                         # Dòng 1-xxx
    │   ├── ProfilePage.jsx                      # Dòng 1-xxx
    │   ├── VerifyEmailPage.jsx                  # Dòng 1-xxx
    │   ├── ForgotPasswordPage.jsx               # Dòng 1-xxx
    │   ├── ResetPasswordPage.jsx                # Dòng 1-xxx
    │   │
    │   └── admin/
    │       ├── UsersPage.jsx                    # Admin: User management
    │       ├── DonationsPage.jsx                # Admin: Donations dashboard
    │       ├── ProgramsPage.jsx                 # Admin: Programs management
    │       ├── NGOsPage.jsx                     # Admin: NGOs management
    │       ├── PartnersPage.jsx                 # Admin: Partners management
    │       ├── GalleryPage.jsx                  # Admin: Gallery management
    │       ├── AboutPage.jsx                    # Admin: About sections management
    │       └── QueriesPage.jsx                  # Admin: Queries management
    │
    ├── services/                                # API Service Layer
    │   ├── api.js                              # Dòng 1-xxx
    │   │   └── Axios instance + JWT interceptor
    │   ├── authServices.js                     # Dòng 1-xxx
    │   │   └── Auth API calls (login, register, verify, etc.)
    │   ├── donationServices.js                 # Dòng 1-xxx
    │   │   └── Donation API calls
    │   ├── programServices.js                  # Dòng 1-xxx
    │   │   └── Program API calls
    │   ├── adminServices.js                    # Dòng 1-xxx
    │   │   └── Admin API calls
    │   ├── ngoServices.js                      # Dòng 1-xxx
    │   ├── partnerServices.js                  # Dòng 1-xxx
    │   ├── galleryServices.js                  # Dòng 1-xxx
    │   ├── contactServices.js                  # Dòng 1-xxx
    │   ├── profileServices.js                  # Dòng 1-xxx
    │   └── queryServices.js                    # Dòng 1-xxx
    │
    ├── contexts/
    │   └── AuthContext.jsx                     # Dòng 1-167
    │       └── Global auth state (user, token, login, logout)
    │
    ├── assets/
    │   └── css/
    │       ├── style.css                       # Dòng 1-1335: Global styles
    │       └── donate.css                      # Dòng 1-384: Donation page styles
    │
    └── utilis/
        ├── helpers.js                          # Utility functions
        └── useCounter.js                       # Counter animation hook
```

### 2.4 Top 10 Files Frontend Quan Trọng Nhất

1. **`main.jsx`**
   - Entry point: React app mounting
   - Import CSS, render App

2. **`App.jsx` (Dòng 1-120)**
   - Routes configuration
   - AuthProvider, ToastContainer
   - AOS initialization

3. **`contexts/AuthContext.jsx` (Dòng 1-167)**
   - Global auth state management
   - JWT token handling, user state

4. **`services/api.js`**
   - Axios instance với JWT interceptor
   - Base URL, error handling

5. **`services/authServices.js`**
   - Login, register, verify email API calls
   - Password reset API calls

6. **`pages/DonatePage.jsx` (Dòng 1-767)**
   - Donation form với program selection
   - Form validation, API call

7. **`pages/LoginPage.jsx`**
   - Login form
   - Integration với AuthContext

8. **`components/layout/Navbar.jsx` (Dòng 1-146)**
   - Navigation với auth state
   - User menu, admin links

9. **`components/admin/AdminRoute.jsx`**
   - Admin route guard
   - Check authentication & role

10. **`services/donationServices.js`**
    - Donation API calls
    - Create donation, get history

---

## 3. Chi Tiết Backend (Phần Máy Chủ)

### 3.1 Program.cs - File Khởi Động Ứng Dụng

**File**: `Backend/Program.cs` **Dòng 1-117**

**Vai trò**: File này là điểm khởi đầu của ứng dụng Backend. Nó cấu hình tất cả các thành phần cần thiết: database, authentication (xác thực), CORS, và các service.

#### A. Cấu Hình JSON Serialization (Dòng 12-20)

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Chuyển đổi từ PascalCase (C#) sang camelCase (JSON)
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        
        // Không phân biệt hoa thường khi đọc property
        options.JsonSerializerOptions.PropertyNameCaseInsensitive = true;
        
        // Bỏ qua circular references (tham chiếu vòng tròn) để tránh lỗi vòng lặp vô tận
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    });
```

**Giải thích:**
- **JSON Serialization**: Quá trình chuyển đổi object C# thành định dạng JSON để gửi cho Frontend
- **PascalCase vs camelCase**: 
  - C# dùng PascalCase: `UserName`, `EmailAddress`
  - JavaScript/JSON thường dùng camelCase: `userName`, `emailAddress`
  - Cấu hình này tự động chuyển đổi giữa 2 định dạng
- **Circular References**: Khi một object tham chiếu đến object khác, và object kia lại tham chiếu lại object đầu tiên → tạo vòng lặp vô tận. Ví dụ: `Donation.User.Donations.User.Donations...` → Cấu hình này bỏ qua để tránh lỗi

#### B. Đăng Ký Database Connection (Dòng 24-27)

```csharp
var conn = builder.Configuration.GetConnectionString("DefaultConnection")
           ?? "Server=(localdb)\\MSSQLLocalDB;Database=GiveAidDB;Trusted_Connection=True;";

builder.Services.AddDbContext<GiveAidContext>(options => options.UseSqlServer(conn));
```

**Giải thích:**
- **Connection String**: Chuỗi kết nối đến database, chứa thông tin: server name, database name, username, password
- **Configuration**: Đọc từ file `appsettings.Development.json` (nếu có) hoặc dùng giá trị mặc định (LocalDB)
- **AddDbContext**: Đăng ký `GiveAidContext` (class quản lý database) vào hệ thống Dependency Injection (DI - Hệ thống tự động tạo đối tượng khi cần)
- **Scoped Lifetime**: Mỗi HTTP request sẽ có 1 instance (bản sao) riêng của DbContext

#### C. Đăng Ký Các Service (Dòng 29-41)

```csharp
builder.Services.AddScoped<AuthService>();
builder.Services.AddScoped<DonationService>();
builder.Services.AddScoped<ProgramService>();
// ... tất cả các service khác
```

**Giải thích:**
- **Service**: Class chứa logic nghiệp vụ (business logic). Ví dụ: `AuthService` chứa logic đăng nhập, đăng ký
- **AddScoped**: Đăng ký service với lifetime (thời gian sống) là "Scoped"
  - **Scoped**: Tạo 1 instance cho mỗi HTTP request, instance này sẽ được dùng xuyên suốt request đó
  - **Singleton**: Tạo 1 instance duy nhất cho toàn bộ ứng dụng (dùng cho service không có state - trạng thái)
  - **Transient**: Tạo instance mới mỗi lần cần dùng (dùng ít)
- **Tại sao dùng Scoped**: 
  - Mỗi request độc lập, không ảnh hưởng lẫn nhau
  - DbContext cũng là Scoped → Service và DbContext cùng scope, dễ quản lý

#### D. Cấu Hình JWT Authentication (Dòng 43-66)

```csharp
var jwtKey = builder.Configuration["Jwt:Key"] ?? "very_secret_key_change_me_please";
var jwtIssuer = builder.Configuration["Jwt:Issuer"] ?? "Give_AID";
var key = Encoding.ASCII.GetBytes(jwtKey);

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false; // Trong production nên đặt true
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,              // Kiểm tra Issuer (người phát hành token)
        ValidIssuer = jwtIssuer,            // Issuer phải là "Give_AID"
        ValidateAudience = false,           // Không kiểm tra Audience
        ValidateIssuerSigningKey = true,    // Kiểm tra chữ ký (signature)
        IssuerSigningKey = new SymmetricSecurityKey(key),  // Dùng HS256 algorithm
        ValidateLifetime = true,            // Kiểm tra token đã hết hạn chưa
    };
});
```

**Giải thích:**
- **JWT (JSON Web Token)**: Mã thông báo dạng JSON, dùng để xác thực người dùng
- **Authentication Scheme**: Phương thức xác thực, ở đây dùng JWT Bearer (Bearer là cách gửi token trong header)
- **Token Validation**: Quá trình kiểm tra token có hợp lệ không:
  1. **ValidateIssuer**: Kiểm tra token được phát hành bởi ai (phải là "Give_AID")
  2. **ValidateIssuerSigningKey**: Kiểm tra chữ ký (signature) của token bằng secret key
  3. **ValidateLifetime**: Kiểm tra token chưa hết hạn
- **Flow**: Khi Frontend gửi request với header `Authorization: Bearer {token}`, middleware (phần mềm trung gian) tự động kiểm tra token. Nếu hợp lệ, set `HttpContext.User` với thông tin user (id, email, role)

#### E. Cấu Hình CORS (Dòng 71-80)

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:5173", "http://localhost:3000", "http://localhost:5174")
              .AllowAnyHeader()      // Cho phép mọi header
              .AllowAnyMethod()      // Cho phép mọi method (GET, POST, PUT, DELETE)
              .AllowCredentials();   // Cho phép gửi credentials (cookies, authorization headers)
    });
});
```

**Giải thích:**
- **CORS (Cross-Origin Resource Sharing)**: Cơ chế cho phép website ở domain này gọi API ở domain khác
- **Vấn đề**: Trình duyệt có chính sách bảo mật "Same-Origin Policy" (chính sách cùng nguồn gốc), chỉ cho phép gọi API cùng domain. Frontend chạy ở `localhost:5173`, Backend chạy ở `localhost:5230` → khác domain → cần CORS
- **WithOrigins**: Cho phép các domain cụ thể gọi API (Frontend dev server)
- **AllowCredentials**: Cho phép gửi credentials (JWT token trong header)

#### F. Data Seeding (Tạo Dữ Liệu Mẫu) (Dòng 84-103)

```csharp
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    var context = services.GetRequiredService<GiveAidContext>();
    var configuration = services.GetRequiredService<IConfiguration>();
    try
    {
        // Tạo admin user mặc định
        await Backend.Helpers.DataSeeder.SeedAdminUserAsync(context, configuration);
        
        // Tạo dữ liệu NGOs và Programs (chỉ nếu database trống)
        await Backend.Helpers.DataSeeder.SeedNGOsAndProgramsAsync(context);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error seeding data: {ex.Message}");
    }
}
```

**Giải thích:**
- **Data Seeding**: Tự động tạo dữ liệu mẫu khi ứng dụng khởi động
- **Scope**: Tạo một scope (phạm vi) mới để lấy services (vì đang ở ngoài HTTP request)
- **SeedAdminUserAsync**: Tạo admin user mặc định (email: admin@giveaid.org, password: Admin123!)
- **SeedNGOsAndProgramsAsync**: Tạo dữ liệu NGOs và Programs mẫu (chỉ tạo nếu database trống)

### 3.2 Mẫu Làm Việc: Controller → Service → Database

**Kiến trúc 3 lớp (3-Layer Architecture):**

```
HTTP Request → Controller (Nhận yêu cầu) → Service (Xử lý logic) → Database (Lưu trữ)
```

**Ví dụ: Tạo Donation**

**Bước 1: Controller nhận request**

**File**: `Backend/Controllers/DonationController.cs` **Dòng 22-83**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] DonationDTO? dto)
{
    // Kiểm tra DTO có null không
    if (dto == null)
    {
        return BadRequest(new { message = "Request body is required" });
    }

    // Kiểm tra validation (ModelState)
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value?.Errors.Count > 0)
            .Select(x => new { Field = x.Key, Errors = x.Value?.Errors.Select(e => e.ErrorMessage) })
            .ToList();
        
        return BadRequest(new { message = "Validation failed", errors });
    }
    
    // Kiểm tra thủ công các trường quan trọng
    if (dto.Amount <= 0)
    {
        return BadRequest(new { message = "Amount must be greater than 0" });
    }

    try
    {
        // Gọi Service để xử lý logic
        var donation = await _donationService.CreateAsync(dto);
        return Ok(donation);
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = "Failed to create donation", error = ex.Message });
    }
}
```

**Giải thích:**
- **Controller**: Nhận HTTP request từ Frontend, kiểm tra validation (xác thực dữ liệu), gọi Service để xử lý
- **FromBody**: Lấy dữ liệu từ body của HTTP request (JSON)
- **ModelState.IsValid**: Kiểm tra dữ liệu có thỏa mãn các ràng buộc (constraints) không (ví dụ: Email phải đúng định dạng, Amount phải > 0)
- **BadRequest**: Trả về lỗi 400 (Bad Request - Yêu cầu không hợp lệ)
- **Ok**: Trả về thành công 200 (OK) kèm dữ liệu

**Bước 2: Service xử lý logic nghiệp vụ**

**File**: `Backend/Services/DonationService.cs` **Dòng 24-135**

```csharp
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    // Chuẩn bị dữ liệu
    var donation = new Donation
    {
        Amount = dto.Amount,
        CauseName = dto.Cause ?? "General Donation",
        PaymentStatus = "Success",  // Dummy payment luôn thành công
        TransactionReference = "TRX-" + Guid.NewGuid().ToString("N").Substring(0, 12),
        UserId = dto.UserId,        // null nếu user chưa đăng nhập
        ProgramId = dto.ProgramId,  // Link đến program cụ thể nếu có
        DonorName = dto.Anonymous ? "Anonymous" : dto.FullName,
        DonorEmail = dto.Email,
        IsAnonymous = dto.Anonymous,
        SubscribeNewsletter = dto.Newsletter
    };
    
    // Lưu vào database
    _context.Donations.Add(donation);
    await _context.SaveChangesAsync();
    
    // Gửi email xác nhận (không chờ kết quả - fire and forget)
    _ = Task.Run(async () =>
    {
        try
        {
            var emailBody = EmailTemplate.DonationReceiptTemplate(...);
            await _emailService.SendEmailAsync(dto.Email, "Thank you", emailBody);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Email sending failed: {ex.Message}");
        }
    });
    
    return donation;
}
```

**Giải thích:**
- **Service**: Chứa logic nghiệp vụ (business logic) - cách xử lý donation, tạo mã giao dịch, gửi email
- **Entity Framework**: `_context.Donations.Add(donation)` - Thêm donation vào context (ngữ cảnh), chưa lưu vào database
- **SaveChangesAsync**: Lưu thay đổi vào database (thực thi câu lệnh SQL INSERT)
- **Fire and Forget**: `Task.Run()` - Chạy tác vụ gửi email ở background (nền), không chờ kết quả. Điều này giúp response nhanh hơn (không phải chờ email gửi xong)

**Bước 3: Database lưu trữ**

Entity Framework tự động tạo câu lệnh SQL:

```sql
INSERT INTO Donations 
  (Amount, CauseName, PaymentStatus, TransactionReference, UserId, ProgramId, DonorName, DonorEmail, ...)
VALUES 
  (100.00, 'Education', 'Success', 'TRX-a3f9b2c1d4e5', 10, 5, 'John Doe', 'john@example.com', ...)
```

### 3.3 Tất Cả Các API Endpoints (Điểm Cuối API)

**Bảng tóm tắt các Controller và Endpoints:**

| Controller | Route Base | Endpoints Chính | Yêu Cầu Auth | File |
|------------|-----------|-----------------|--------------|------|
| **AuthController** | `/api/auth` | `POST /register` - Đăng ký<br>`POST /login` - Đăng nhập<br>`POST /verify-email` - Xác thực email<br>`POST /forgot-password` - Quên mật khẩu<br>`POST /reset-password` - Đặt lại mật khẩu | Không | Dòng 1-205 |
| **DonationController** | `/api/donation` | `POST /` - Tạo donation<br>`GET /my-donations` - Lịch sử của user<br>`GET /` - Tất cả donations (admin)<br>`GET /{id}` - Chi tiết donation (admin) | Varies | Dòng 1-130 |
| **ProgramController** | `/api/program` | `GET /` - Tất cả programs<br>`GET /{id}/stats` - Thống kê program<br>`POST /{id}/register` - Đăng ký tham gia<br>`POST /` - Tạo program (admin) | Varies | - |
| **AdminController** | `/api/admin` | `GET /users` - Tất cả users<br>`PUT /users/{id}/role` - Đổi role<br>`DELETE /users/{id}` - Xóa user<br>`GET /donations` - Tất cả donations<br>`POST /queries/{id}/reply` - Trả lời query | Admin only | - |
| **ProfileController** | `/api/profile` | `GET /` - Thông tin profile<br>`PUT /` - Cập nhật profile<br>`POST /change-password` - Đổi mật khẩu | User | - |

**Giải thích:**
- **Endpoint**: Điểm cuối của API, là URL mà Frontend gọi để thực hiện một hành động cụ thể
- **Route Base**: Phần URL chung cho tất cả endpoints trong controller. Ví dụ: `/api/auth` là route base của AuthController
- **Auth Required**: Yêu cầu đăng nhập hay không
  - **Không**: Bất kỳ ai cũng có thể gọi
  - **User**: Phải đăng nhập
  - **Admin**: Phải đăng nhập và có role là Admin
  - **Varies**: Tùy endpoint cụ thể

---

**Tiếp theo: [4. Chi Tiết Frontend (Phần Giao Diện)](#4-chi-tiết-frontend-phần-giao-diện)**

---

## 4. Chi Tiết Frontend (Phần Giao Diện)

### 4.1 App.jsx - Component Chính và Routing

**File**: `FRONTEND/src/App.jsx` **Dòng 1-120**

**Vai trò**: Component chính của ứng dụng, định nghĩa routing (điều hướng) giữa các trang và bọc các Provider (nhà cung cấp context).

#### A. Cấu Trúc Routing (Dòng 52-101)

```jsx
<BrowserRouter>
  <AuthProvider>  {/* Cung cấp context xác thực cho toàn bộ app */}
    <ScrollToTop />  {/* Tự động cuộn lên đầu trang khi chuyển trang */}
    <Routes>
      {/* Admin Routes (Routes chỉ admin mới vào được) */}
      <Route path="/admin/*" element={
        <AdminRoute>  {/* Component bảo vệ route */}
          <AdminLayout>
            <Routes>
              <Route path="users" element={<UsersPage />} />
              <Route path="donations" element={<DonationsPageAdmin />} />
              {/* ... các route admin khác */}
            </Routes>
          </AdminLayout>
        </AdminRoute>
      } />
      
      {/* Public Routes (Routes công khai) */}
      <Route path="/*" element={
        <Layout>  {/* Layout chứa Navbar + Footer */}
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/donate" element={<DonatePage />} />
            <Route path="/login" element={<LoginPage />} />
            {/* ... các route công khai khác */}
          </Routes>
        </Layout>
      } />
    </Routes>
    
    {/* Container hiển thị thông báo (toast notifications) */}
    <ToastContainer position="top-right" autoClose={3000} />
  </AuthProvider>
</BrowserRouter>
```

**Giải thích:**
- **BrowserRouter**: Component của React Router, cung cấp routing (điều hướng) cho ứng dụng
- **Routes**: Container chứa các Route (tuyến đường)
- **Route**: Định nghĩa một tuyến đường, khi URL khớp với path, hiển thị component tương ứng
- **AuthProvider**: Provider cung cấp context xác thực (user, token, login, logout) cho toàn bộ app
- **AdminRoute**: Component bảo vệ route admin (chỉ admin mới vào được)
- **Layout**: Component bọc ngoài các trang công khai (chứa Navbar + Footer)
- **ToastContainer**: Container hiển thị thông báo (toast notifications) - thông báo nhỏ ở góc màn hình

#### B. AOS Initialization (Khởi Tạo Animation) (Dòng 36-49)

```jsx
useEffect(() => {
  import('aos').then(AOS => {
    AOS.init({
      duration: 700,          // Thời gian animation (milliseconds)
      easing: 'ease-in-out',  // Kiểu animation (ease-in-out = mượt mà)
      once: true,             // Chỉ animate 1 lần (không lặp lại khi scroll)
      offset: 50              // Offset (độ lệch) để trigger animation
    });
  });
  import('aos/dist/aos.css');  // Import CSS của AOS
}, []);
```

**Giải thích:**
- **AOS (Animate On Scroll)**: Thư viện tạo animation khi scroll (cuộn) trang
- **useEffect**: Hook của React, chạy code khi component mount (được render lần đầu)
- **Import dynamic**: `import('aos')` - Import thư viện động (lazy load - tải chậm) để tăng tốc độ load trang ban đầu

### 4.2 AuthContext - Quản Lý Trạng Thái Xác Thực

**File**: `FRONTEND/src/contexts/AuthContext.jsx`

**Vai trò**: Quản lý trạng thái đăng nhập toàn cục (global state) - user, token, isAuthenticated, isAdmin.

**State (Trạng thái):**
- `user` - Thông tin user đang đăng nhập (decoded từ JWT token): `{ id, email, role, fullName }`
- `token` - JWT token string (lưu trong localStorage)
- `isAuthenticated` - Boolean: `true` nếu đã đăng nhập, `false` nếu chưa
- `isAdmin` - Boolean: `true` nếu user có role là "Admin"
- `isLoading` - Boolean: `true` khi đang khởi tạo (load user từ localStorage)

**Các hàm chính:**

```jsx
const login = (newToken) => {
  // Lưu token vào localStorage (kho lưu trữ trình duyệt)
  localStorage.setItem('token', newToken);
  
  // Decode (giải mã) JWT token để lấy thông tin user
  const decoded = decodeToken(newToken);
  
  // Cập nhật state
  setUser(decoded);
  setToken(newToken);
};

const logout = () => {
  // Xóa token khỏi localStorage
  localStorage.removeItem('token');
  
  // Reset state
  setUser(null);
  setToken(null);
  
  // Chuyển hướng đến trang login
  navigate('/login');
};

// Tự động load user từ localStorage khi app khởi động
useEffect(() => {
  const storedToken = localStorage.getItem('token');
  if (storedToken) {
    try {
      const decoded = decodeToken(storedToken);
      // Kiểm tra token đã hết hạn chưa
      if (decoded.exp * 1000 > Date.now()) {
        setUser(decoded);
        setToken(storedToken);
      } else {
        // Token đã hết hạn, xóa nó
        localStorage.removeItem('token');
      }
    } catch (error) {
      // Token không hợp lệ, xóa nó
      localStorage.removeItem('token');
    }
  }
  setIsLoading(false);
}, []);
```

**Giải thích:**
- **Context**: Cơ chế quản lý state toàn cục trong React, không cần truyền props (thuộc tính) qua nhiều component
- **localStorage**: Kho lưu trữ trong trình duyệt, dữ liệu vẫn tồn tại sau khi đóng trình duyệt
- **Decode Token**: JWT token gồm 3 phần (header.payload.signature), decode phần payload để lấy thông tin user
- **Token Expiry**: JWT token có thời gian hết hạn (exp), kiểm tra `exp * 1000 > Date.now()` để đảm bảo token còn hợp lệ

### 4.3 API Service Layer - Cấu Hình Axios

**File**: `FRONTEND/src/services/api.js`

**Vai trò**: Cấu hình Axios (HTTP client), tự động thêm JWT Token vào mọi request, xử lý lỗi 401 (tự động đăng xuất).

```javascript
import axios from 'axios';

// Tạo instance Axios với base URL
const api = axios.create({
  baseURL: 'http://localhost:5230/api',  // URL cơ sở của API
  headers: { 'Content-Type': 'application/json' }  // Header mặc định
});

// Request Interceptor (Bộ chặn request): Tự động thêm JWT Token vào mọi request
api.interceptors.request.use(
  (config) => {
    // Lấy token từ localStorage
    const token = localStorage.getItem('token');
    if (token) {
      // Thêm token vào header Authorization
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response Interceptor (Bộ chặn response): Xử lý lỗi 401 (tự động đăng xuất)
api.interceptors.response.use(
  (response) => response,  // Nếu thành công, trả về response
  (error) => {
    // Nếu lỗi 401 (Unauthorized - Chưa được phép), tự động đăng xuất
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

**Giải thích:**
- **Axios**: Thư viện JavaScript để gọi API (HTTP client), dễ sử dụng hơn `fetch`
- **Interceptor**: Bộ chặn (middleware), tự động chạy trước/sau mỗi request/response
- **Request Interceptor**: Tự động thêm JWT Token vào header `Authorization: Bearer {token}` trước khi gửi request
- **Response Interceptor**: Tự động xử lý lỗi 401 (Unauthorized) - nếu nhận lỗi 401, xóa token và chuyển đến trang login
- **Base URL**: URL cơ sở, tất cả các request sẽ thêm vào sau base URL. Ví dụ: `api.get('/auth/login')` → `http://localhost:5230/api/auth/login`

### 4.4 Service Files - Các Hàm Gọi API

**Các service file chính:**

| Service File | Vai Trò | Các Hàm Chính |
|-------------|---------|---------------|
| **authServices.js** | Gọi API xác thực | `login(credentials)` - Đăng nhập<br>`register(userData)` - Đăng ký<br>`verifyEmail(token)` - Xác thực email<br>`forgotPassword(email)` - Quên mật khẩu<br>`resetPassword(data)` - Đặt lại mật khẩu |
| **donationServices.js** | Gọi API donation | `create(donationData)` - Tạo donation<br>`getMyDonations()` - Lấy lịch sử donation của user<br>`getAll()` - Lấy tất cả donations (admin) |
| **programServices.js** | Gọi API program | `getAll()` - Lấy tất cả programs<br>`getStats(programId)` - Lấy thống kê program<br>`registerInterest(data)` - Đăng ký tham gia program |
| **adminServices.js** | Gọi API admin | `getUsers()` - Lấy tất cả users<br>`updateUserRole(userId, role)` - Đổi role user<br>`getDonations()` - Lấy tất cả donations |

**Ví dụ sử dụng:**

```javascript
// authServices.js
import api from './api';

export const login = async (credentials) => {
  const response = await api.post('/auth/login', credentials);
  return response.data;  // Trả về dữ liệu từ response
};

// Sử dụng trong component
const handleLogin = async () => {
  try {
    const result = await login({ usernameOrEmail: 'user@example.com', password: 'password123' });
    // result chứa { message, token, user }
    authContext.login(result.token);
  } catch (error) {
    toast.error(error.response?.data?.message || 'Login failed');
  }
};
```

---

**Tiếp theo: [5. Cấu Trúc Cơ Sở Dữ Liệu](#5-cấu-trúc-cơ-sở-dữ-liệu)**

---

## 5. Cấu Trúc Cơ Sở Dữ Liệu

### 5.1 Sơ Đồ Quan Hệ Giữa Các Bảng (Entity Relationship Diagram)

```
┌─────────────┐       ┌──────────────┐       ┌──────────────┐
│   Users     │──────<│  Donations   │>──────│  NgoPrograms │
│             │ 1:N   │              │  N:1  │              │
│ Id (PK)     │       │ Id (PK)      │       │ Id (PK)      │
│ Email       │       │ Amount       │       │ Title        │
│ PasswordHash│       │ CauseName    │       │ Description  │
│ Role        │       │ UserId (FK)  │       │ NGOId (FK)   │
│ EmailVerified│       │ ProgramId(FK)│       │ GoalAmount   │
└─────────────┘       │ DonorName    │       └──────────────┘
       │              │ DonorEmail   │              │
       │              │ PaymentStatus│              │
       │              └──────────────┘              │
       │                                            │
       └────────────<│ProgramReg.   │>─────────────┘
                  1:N │              │ N:1
                      │ Id (PK)      │
                      │ UserId (FK)  │
                      │ ProgramId(FK)│
                      │ Email        │
                      └──────────────┘
```

**Giải thích:**
- **PK (Primary Key)**: Khóa chính, định danh duy nhất cho mỗi dòng trong bảng
- **FK (Foreign Key)**: Khóa ngoại, tham chiếu đến khóa chính của bảng khác
- **1:N (One-to-Many)**: Quan hệ một-nhiều. Ví dụ: 1 User có thể có nhiều Donations
- **N:1 (Many-to-One)**: Quan hệ nhiều-một. Ví dụ: Nhiều Donations có thể thuộc về 1 Program
- **Relationship**: Quan hệ giữa các bảng:
  - Users ↔ Donations: 1 User có nhiều Donations (1:N)
  - NgoPrograms ↔ Donations: 1 Program có nhiều Donations (1:N)
  - Users ↔ ProgramRegistrations: 1 User có nhiều ProgramRegistrations (1:N)
  - NgoPrograms ↔ ProgramRegistrations: 1 Program có nhiều ProgramRegistrations (1:N)

### 5.2 Các Bảng Chính

#### Bảng Users (Người Dùng)

**File**: `Backend/Models/User.cs` **Dòng 1-45**

**Các cột (columns) chính:**

| Cột | Kiểu Dữ Liệu | Ràng Buộc | Mô Tả |
|-----|-------------|-----------|-------|
| Id | int | PK, IDENTITY | Khóa chính, tự tăng |
| Username | string(50) | UNIQUE, NOT NULL | Tên đăng nhập (duy nhất) |
| Email | string(255) | UNIQUE, NOT NULL | Email (duy nhất) |
| PasswordHash | string(255) | NOT NULL | Mật khẩu đã mã hóa (BCrypt) |
| FullName | string(100) | NOT NULL | Họ và tên |
| Role | string(20) | NOT NULL, DEFAULT='User' | Vai trò: User hoặc Admin |
| EmailVerified | bool | NOT NULL, DEFAULT=0 | Đã xác thực email chưa |
| EmailVerificationToken | string(255) | NULL | Token xác thực email (đã mã hóa) |
| EmailVerificationTokenExpiry | DateTime | NULL | Thời gian hết hạn token (24 giờ) |
| PasswordResetToken | string(255) | NULL | Token reset password (đã mã hóa) |
| PasswordResetTokenExpiry | DateTime | NULL | Thời gian hết hạn token (1 giờ) |
| CreatedAt | DateTime | NOT NULL | Thời gian tạo tài khoản |

**Giải thích:**
- **IDENTITY**: Tự động tăng giá trị (1, 2, 3, ...)
- **UNIQUE**: Giá trị phải duy nhất (không trùng lặp)
- **NOT NULL**: Không được để trống (bắt buộc phải có giá trị)
- **DEFAULT**: Giá trị mặc định nếu không cung cấp
- **BCrypt**: Thuật toán mã hóa mật khẩu một chiều (one-way hashing), không thể giải mã ngược

#### Bảng Donations (Quyên Góp)

**File**: `Backend/Models/Donation.cs`

**Các cột chính:**

| Cột | Kiểu Dữ Liệu | Ràng Buộc | Mô Tả |
|-----|-------------|-----------|-------|
| Id | int | PK, IDENTITY | Khóa chính |
| Amount | decimal(18,2) | NOT NULL | Số tiền quyên góp |
| CauseName | string(200) | NOT NULL | Tên chương trình/cause |
| PaymentStatus | string(20) | NOT NULL, DEFAULT='Pending' | Trạng thái: Success/Failed/Pending |
| UserId | int | FK→Users, NULL | ID người dùng (null nếu chưa đăng nhập) |
| ProgramId | int | FK→NgoPrograms, NULL | ID chương trình (null nếu quyên góp chung) |
| TransactionReference | string(50) | UNIQUE | Mã giao dịch (TRX-xxxxxxxxxxxxx) |
| DonorName | string(100) | NOT NULL | Tên người quyên góp (hoặc "Anonymous") |
| DonorEmail | string(255) | NOT NULL | Email người quyên góp |
| DonorPhone | string(20) | NULL | Số điện thoại |
| DonorAddress | string(250) | NULL | Địa chỉ |
| IsAnonymous | bool | NOT NULL, DEFAULT=0 | Quyên góp ẩn danh |
| SubscribeNewsletter | bool | NOT NULL, DEFAULT=0 | Đăng ký nhận newsletter |
| CreatedAt | DateTime | NOT NULL | Thời gian tạo donation |

**Giải thích:**
- **FK (Foreign Key)**: Khóa ngoại, tham chiếu đến bảng khác
  - `UserId` tham chiếu đến `Users.Id`
  - `ProgramId` tham chiếu đến `NgoPrograms.Id`
- **NULL**: Cho phép giá trị null (có thể để trống)
- **decimal(18,2)**: Số thập phân với 18 chữ số tổng cộng, 2 chữ số sau dấu phẩy (ví dụ: 1234567890123456.12)

#### Bảng NgoPrograms (Chương Trình)

**File**: `Backend/Models/NgoProgram.cs`

**Các cột chính:**

| Cột | Kiểu Dữ Liệu | Ràng Buộc | Mô Tả |
|-----|-------------|-----------|-------|
| Id | int | PK, IDENTITY | Khóa chính |
| Title | string(200) | NOT NULL | Tên chương trình |
| Description | string | NULL | Mô tả chương trình |
| Location | string(200) | NULL | Địa điểm |
| StartDate | DateTime | NULL | Ngày bắt đầu |
| EndDate | DateTime | NULL | Ngày kết thúc |
| Status | string(20) | NOT NULL, DEFAULT='Active' | Trạng thái: Active/Completed/Cancelled |
| NGOId | int | FK→NGOs, NULL | ID tổ chức NGO |
| GoalAmount | decimal(18,2) | NULL | Mục tiêu quyên góp |
| CreatedAt | DateTime | NOT NULL | Thời gian tạo |

### 5.3 Migrations (Bản Ghi Thay Đổi Database)

**Vị trí**: `Backend/Migrations/`

**Migrations chính:**

1. **`20251021161050_InitialCreate.cs`**
   - Migration đầu tiên, tạo tất cả các bảng cơ bản: Users, Donations, NGOs, NgoPrograms, Partners, Gallery, Queries, Invitations

2. **`20251103131053_AddProgramRegistration.cs`**
   - Thêm bảng ProgramRegistrations (đăng ký tham gia chương trình)

3. **`20251105155757_AddProgramGoalAndDonationLink.cs`**
   - Thêm cột `GoalAmount` vào bảng NgoPrograms
   - Thêm cột `ProgramId` (FK) vào bảng Donations

**Cách sử dụng Migrations:**

```bash
# Tạo migration mới
dotnet ef migrations add TenMigration

# Áp dụng migrations vào database
dotnet ef database update

# Xóa database và tạo lại (cẩn thận - mất dữ liệu!)
dotnet ef database drop
dotnet ef database update

# Hoàn tác (rollback) migration cuối cùng
dotnet ef database update TenMigrationTruocDo
```

**Giải thích:**
- **Migration**: File ghi lại các thay đổi của database (tạo bảng, thêm cột, xóa cột, etc.)
- **Tại sao cần Migration**: 
  - Quản lý phiên bản database (version control)
  - Đồng bộ database giữa các developer
  - Có thể rollback (hoàn tác) nếu cần
- **Database Update**: Áp dụng tất cả migrations chưa được áp dụng vào database

---

**Tiếp theo: [6. Luồng Xác Thực Người Dùng](#6-luồng-xác-thực-người-dùng)**

---

## 6. Luồng Xác Thực Người Dùng

### 6.1 Luồng Đăng Ký → Xác Thực Email → Đăng Nhập

**Tổng quan luồng:**

```
1. User điền form đăng ký (RegisterPage.jsx)
   ↓
2. POST /api/auth/register
   ↓
3. AuthController.Register() (Dòng 24)
   ↓
4. AuthService.RegisterAsync() (Dòng 29)
   ├─ Kiểm tra email/username đã tồn tại chưa (Dòng 32-43)
   ├─ Mã hóa mật khẩu bằng BCrypt (Dòng 51)
   ├─ Tạo token xác thực email (Dòng 59)
   ├─ Lưu user vào database (Dòng 64-65)
   └─ Gửi email xác thực (fire-and-forget) (Dòng 74-111)
   ↓
5. User nhận email → Click link xác thực
   ↓
6. Frontend: /verify-email?token=xxx
   ↓
7. POST /api/auth/verify-email (token)
   ↓
8. AuthService.VerifyEmailAsync() (Dòng 139)
   ├─ Tìm user bằng token đã mã hóa (Dòng 164-193)
   ├─ Kiểm tra token đã hết hạn chưa (Dòng 202-204)
   ├─ Đặt EmailVerified = true (Dòng 211)
   └─ Xóa token xác thực (Dòng 212-213)
   ↓
9. User có thể đăng nhập
   ↓
10. POST /api/auth/login (username/email + password)
   ↓
11. AuthService.LoginAsync() (Dòng 292)
   ├─ Tìm user theo username HOẶC email (Dòng 295-300)
   ├─ Kiểm tra mật khẩu bằng BCrypt (Dòng 306-308)
   ├─ Kiểm tra email đã xác thực chưa (Dòng 311-312)
   └─ Tạo JWT Token (Dòng 315)
       └─> JwtHelper.GenerateToken(user, config)
           ├─ Tạo claims: userId, email, role
           ├─ Ký bằng secret key (HS256)
           └─ Đặt thời gian hết hạn (7 ngày)
   ↓
12. Frontend nhận token
   ↓
13. AuthContext.login(token)
   ├─ Lưu vào localStorage
   ├─ Decode JWT → setUser(decoded)
   └─ Điều hướng đến trang chủ
```

### 6.2 Chi Tiết Từng Bước

#### Bước 1-4: Đăng Ký (Registration)

**Frontend**: `FRONTEND/src/pages/RegisterPage.jsx`

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  // Chuẩn bị dữ liệu
  const userData = {
    email: formData.email,
    username: formData.username,
    password: formData.password,
    fullName: formData.fullName,
    phone: formData.phone,
    address: formData.address
  };
  
  try {
    // Gọi API đăng ký
    const result = await register(userData);
    
    // Hiển thị thông báo thành công
    toast.success('Đăng ký thành công! Vui lòng kiểm tra email để xác thực.');
    
    // Chuyển đến trang verify email
    navigate('/verify-email');
  } catch (error) {
    toast.error(error.response?.data?.message || 'Đăng ký thất bại');
  }
};
```

**Backend**: `Backend/Controllers/AuthController.cs` **Dòng 24-50**

```csharp
[HttpPost("register")]
public async Task<IActionResult> Register([FromBody] RegisterRequest? request)
{
    // Kiểm tra request có null không
    if (request == null)
    {
        return BadRequest(new { message = "Request body is required" });
    }

    // Kiểm tra validation (ModelState)
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value?.Errors.Count > 0)
            .SelectMany(x => x.Value.Errors.Select(e => new { Field = x.Key, Message = e.ErrorMessage }))
            .ToList();
        
        return BadRequest(new { message = "Validation failed", errors });
    }

    // Gọi Service để xử lý
    var result = await _authService.RegisterAsync(request);

    if (!result.success)
        return BadRequest(new { message = result.message });

    return Ok(new { message = result.message });
}
```

**Service**: `Backend/Services/AuthService.cs` **Dòng 29-116**

```csharp
public async Task<(bool success, string message, string? token)> RegisterAsync(RegisterRequest req)
{
    // Kiểm tra email/username đã tồn tại chưa (Dòng 32-43)
    var existingUser = await _context.Users
        .Where(u => u.Email == req.Email || u.Username == req.Username)
        .FirstOrDefaultAsync();

    if (existingUser != null)
    {
        if (existingUser.Email == req.Email)
            return (false, "Email already exists", null);
        if (existingUser.Username == req.Username)
            return (false, "Username already exists", null);
    }

    // Tạo user mới
    var user = new User
    {
        FullName = req.FullName,
        Username = req.Username,
        Email = req.Email,
        PasswordHash = PasswordHasher.Hash(req.Password),  // Mã hóa mật khẩu bằng BCrypt
        Role = "User",
        EmailVerified = false
    };

    // Tạo token xác thực email (Dòng 59)
    var verificationToken = GenerateEmailVerificationToken();
    user.EmailVerificationToken = PasswordHasher.Hash(verificationToken);  // Mã hóa token trước khi lưu
    user.EmailVerificationTokenExpiry = DateTime.UtcNow.AddHours(24);  // Token hết hạn sau 24 giờ

    // Lưu user vào database (Dòng 64-65)
    _context.Users.Add(user);
    await _context.SaveChangesAsync();

    // Gửi email xác thực (fire-and-forget) (Dòng 74-111)
    _ = Task.Run(async () =>
    {
        var verificationLink = $"{frontendUrl}/verify-email?token={verificationToken}";
        var emailBody = EmailTemplate.VerificationEmailTemplate(user.FullName, verificationLink, verificationToken);
        await _emailService.SendEmailAsync(user.Email, "Verify Your Give-AID Account", emailBody);
    });

    return (true, "Registration successful! Please check your email to verify your account.", null);
}
```

**Giải thích:**
- **BCrypt**: Thuật toán mã hóa mật khẩu một chiều, không thể giải mã ngược. Khi đăng nhập, so sánh mật khẩu nhập vào với hash đã lưu
- **Token xác thực email**: Token ngẫu nhiên 32 ký tự, được mã hóa trước khi lưu vào database (bảo mật)
- **Fire-and-forget**: Gửi email ở background, không chờ kết quả → response nhanh hơn (100-200ms thay vì 1-2 giây)

#### Bước 5-8: Xác Thực Email (Email Verification)

**Frontend**: `FRONTEND/src/pages/VerifyEmailPage.jsx`

```jsx
// Lấy token từ URL
const searchParams = new URLSearchParams(location.search);
const token = searchParams.get('token');

// Gọi API xác thực
const handleVerify = async () => {
  try {
    const result = await verifyEmail(token);
    toast.success('Xác thực email thành công!');
    navigate('/login');
  } catch (error) {
    toast.error('Token không hợp lệ hoặc đã hết hạn');
  }
};
```

**Backend**: `Backend/Services/AuthService.cs` **Dòng 139-215**

```csharp
public async Task<(bool success, string message)> VerifyEmailAsync(string token)
{
    // Tìm user có token xác thực (Dòng 164-193)
    // Lưu ý: Token trong database đã được mã hóa, nên cần so sánh bằng BCrypt
    var users = await _context.Users
        .Where(u => u.EmailVerificationToken != null && u.EmailVerified == false)
        .ToListAsync();

    User? foundUser = null;
    foreach (var user in users)
    {
        if (PasswordHasher.Verify(token, user.EmailVerificationToken))
        {
            foundUser = user;
            break;
        }
    }

    if (foundUser == null)
    {
        return (false, "Invalid or expired verification token");
    }

    // Kiểm tra token đã hết hạn chưa (Dòng 202-204)
    if (foundUser.EmailVerificationTokenExpiry < DateTime.UtcNow)
    {
        return (false, "Verification token has expired. Please request a new one.");
    }

    // Đặt EmailVerified = true (Dòng 211)
    foundUser.EmailVerified = true;
    foundUser.EmailVerificationToken = null;  // Xóa token
    foundUser.EmailVerificationTokenExpiry = null;

    await _context.SaveChangesAsync();

    return (true, "Email verified successfully. You can now login.");
}
```

**Giải thích:**
- **So sánh token**: Vì token trong database đã được mã hóa (hashed), không thể so sánh trực tiếp. Phải dùng `PasswordHasher.Verify()` để so sánh
- **Token expiry**: Token chỉ có hiệu lực trong 24 giờ. Sau đó phải yêu cầu gửi lại email xác thực

#### Bước 9-13: Đăng Nhập (Login)

**Frontend**: `FRONTEND/src/pages/LoginPage.jsx`

```jsx
const handleLogin = async (e) => {
  e.preventDefault();
  
  try {
    // Gọi API đăng nhập
    const result = await login({
      usernameOrEmail: formData.usernameOrEmail,
      password: formData.password
    });
    
    // Lưu token và đăng nhập
    authContext.login(result.token);
    
    // Chuyển đến trang chủ
    navigate('/');
  } catch (error) {
    toast.error(error.response?.data?.message || 'Đăng nhập thất bại');
  }
};
```

**Backend**: `Backend/Services/AuthService.cs` **Dòng 292-327**

```csharp
public async Task<(bool success, string message, string? token, User? user)> LoginAsync(LoginRequest req)
{
    // Tìm user theo username HOẶC email (Dòng 295-300)
    var user = await _context.Users
        .FirstOrDefaultAsync(u => 
            u.Username == req.UsernameOrEmail || 
            u.Email == req.UsernameOrEmail);

    if (user == null)
    {
        return (false, "Invalid username or password", null, null);
    }

    // Kiểm tra mật khẩu bằng BCrypt (Dòng 306-308)
    if (!PasswordHasher.Verify(req.Password, user.PasswordHash))
    {
        return (false, "Invalid username or password", null, null);
    }

    // Kiểm tra email đã xác thực chưa (Dòng 311-312)
    if (!user.EmailVerified)
    {
        return (false, "Please verify your email before logging in", null, null);
    }

    // Tạo JWT Token (Dòng 315)
    var token = JwtHelper.GenerateToken(user, _config);
    
    return (true, "Login successful", token, user);
}
```

**JWT Token Generation**: `Backend/Helpers/JwtHelper.cs` **Dòng 13-47**

```csharp
public static string GenerateToken(User user, IConfiguration config)
{
    // Lấy secret key từ configuration
    var jwtKey = config["Jwt:Key"] ?? "very_secret_key_change_me_please";
    var jwtIssuer = config["Jwt:Issuer"] ?? "Give_AID";
    var jwtExpireMinutes = 10080;  // 7 ngày (mặc định)

    var key = Encoding.ASCII.GetBytes(jwtKey);

    // Tạo claims (thông tin trong token)
    var claims = new[]
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),  // User ID
        new Claim(ClaimTypes.Email, user.Email ?? string.Empty),   // Email
        new Claim(ClaimTypes.Name, user.FullName ?? string.Empty), // Full Name
        new Claim(ClaimTypes.Role, user.Role ?? "User")            // Role
    };

    // Tạo token descriptor (mô tả token)
    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(claims),
        Expires = DateTime.UtcNow.AddMinutes(jwtExpireMinutes),  // Hết hạn sau 7 ngày
        Issuer = jwtIssuer,
        SigningCredentials = new SigningCredentials(
            new SymmetricSecurityKey(key),
            SecurityAlgorithms.HmacSha256Signature  // Dùng HS256 algorithm
        )
    };

    var tokenHandler = new JwtSecurityTokenHandler();
    var token = tokenHandler.CreateToken(tokenDescriptor);
    return tokenHandler.WriteToken(token);  // Trả về token dạng string
}
```

**Giải thích:**
- **Claims**: Thông tin chứa trong JWT token (user ID, email, role). Claims này được encode (mã hóa) vào token
- **HS256**: Thuật toán ký (signing) token, dùng secret key để tạo chữ ký (signature)
- **Token Expiry**: Token hết hạn sau 7 ngày. Sau đó user phải đăng nhập lại

### 6.3 Cấu Trúc JWT Token

**JWT Token gồm 3 phần, ngăn cách bởi dấu chấm (.)**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1laWQiOiIxMjMiLCJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJyb2xlIjoiVXNlciIsImlzcyI6IkdpdmVfQUlEIiwiZXhwIjoxNjk5OTk5OTk5fQ.signature
```

**1. Header (Phần đầu):**
```json
{
  "alg": "HS256",  // Algorithm (Thuật toán ký)
  "typ": "JWT"     // Type (Loại token)
}
```

**2. Payload (Phần thân - chứa claims):**
```json
{
  "nameid": "123",              // User ID (ClaimTypes.NameIdentifier)
  "email": "user@example.com",  // Email (ClaimTypes.Email)
  "name": "John Doe",           // Full Name (ClaimTypes.Name)
  "role": "User",               // Role (ClaimTypes.Role)
  "iss": "Give_AID",            // Issuer (Người phát hành)
  "exp": 1699999999             // Expiry (Thời gian hết hạn - Unix timestamp)
}
```

**3. Signature (Chữ ký):**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**Giải thích:**
- **Base64UrlEncode**: Mã hóa header và payload sang dạng Base64 URL-safe (an toàn cho URL)
- **Signature**: Chữ ký được tạo bằng cách mã hóa (header + payload + secret key) bằng HS256. Chữ ký này đảm bảo token không bị giả mạo
- **Decode token**: Có thể decode token tại [jwt.io](https://jwt.io) để xem payload (không cần secret key). Nhưng không thể tạo token mới mà không có secret key

### 6.4 Protected Routes (Routes Bảo Vệ)

**Backend - Sử dụng Attribute (Thuộc tính):**

```csharp
// Yêu cầu đăng nhập (bất kỳ user nào)
[Authorize]
[HttpGet("my-donations")]
public async Task<IActionResult> GetMyDonations()
{
    // Lấy user ID từ JWT token
    var userIdClaim = User.FindFirst(ClaimTypes.NameIdentifier);
    int userId = int.Parse(userIdClaim.Value);
    
    var donations = await _donationService.GetByUserIdAsync(userId);
    return Ok(donations);
}

// Yêu cầu role Admin
[Authorize(Roles = "Admin")]
[HttpGet("users")]
public async Task<IActionResult> GetAllUsers()
{
    var users = await _userService.GetAllAsync();
    return Ok(users);
}
```

**Giải thích:**
- **Authorize**: Attribute (thuộc tính) bảo vệ endpoint, yêu cầu phải đăng nhập
- **Roles = "Admin"**: Chỉ user có role "Admin" mới được phép truy cập
- **User.FindFirst(ClaimTypes.NameIdentifier)**: Lấy user ID từ JWT token (đã được decode bởi middleware)

**Frontend - Sử dụng Component Guard:**

**File**: `FRONTEND/src/components/admin/AdminRoute.jsx`

```jsx
export default function AdminRoute({ children }) {
  const { isAuthenticated, isAdmin, isLoading } = useAuth();
  
  // Đang load (khởi tạo)
  if (isLoading) {
    return <div>Đang tải...</div>;
  }
  
  // Chưa đăng nhập
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  // Không phải admin
  if (!isAdmin) {
    return (
      <div>
        <h1>403 - Forbidden</h1>
        <p>Bạn không có quyền truy cập trang này.</p>
      </div>
    );
  }
  
  // Đã đăng nhập và là admin → Cho phép truy cập
  return children;
}
```

**Giải thích:**
- **AdminRoute**: Component bảo vệ route admin, kiểm tra `isAuthenticated` và `isAdmin`
- **Navigate**: Component của React Router, tự động chuyển hướng đến URL khác
- **useAuth**: Hook lấy thông tin từ AuthContext (user, isAuthenticated, isAdmin)

---

**Tiếp theo: [7. Các Tính Năng Chính (Ví Dụ)](#7-các-tính-năng-chính-ví-dụ)**

---

## 7. Các Tính Năng Chính (Ví Dụ)

### 7.1 Ví Dụ 1: Tạo Donation (Quyên Góp) - Luồng Hoàn Chỉnh

**Mô tả**: Người dùng điền form quyên góp, chọn chương trình, nhập thông tin thanh toán → Hệ thống lưu donation, gửi email xác nhận.

**Luồng xử lý:**

```
1. User điền form trên DonatePage.jsx
   ↓
2. User nhấn nút "Quyên Góp"
   ↓
3. Frontend: handleSubmit() trong DonatePage.jsx (Dòng 150+)
   ├─ Validate dữ liệu client-side
   ├─ Tìm program title từ programId
   └─ Chuẩn bị payload
   ↓
4. Frontend: donationServices.create(donationData)
   ↓
5. HTTP: POST /api/donation (kèm JWT Token trong header)
   ↓
6. Backend: DonationController.Create() (Dòng 22-83)
   ├─ Kiểm tra DTO null
   ├─ Kiểm tra ModelState validation
   ├─ Kiểm tra Amount > 0
   └─ Gọi Service
   ↓
7. Backend: DonationService.CreateAsync() (Dòng 24-135)
   ├─ Chuẩn bị donation object
   ├─ Tạo mã giao dịch: "TRX-" + GUID
   ├─ Lưu vào database (_context.Donations.Add + SaveChangesAsync)
   └─ Gửi email xác nhận (fire-and-forget)
   ↓
8. Database: INSERT INTO Donations
   ↓
9. Email Service: Gửi email xác nhận (background)
   ↓
10. Backend: Trả về donation object (JSON)
   ↓
11. Frontend: Nhận response, hiển thị thông báo thành công
   ↓
12. Frontend: Navigate đến /donation-history
```

**Code chi tiết:**

**Frontend - DonatePage.jsx (Dòng 150-250):**

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  // Validation client-side
  if (!formData.amount || formData.amount <= 0) {
    toast.error('Số tiền phải lớn hơn 0');
    return;
  }
  
  // Tìm program title từ programId
  const selectedProgram = programs.find(p => p.id === parseInt(formData.programId));
  const causeName = selectedProgram ? selectedProgram.title : "General Donation";
  
  // Chuẩn bị payload
  const donationData = {
    amount: parseFloat(formData.amount),
    cause: causeName,
    programId: parseInt(formData.programId) || null,
    fullName: formData.fullName,
    email: formData.email,
    phone: formData.phone || null,
    address: formData.address || null,
    paymentMethod: formData.paymentMethod,
    anonymous: formData.anonymous,
    newsletter: formData.newsletter,
    userId: user?.id || null  // null nếu chưa đăng nhập
  };
  
  try {
    // Gọi API
    const result = await donationService.create(donationData);
    
    // Thành công
    toast.success('Quyên góp thành công! Kiểm tra email để xem biên lai.');
    navigate('/donation-history');
  } catch (error) {
    toast.error(error.response?.data?.message || 'Quyên góp thất bại');
  }
};
```

**Backend - DonationController.cs (Dòng 22-83):**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] DonationDTO? dto)
{
    // Kiểm tra null
    if (dto == null)
    {
        return BadRequest(new { message = "Request body is required" });
    }

    // Kiểm tra validation
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value?.Errors.Count > 0)
            .Select(x => new { Field = x.Key, Errors = x.Value?.Errors.Select(e => e.ErrorMessage) })
            .ToList();
        
        return BadRequest(new { message = "Validation failed", errors });
    }
    
    // Kiểm tra Amount
    if (dto.Amount <= 0)
    {
        return BadRequest(new { message = "Amount must be greater than 0" });
    }

    try
    {
        // Gọi Service
        var donation = await _donationService.CreateAsync(dto);
        return Ok(donation);
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = "Failed to create donation", error = ex.Message });
    }
}
```

**Backend - DonationService.cs (Dòng 24-135):**

```csharp
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    // Chuẩn bị donation object
    var donation = new Donation
    {
        Amount = dto.Amount,
        CauseName = dto.Cause ?? "General Donation",
        PaymentStatus = "Success",  // Dummy payment luôn thành công
        TransactionReference = "TRX-" + Guid.NewGuid().ToString("N").Substring(0, 12),
        UserId = dto.UserId,        // null nếu user chưa đăng nhập
        ProgramId = dto.ProgramId,  // Link đến program cụ thể
        DonorName = dto.Anonymous ? "Anonymous" : dto.FullName,
        DonorEmail = dto.Email,
        IsAnonymous = dto.Anonymous,
        SubscribeNewsletter = dto.Newsletter
    };
    
    // Lưu vào database
    _context.Donations.Add(donation);
    await _context.SaveChangesAsync();
    
    // Gửi email xác nhận (fire-and-forget)
    _ = Task.Run(async () =>
    {
        try
        {
            var emailBody = EmailTemplate.DonationReceiptTemplate(
                donation.DonorName,
                donation.Amount,
                donation.CauseName,
                donation.TransactionReference,
                donation.CreatedAt
            );
            
            await _emailService.SendEmailAsync(
                dto.Email,
                $"Thank you for your donation - #{donation.TransactionReference}",
                emailBody
            );
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Email sending failed: {ex.Message}");
        }
    });
    
    return donation;
}
```

**Database - SQL Query (tự động tạo bởi Entity Framework):**

```sql
INSERT INTO Donations 
  (Amount, CauseName, PaymentStatus, TransactionReference, UserId, ProgramId, DonorName, DonorEmail, IsAnonymous, SubscribeNewsletter, CreatedAt)
VALUES 
  (100.00, 'Education for Children', 'Success', 'TRX-a3f9b2c1d4e5', 10, 5, 'John Doe', 'john@example.com', 0, 1, GETUTCDATE())
```

**Email Template - EmailTemplate.cs (Dòng 64-135):**

Email xác nhận được gửi dưới dạng HTML đẹp mắt, chứa:
- Lời cảm ơn
- Số tiền quyên góp
- Tên chương trình/cause
- Mã giao dịch (Transaction Reference)
- Thời gian quyên góp

### 7.2 Ví Dụ 2: Đăng Ký Tham Gia Chương Trình (Prevent Duplicates)

**Mô tả**: User đăng ký tham gia một chương trình, hệ thống kiểm tra xem email đã đăng ký chưa để tránh trùng lặp.

**Luồng xử lý:**

```
1. User xem danh sách programs trên ProgramsPage.jsx
   ↓
2. User nhấn nút "Đăng Ký Tham Gia" trên một program
   ↓
3. Modal mở, user điền form (fullName, email, phone, message)
   ↓
4. Frontend: programServices.registerInterest(data)
   ↓
5. HTTP: POST /api/program/{programId}/register
   ↓
6. Backend: ProgramController.Register() 
   ↓
7. Backend: ProgramService.RegisterAsync() 
   ├─ Kiểm tra duplicate: Email đã đăng ký chương trình này chưa?
   ├─ Nếu đã đăng ký → Trả về lỗi
   ├─ Nếu chưa → Tạo ProgramRegistration
   └─ Lưu vào database
   ↓
8. Backend: Trả về success
   ↓
9. Frontend: Hiển thị thông báo thành công
```

**Code chi tiết:**

**Backend - ProgramService.cs:**

```csharp
public async Task<(bool success, string message)> RegisterAsync(int programId, ProgramRegistrationRequest request)
{
    // Kiểm tra program có tồn tại không
    var program = await _context.NgoPrograms.FindAsync(programId);
    if (program == null)
    {
        return (false, "Program not found");
    }

    // Kiểm tra duplicate: Email đã đăng ký chương trình này chưa? (case-insensitive)
    var existing = await _context.ProgramRegistrations
        .FirstOrDefaultAsync(pr => 
            pr.ProgramId == programId && 
            pr.Email.ToLower() == request.Email.ToLower());

    if (existing != null)
    {
        return (false, "Bạn đã đăng ký tham gia chương trình này rồi.");
    }

    // Tạo registration mới
    var registration = new ProgramRegistration
    {
        ProgramId = programId,
        Email = request.Email,
        FullName = request.FullName,
        Phone = request.Phone,
        Message = request.Message,
        CreatedAt = DateTime.UtcNow
    };

    _context.ProgramRegistrations.Add(registration);
    await _context.SaveChangesAsync();

    return (true, "Đăng ký thành công! Chúng tôi sẽ liên hệ với bạn sớm nhất.");
}
```

**Giải thích:**
- **Duplicate Prevention**: Kiểm tra email đã đăng ký chưa bằng cách so sánh `Email.ToLower()` (không phân biệt hoa thường) và `ProgramId`
- **Case-insensitive**: So sánh không phân biệt hoa thường để tránh trùng lặp (ví dụ: "John@example.com" và "john@example.com" được coi là giống nhau)

### 7.3 Ví Dụ 3: Admin Xem Tất Cả Donations

**Mô tả**: Admin đăng nhập, vào trang admin, xem danh sách tất cả donations từ tất cả users.

**Luồng xử lý:**

```
1. Admin đăng nhập → Lưu JWT token (có role="Admin")
   ↓
2. Admin truy cập /admin/donations
   ↓
3. Frontend: DonationsPageAdmin.jsx
   ├─ useEffect() gọi API khi component mount
   └─ Gọi adminServices.getDonations()
   ↓
4. HTTP: GET /api/admin/donations (kèm JWT Token với role="Admin")
   ↓
5. Backend: JWT Middleware kiểm tra token
   ├─ Validate signature
   ├─ Validate expiry
   └─ Check role="Admin"
   ↓
6. Backend: AdminController.GetAllDonations()
   ↓
7. Backend: DonationService.GetAllAsync()
   ├─ Query: SELECT * FROM Donations ORDER BY CreatedAt DESC
   └─ Include User (join với bảng Users)
   ↓
8. Database: Trả về danh sách donations
   ↓
9. Backend: Trả về JSON array
   ↓
10. Frontend: Cập nhật state, render table
```

**Code chi tiết:**

**Frontend - DonationsPageAdmin.jsx:**

```jsx
const [donations, setDonations] = useState([]);
const [loading, setLoading] = useState(true);

useEffect(() => {
  loadDonations();
}, []);

const loadDonations = async () => {
  setLoading(true);
  try {
    const data = await adminServices.getDonations();
    setDonations(data);
  } catch (error) {
    toast.error('Không thể tải danh sách donations');
  } finally {
    setLoading(false);
  }
};
```

**Backend - AdminController.cs:**

```csharp
[Authorize(Roles = "Admin")]  // Chỉ Admin mới vào được
[HttpGet("donations")]
public async Task<IActionResult> GetAllDonations()
{
    try
    {
        var donations = await _donationService.GetAllAsync();
        return Ok(donations);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"[AdminController] Error: {ex.Message}");
        return StatusCode(500, new { message = "Failed to retrieve donations" });
    }
}
```

**Backend - DonationService.cs (Dòng 137-157):**

```csharp
public async Task<List<Donation>> GetAllAsync()
{
    // Lấy tất cả donations, sắp xếp theo mới nhất trước
    return await _context.Donations
        .Include(d => d.User)  // Join với bảng Users (lazy loading)
        .OrderByDescending(d => d.CreatedAt)  // Sắp xếp mới nhất trước
        .ToListAsync();
}
```

**Giải thích:**
- **Include(d => d.User)**: Eager loading (tải sẵn) - Load thông tin User cùng lúc với Donation, tránh N+1 query problem (vấn đề truy vấn N+1)
- **OrderByDescending**: Sắp xếp giảm dần (mới nhất trước)
- **Authorize(Roles = "Admin")**: Chỉ user có role="Admin" mới được truy cập endpoint này

### 7.4 Ví Dụ 4: User Xem Lịch Sử Donation Của Mình

**Mô tả**: User đăng nhập, vào trang "Donation History", xem tất cả donations mà mình đã quyên góp.

**Luồng xử lý:**

```
1. User đăng nhập → Lưu JWT token (có userId)
   ↓
2. User truy cập /donation-history
   ↓
3. Frontend: DonationHistoryPage.jsx
   ├─ useEffect() gọi API khi component mount
   └─ Gọi donationServices.getMyDonations()
   ↓
4. HTTP: GET /api/donation/my-donations (kèm JWT Token)
   ↓
5. Backend: JWT Middleware kiểm tra token
   ├─ Validate token
   └─ Set HttpContext.User với claims (userId)
   ↓
6. Backend: DonationController.GetMyDonations()
   ├─ Lấy userId từ JWT token: User.FindFirst(ClaimTypes.NameIdentifier)
   └─ Gọi Service
   ↓
7. Backend: DonationService.GetByUserIdAsync(userId)
   ├─ Query: SELECT * FROM Donations WHERE UserId = {userId} ORDER BY CreatedAt DESC
   ↓
8. Database: Trả về danh sách donations của user
   ↓
9. Backend: Trả về JSON array
   ↓
10. Frontend: Tính toán thống kê (tổng số, tổng tiền, thành công, etc.)
   ↓
11. Frontend: Render table với donations
```

**Code chi tiết:**

**Frontend - DonationHistoryPage.jsx (Dòng 23-41):**

```jsx
useEffect(() => {
  if (isAuthenticated) {
    loadDonations();
  }
}, [isAuthenticated]);

const loadDonations = async () => {
  setLoading(true);
  try {
    const data = await donationService.getMyDonations();
    setDonations(Array.isArray(data) ? data : []);
  } catch (error) {
    console.error('Failed to load donations:', error);
    toast.error('Không thể tải lịch sử quyên góp');
  } finally {
    setLoading(false);
  }
};

// Tính toán thống kê
const totalDonations = donations.length;
const totalAmount = donations.reduce((sum, d) => sum + Number(d.amount || 0), 0);
const successDonations = donations.filter((d) => d.paymentStatus === 'Success').length;
const successAmount = donations
  .filter((d) => d.paymentStatus === 'Success')
  .reduce((sum, d) => sum + Number(d.amount || 0), 0);
```

**Backend - DonationController.cs (Dòng 89-110):**

```csharp
[Authorize]  // Yêu cầu đăng nhập
[HttpGet("my-donations")]
public async Task<IActionResult> GetMyDonations()
{
    try
    {
        // Lấy user ID từ JWT token
        var userIdClaim = User.FindFirst(ClaimTypes.NameIdentifier);
        if (userIdClaim == null || !int.TryParse(userIdClaim.Value, out int userId))
        {
            return Unauthorized(new { message = "Invalid token" });
        }

        // Lấy donations của user
        var donations = await _donationService.GetByUserIdAsync(userId);
        return Ok(donations);
    }
    catch (Exception ex)
    {
        return StatusCode(500, new { message = "Failed to retrieve donations" });
    }
}
```

**Backend - DonationService.cs (Dòng 167-187):**

```csharp
public async Task<List<Donation>> GetByUserIdAsync(int userId)
{
    // Lấy donations của user cụ thể, sắp xếp theo mới nhất trước
    return await _context.Donations
        .Where(d => d.UserId == userId)
        .OrderByDescending(d => d.CreatedAt)
        .ToListAsync();
}
```

**Giải thích:**
- **ClaimTypes.NameIdentifier**: Claim chứa user ID trong JWT token
- **User.FindFirst()**: Lấy claim đầu tiên khớp với tên claim
- **Where(d => d.UserId == userId)**: Lọc chỉ lấy donations của user hiện tại

### 7.5 Ví Dụ 5: Quên Mật Khẩu → Reset Password

**Mô tả**: User quên mật khẩu, nhập email → Nhận email reset password → Click link → Đặt lại mật khẩu mới.

**Luồng xử lý:**

```
1. User vào /forgot-password
   ↓
2. User nhập email → Nhấn "Gửi Email"
   ↓
3. Frontend: authServices.forgotPassword(email)
   ↓
4. HTTP: POST /api/auth/forgot-password
   ↓
5. Backend: AuthController.ForgotPassword()
   ↓
6. Backend: AuthService.ForgotPasswordAsync()
   ├─ Tìm user theo email
   ├─ Tạo token reset password (ngẫu nhiên)
   ├─ Mã hóa token bằng BCrypt
   ├─ Lưu token + expiry (1 giờ) vào database
   └─ Gửi email chứa reset link (fire-and-forget)
   ↓
7. User nhận email → Click link reset
   ↓
8. Frontend: /reset-password?token=xxx
   ↓
9. User nhập mật khẩu mới → Nhấn "Đặt Lại"
   ↓
10. Frontend: authServices.resetPassword({ token, newPassword })
   ↓
11. HTTP: POST /api/auth/reset-password
   ↓
12. Backend: AuthService.ResetPasswordAsync()
   ├─ Tìm user bằng token đã mã hóa
   ├─ Kiểm tra token đã hết hạn chưa (1 giờ)
   ├─ Mã hóa mật khẩu mới bằng BCrypt
   ├─ Cập nhật PasswordHash
   └─ Xóa reset token
   ↓
13. Backend: Trả về success
   ↓
14. Frontend: Hiển thị thông báo, chuyển đến /login
```

**Code chi tiết:**

**Backend - AuthService.cs - ForgotPasswordAsync() (Dòng 371+):**

```csharp
public async Task<(bool success, string message)> ForgotPasswordAsync(string email)
{
    // Tìm user theo email
    var user = await _context.Users.FirstOrDefaultAsync(u => u.Email == email);
    
    // Luôn trả về success (bảo mật - không tiết lộ email có tồn tại không)
    if (user == null)
    {
        return (true, "If the email exists in our system, a password reset link has been sent.");
    }

    // Tạo token reset password
    var resetToken = GeneratePasswordResetToken();
    user.PasswordResetToken = PasswordHasher.Hash(resetToken);  // Mã hóa token
    user.PasswordResetTokenExpiry = DateTime.UtcNow.AddHours(1);  // Hết hạn sau 1 giờ

    await _context.SaveChangesAsync();

    // Gửi email reset password (fire-and-forget)
    _ = Task.Run(async () =>
    {
        var resetLink = $"{frontendUrl}/reset-password?token={resetToken}";
        var emailBody = EmailTemplate.PasswordResetEmailTemplate(user.FullName, resetLink, resetToken);
        await _emailService.SendEmailAsync(user.Email, "Reset Your Password", emailBody);
    });

    return (true, "If the email exists in our system, a password reset link has been sent.");
}
```

**Backend - AuthService.cs - ResetPasswordAsync():**

```csharp
public async Task<(bool success, string message)> ResetPasswordAsync(string token, string newPassword)
{
    // Tìm user có reset token (so sánh bằng BCrypt)
    var users = await _context.Users
        .Where(u => u.PasswordResetToken != null)
        .ToListAsync();

    User? foundUser = null;
    foreach (var user in users)
    {
        if (PasswordHasher.Verify(token, user.PasswordResetToken))
        {
            foundUser = user;
            break;
        }
    }

    if (foundUser == null)
    {
        return (false, "Invalid or expired reset token");
    }

    // Kiểm tra token đã hết hạn chưa
    if (foundUser.PasswordResetTokenExpiry < DateTime.UtcNow)
    {
        return (false, "Reset token has expired. Please request a new one.");
    }

    // Cập nhật mật khẩu mới
    foundUser.PasswordHash = PasswordHasher.Hash(newPassword);
    foundUser.PasswordResetToken = null;  // Xóa token
    foundUser.PasswordResetTokenExpiry = null;

    await _context.SaveChangesAsync();

    return (true, "Password reset successfully. You can now login with your new password.");
}
```

**Giải thích:**
- **Security**: Luôn trả về success message ngay cả khi email không tồn tại → Không tiết lộ email có trong hệ thống hay không (bảo mật)
- **Token expiry**: Token reset password chỉ có hiệu lực 1 giờ (ngắn hơn token xác thực email - 24 giờ) vì liên quan đến bảo mật
- **Token hashing**: Token được mã hóa bằng BCrypt trước khi lưu vào database → Bảo mật cao hơn

---

**Tiếp theo: [8. Hướng Dẫn Gỡ Lỗi](#8-hướng-dẫn-gỡ-lỗi)**

---

## 8. Hướng Dẫn Gỡ Lỗi

### 8.1 Lỗi Thường Gặp và Cách Xử Lý

#### Lỗi 1: "Cannot connect to database" (Không thể kết nối database)

**Triệu chứng:**
- Backend không khởi động được
- Lỗi: `SqlException: Cannot open database "GiveAidDB"`

**Cách xử lý:**

1. **Kiểm tra Connection String:**
   - Mở file `Backend/appsettings.Development.json`
   - Kiểm tra connection string có đúng không:
     ```json
     {
       "ConnectionStrings": {
         "DefaultConnection": "Server=TEN_MAY_CUA_BAN;Database=Give_AID_API_Db;User Id=USER_CUA_BAN;Password=PASS_CUA_BAN;Encrypt=False;TrustServerCertificate=True;"
       }
     }
     ```

2. **Kiểm tra SQL Server có đang chạy không:**
   ```bash
   # Mở SQL Server Management Studio (SSMS)
   # Hoặc kiểm tra trong Services (Windows Services)
   ```

3. **Kiểm tra database đã tồn tại chưa:**
   ```sql
   -- Mở SSMS, chạy query:
   SELECT name FROM sys.databases WHERE name = 'Give_AID_API_Db';
   ```

4. **Tạo database nếu chưa có:**
   ```bash
   # Chạy migration để tạo database tự động
   cd Backend
   dotnet ef database update
   ```

#### Lỗi 2: "400 Bad Request" khi tạo donation

**Triệu chứng:**
- Frontend gọi API tạo donation → Nhận lỗi 400
- Console hiển thị: `Validation failed`

**Cách xử lý:**

1. **Kiểm tra Console Log (Backend):**
   - Mở terminal chạy Backend
   - Xem log: `[Donation] Received: Amount=..., Cause=..., FullName=...`
   - Xem log: `[Donation] ModelState validation failed: ...`

2. **Kiểm tra Frontend Console (Browser):**
   - Mở Developer Tools (F12) → Console tab
   - Xem error message chi tiết
   - Xem Network tab → Xem request payload (body)

3. **Kiểm tra Validation:**
   - `Amount` phải > 0
   - `Email` phải đúng định dạng email
   - `FullName` không được để trống
   - `Cause` không được để trống

4. **Kiểm tra DTO Binding:**
   - Kiểm tra tên field có đúng không (camelCase vs PascalCase)
   - Xem `DonationDTO.cs` để biết tên field đúng

**Ví dụ debug:**

```javascript
// Frontend - Thêm console.log trước khi gọi API
console.log('Donation data:', donationData);

try {
  const result = await donationService.create(donationData);
  console.log('Success:', result);
} catch (error) {
  console.error('Error response:', error.response?.data);
  console.error('Error details:', error);
}
```

```csharp
// Backend - Đã có sẵn trong DonationController.cs (Dòng 32)
Console.WriteLine($"[Donation] Received: Amount={dto.Amount}, Cause={dto.Cause}, FullName={dto.FullName}, Email={dto.Email}");
```

#### Lỗi 3: "401 Unauthorized" (Chưa được phép)

**Triệu chứng:**
- Frontend gọi API yêu cầu đăng nhập → Nhận lỗi 401
- Console hiển thị: `Unauthorized`

**Cách xử lý:**

1. **Kiểm tra JWT Token:**
   ```javascript
   // Frontend - Kiểm tra token có trong localStorage không
   const token = localStorage.getItem('token');
   console.log('Token:', token);
   ```

2. **Kiểm tra Token đã hết hạn chưa:**
   - Decode JWT token tại [jwt.io](https://jwt.io)
   - Kiểm tra `exp` (expiry) - phải > thời gian hiện tại (Unix timestamp)

3. **Kiểm tra Token có trong request header không:**
   - Mở Developer Tools → Network tab
   - Xem request → Headers → Authorization
   - Phải có: `Authorization: Bearer {token}`

4. **Kiểm tra Backend JWT Configuration:**
   - Kiểm tra `appsettings.json` có `Jwt:Key` không
   - Kiểm tra `Program.cs` có cấu hình JWT authentication không

**Ví dụ debug:**

```javascript
// Frontend - Kiểm tra token trước khi gọi API
const token = localStorage.getItem('token');
if (!token) {
  console.error('No token found!');
  // Redirect to login
}

// Kiểm tra token expiry
const decoded = JSON.parse(atob(token.split('.')[1]));
const exp = decoded.exp * 1000; // Convert to milliseconds
if (exp < Date.now()) {
  console.error('Token expired!');
  // Clear token and redirect to login
  localStorage.removeItem('token');
}
```

```csharp
// Backend - Đã có sẵn trong Program.cs
// Kiểm tra JWT middleware có được cấu hình đúng không
```

#### Lỗi 4: "403 Forbidden" (Không có quyền)

**Triệu chứng:**
- User (không phải Admin) truy cập trang admin → Nhận lỗi 403

**Cách xử lý:**

1. **Kiểm tra Role trong JWT Token:**
   - Decode JWT token tại [jwt.io](https://jwt.io)
   - Kiểm tra claim `role` - phải là "Admin"

2. **Kiểm tra User có role Admin không:**
   ```sql
   -- Kiểm tra trong database
   SELECT Id, Username, Email, Role FROM Users WHERE Email = 'admin@giveaid.org';
   ```

3. **Kiểm tra Frontend AdminRoute:**
   - Xem `FRONTEND/src/components/admin/AdminRoute.jsx`
   - Kiểm tra logic kiểm tra `isAdmin`

**Ví dụ debug:**

```javascript
// Frontend - Kiểm tra role
const { user, isAdmin } = useAuth();
console.log('User:', user);
console.log('Is Admin:', isAdmin);
```

#### Lỗi 5: Email không được gửi

**Triệu chứng:**
- User đăng ký nhưng không nhận email xác thực
- User quyên góp nhưng không nhận email xác nhận

**Cách xử lý:**

1. **Kiểm tra SMTP Configuration:**
   - Mở file `Backend/appsettings.json`
   - Kiểm tra cấu hình SMTP:
     ```json
     {
       "Smtp": {
         "Host": "smtp.gmail.com",
         "Port": "587",
         "User": "your-email@gmail.com",
         "Pass": "your-app-password",
         "From": "your-email@gmail.com"
       }
     }
     ```

2. **Kiểm tra Gmail App Password:**
   - Nếu dùng Gmail, phải tạo App Password (không dùng mật khẩu thường)
   - Hướng dẫn: [Google App Passwords](https://support.google.com/accounts/answer/185833)

3. **Kiểm tra Console Log (Backend):**
   - Xem log: `[Warning] Email skipped: missing configuration`
   - Xem log: `[Error] Failed to send email to ...`

4. **Kiểm tra Firewall/Antivirus:**
   - Có thể bị chặn bởi firewall hoặc antivirus
   - Thử tắt tạm thời để test

**Ví dụ debug:**

```csharp
// Backend - Đã có sẵn trong EmailService.cs
// Kiểm tra log console để xem chi tiết lỗi
Console.WriteLine($"[Error] Failed to send email to {to}: {ex.Message}");
```

### 8.2 Cách Đặt Breakpoint và Debug

#### Backend (C# - Visual Studio / Rider)

1. **Đặt Breakpoint:**
   - Click vào lề trái của dòng code (gần số dòng)
   - Breakpoint xuất hiện (chấm đỏ)

2. **Chạy Debug Mode:**
   - Nhấn F5 (hoặc Run → Start Debugging)
   - Hoặc: `dotnet run` trong terminal

3. **Xem Variables:**
   - Hover chuột vào biến để xem giá trị
   - Hoặc mở Watch window để theo dõi biến cụ thể

4. **Step Through:**
   - F10: Step Over (bước qua)
   - F11: Step Into (bước vào)
   - F5: Continue (tiếp tục)

**Ví dụ:**
```csharp
// Đặt breakpoint ở đây (Dòng 73 trong DonationController.cs)
var donation = await _donationService.CreateAsync(dto);
// Khi chạy đến đây, debugger sẽ dừng lại
// Hover vào 'dto' để xem giá trị
// Hover vào 'donation' sau khi chạy xong để xem kết quả
```

#### Frontend (React - Chrome DevTools)

1. **Đặt Breakpoint trong Source Code:**
   - Mở Developer Tools (F12) → Sources tab
   - Tìm file cần debug (ví dụ: `DonatePage.jsx`)
   - Click vào số dòng để đặt breakpoint

2. **Sử dụng `debugger` Statement:**
   ```javascript
   const handleSubmit = async (e) => {
     e.preventDefault();
     debugger;  // Dừng ở đây khi chạy
     // Code tiếp theo...
   };
   ```

3. **Xem Variables:**
   - Mở Scope panel (bên trái)
   - Xem Local variables, Closures, Global variables

4. **Step Through:**
   - F10: Step Over
   - F11: Step Into
   - F8: Continue

**Ví dụ:**
```javascript
// Đặt breakpoint hoặc thêm debugger statement
const handleSubmit = async (e) => {
  e.preventDefault();
  debugger;  // Dừng ở đây
  console.log('Form data:', formData);
  // ...
};
```

### 8.3 Kiểm Tra Database

#### Xem Dữ Liệu Trong Database

1. **Mở SQL Server Management Studio (SSMS):**
   - Connect đến SQL Server
   - Mở database `Give_AID_API_Db`

2. **Chạy Query để xem dữ liệu:**
   ```sql
   -- Xem tất cả users
   SELECT * FROM Users;

   -- Xem tất cả donations
   SELECT * FROM Donations;

   -- Xem donations của user cụ thể
   SELECT * FROM Donations WHERE UserId = 1;

   -- Xem programs
   SELECT * FROM NgoPrograms;

   -- Xem program registrations
   SELECT * FROM ProgramRegistrations;
   ```

3. **Kiểm tra User có role Admin không:**
   ```sql
   SELECT Id, Username, Email, Role, EmailVerified 
   FROM Users 
   WHERE Role = 'Admin';
   ```

4. **Kiểm tra Email Verification Token:**
   ```sql
   SELECT Id, Email, EmailVerified, EmailVerificationToken, EmailVerificationTokenExpiry
   FROM Users
   WHERE Email = 'user@example.com';
   ```

#### Kiểm Tra Migration

1. **Xem tất cả migrations đã apply:**
   ```bash
   cd Backend
   dotnet ef migrations list
   ```

2. **Xem migration history trong database:**
   ```sql
   SELECT * FROM __EFMigrationsHistory;
   ```

3. **Xóa database và tạo lại (cẩn thận - mất dữ liệu!):**
   ```bash
   cd Backend
   dotnet ef database drop
   dotnet ef database update
   ```

### 8.4 Kiểm Tra Logs

#### Backend Logs

Backend sử dụng `Console.WriteLine()` để log. Xem logs trong terminal chạy Backend:

```csharp
// Ví dụ log trong DonationController.cs (Dòng 32)
Console.WriteLine($"[Donation] Received: Amount={dto.Amount}, Cause={dto.Cause}");

// Ví dụ log trong DonationService.cs (Dòng 71)
Console.WriteLine($"[DonationService] Creating donation: Amount={donation.Amount}");

// Ví dụ log error
Console.WriteLine($"[Donation] Exception: {ex.Message}");
Console.WriteLine($"[Donation] StackTrace: {ex.StackTrace}");
```

#### Frontend Logs

Frontend sử dụng `console.log()`, `console.error()`. Xem logs trong Browser Console (F12):

```javascript
// Ví dụ log trong DonatePage.jsx
console.log('Form data:', formData);
console.error('Error:', error);

// Ví dụ log API call
console.log('API response:', response);
console.error('API error:', error.response?.data);
```

---

**Tiếp theo: [9. Best Practices (Thực Hành Tốt)](#9-best-practices-thực-hành-tốt)**

---

## 9. Best Practices (Thực Hành Tốt)

### 9.1 Naming Conventions (Quy Tắc Đặt Tên)

#### Backend (C#)

- **Classes**: PascalCase
  ```csharp
  public class DonationService { }
  public class AuthController { }
  ```

- **Methods**: PascalCase
  ```csharp
  public async Task<Donation> CreateAsync(DonationDTO dto) { }
  public async Task<bool> VerifyEmailAsync(string token) { }
  ```

- **Variables/Parameters**: camelCase
  ```csharp
  var donation = new Donation();
  string donorName = "John Doe";
  ```

- **Constants**: PascalCase
  ```csharp
  public const int MaxRetryAttempts = 3;
  public const string DefaultRole = "User";
  ```

- **Private Fields**: camelCase với prefix `_`
  ```csharp
  private readonly GiveAidContext _context;
  private readonly EmailService _emailService;
  ```

#### Frontend (JavaScript/React)

- **Components**: PascalCase
  ```jsx
  function DonatePage() { }
  function DonationHistoryPage() { }
  ```

- **Functions/Variables**: camelCase
  ```javascript
  const handleSubmit = () => { };
  const formData = { };
  ```

- **Constants**: UPPER_SNAKE_CASE
  ```javascript
  const API_BASE_URL = 'http://localhost:5230/api';
  const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
  ```

- **CSS Classes**: kebab-case
  ```css
  .donation-card { }
  .category-title { }
  ```

### 9.2 Error Handling (Xử Lý Lỗi)

#### Backend - Luôn Bắt Exception

```csharp
// ✅ Tốt - Bắt exception và log chi tiết
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    try
    {
        // Logic xử lý
        _context.Donations.Add(donation);
        await _context.SaveChangesAsync();
        return donation;
    }
    catch (DbUpdateException dbEx)
    {
        Console.WriteLine($"[DonationService] Database error: {dbEx.Message}");
        throw new Exception("Failed to save donation", dbEx);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"[DonationService] Error: {ex.Message}");
        throw;
    }
}
```

#### Frontend - Luôn Xử Lý Error

```javascript
// ✅ Tốt - Bắt error và hiển thị thông báo
const handleSubmit = async () => {
  try {
    const result = await donationService.create(donationData);
    toast.success('Quyên góp thành công!');
  } catch (error) {
    // Xử lý lỗi chi tiết
    const errorMessage = error.response?.data?.message || 'Đã xảy ra lỗi';
    toast.error(errorMessage);
    console.error('Error details:', error);
  }
};
```

### 9.3 Validation (Xác Thực Dữ Liệu)

#### Backend - Validate Cả DTO và Service

```csharp
// ✅ Tốt - Validate trong DTO (attributes)
public class DonationDTO
{
    [Required]
    [Range(0.01, double.MaxValue, ErrorMessage = "Amount must be greater than 0")]
    public decimal Amount { get; set; }

    [Required]
    [EmailAddress]
    [MaxLength(255)]
    public string Email { get; set; }
}

// ✅ Tốt - Validate thêm trong Service
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    if (dto.Amount <= 0)
        throw new ArgumentException("Amount must be greater than 0");
    
    if (string.IsNullOrWhiteSpace(dto.Email))
        throw new ArgumentException("Email is required");
    
    // Logic xử lý
}
```

#### Frontend - Validate Trước Khi Gửi

```javascript
// ✅ Tốt - Validate client-side trước khi gọi API
const handleSubmit = async (e) => {
  e.preventDefault();
  
  // Validation
  if (!formData.amount || formData.amount <= 0) {
    toast.error('Số tiền phải lớn hơn 0');
    return;
  }
  
  if (!formData.email || !/\S+@\S+\.\S+/.test(formData.email)) {
    toast.error('Email không hợp lệ');
    return;
  }
  
  // Gọi API
  try {
    await donationService.create(donationData);
  } catch (error) {
    // ...
  }
};
```

### 9.4 Security (Bảo Mật)

#### Backend

1. **Mã hóa mật khẩu bằng BCrypt:**
   ```csharp
   // ✅ Tốt
   user.PasswordHash = PasswordHasher.Hash(password);
   
   // ❌ Không bao giờ làm thế này
   user.PasswordHash = password; // Plain text!
   ```

2. **Mã hóa token trước khi lưu vào database:**
   ```csharp
   // ✅ Tốt
   user.EmailVerificationToken = PasswordHasher.Hash(token);
   
   // ❌ Không bao giờ làm thế này
   user.EmailVerificationToken = token; // Plain text!
   ```

3. **Luôn kiểm tra Authorization:**
   ```csharp
   // ✅ Tốt
   [Authorize(Roles = "Admin")]
   [HttpGet("users")]
   public async Task<IActionResult> GetAllUsers() { }
   ```

#### Frontend

1. **Không lưu mật khẩu trong localStorage:**
   ```javascript
   // ✅ Tốt - Chỉ lưu JWT token
   localStorage.setItem('token', token);
   
   // ❌ Không bao giờ làm thế này
   localStorage.setItem('password', password); // Nguy hiểm!
   ```

2. **Xóa token khi logout:**
   ```javascript
   // ✅ Tốt
   const logout = () => {
     localStorage.removeItem('token');
     setUser(null);
   };
   ```

### 9.5 Performance (Hiệu Suất)

#### Backend

1. **Sử dụng Eager Loading để tránh N+1 Query:**
   ```csharp
   // ✅ Tốt - Load User cùng lúc với Donation
   return await _context.Donations
       .Include(d => d.User)
       .ToListAsync();
   
   // ❌ Không tốt - N+1 query problem
   var donations = await _context.Donations.ToListAsync();
   foreach (var donation in donations)
   {
       var user = await _context.Users.FindAsync(donation.UserId); // Query riêng!
   }
   ```

2. **Fire-and-Forget cho Email:**
   ```csharp
   // ✅ Tốt - Gửi email ở background, không chờ kết quả
   _ = Task.Run(async () =>
   {
       await _emailService.SendEmailAsync(email, subject, body);
   });
   ```

#### Frontend

1. **Sử dụng `useMemo` và `useCallback` khi cần:**
   ```javascript
   // ✅ Tốt - Memoize expensive calculations
   const totalAmount = useMemo(() => {
     return donations.reduce((sum, d) => sum + Number(d.amount || 0), 0);
   }, [donations]);
   ```

2. **Lazy Load Components:**
   ```javascript
   // ✅ Tốt - Lazy load components lớn
   const AdminPage = lazy(() => import('./pages/AdminPage'));
   ```

### 9.6 Code Organization (Tổ Chức Code)

#### Backend - Separation of Concerns (Tách Biệt Mối Quan Tâm)

```
Backend/
├── Controllers/     # Nhận HTTP request, trả về response
├── Services/        # Logic nghiệp vụ (business logic)
├── Models/          # Entity models (database tables)
├── DTOs/            # Data Transfer Objects (dữ liệu truyền qua API)
├── Helpers/         # Helper functions (JWT, Email, Password)
└── Data/            # Database context
```

**Nguyên tắc:**
- Controller: Chỉ nhận request, validate, gọi Service, trả về response
- Service: Chứa logic nghiệp vụ, tương tác với database
- Model: Chỉ định nghĩa cấu trúc dữ liệu, không chứa logic

#### Frontend - Component Structure

```
FRONTEND/src/
├── pages/           # Các trang chính (HomePage, DonatePage, etc.)
├── components/      # Các component tái sử dụng (Navbar, Footer, etc.)
├── contexts/        # Context providers (AuthContext, etc.)
├── services/        # API service layer (authServices, donationServices, etc.)
└── assets/          # Static assets (images, CSS, etc.)
```

**Nguyên tắc:**
- Pages: Component lớn, đại diện cho một trang
- Components: Component nhỏ, có thể tái sử dụng
- Services: Tách logic gọi API ra khỏi component

---

**Tiếp theo: [10. Onboarding Checklist (Danh Sách Cho Dev Mới)](#10-onboarding-checklist-danh-sách-cho-dev-mới)**

---

## 10. Onboarding Checklist (Danh Sách Cho Dev Mới)

### 10.1 Setup Môi Trường (Environment Setup)

- [ ] **Cài đặt .NET SDK (Backend)**
  - Download: [.NET 8.0 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
  - Kiểm tra: `dotnet --version` (phải >= 8.0)

- [ ] **Cài đặt Node.js (Frontend)**
  - Download: [Node.js LTS](https://nodejs.org/)
  - Kiểm tra: `node --version` (phải >= 18.0)
  - Kiểm tra: `npm --version`

- [ ] **Cài đặt SQL Server**
  - Download: [SQL Server Express](https://www.microsoft.com/sql-server/sql-server-downloads)
  - Hoặc dùng SQL Server LocalDB (đi kèm với Visual Studio)

- [ ] **Cài đặt IDE/Editor**
  - Visual Studio 2022 (khuyến nghị cho Backend)
  - Visual Studio Code (khuyến nghị cho Frontend)
  - Hoặc Rider (JetBrains)

- [ ] **Cài đặt Git**
  - Download: [Git](https://git-scm.com/downloads)
  - Kiểm tra: `git --version`

### 10.2 Clone và Setup Project

- [ ] **Clone Repository**
  ```bash
  git clone <repository-url>
  cd NGO_Website_template/GIVE-AID
  ```

- [ ] **Setup Backend**
  ```bash
  cd Backend
  # Tạo appsettings.Development.json từ appsettings.example.json
  cp appsettings.example.json appsettings.Development.json
  # Chỉnh sửa connection string trong appsettings.Development.json
  # Cài đặt dependencies
  dotnet restore
  # Chạy migrations để tạo database
  dotnet ef database update
  # Chạy backend
  dotnet run
  ```
  - Backend chạy ở: `http://localhost:5230`

- [ ] **Setup Frontend**
  ```bash
  cd FRONTEND
  # Cài đặt dependencies
  npm install
  # Chạy frontend
  npm run dev
  ```
  - Frontend chạy ở: `http://localhost:5173`

### 10.3 Kiểm Tra Cấu Hình

- [ ] **Kiểm tra Backend Configuration**
  - [ ] `appsettings.Development.json` có connection string đúng không
  - [ ] `appsettings.json` có JWT key không
  - [ ] `appsettings.json` có SMTP config không (nếu cần gửi email)

- [ ] **Kiểm tra Frontend Configuration**
  - [ ] `src/services/api.js` có baseURL đúng không (`http://localhost:5230/api`)

- [ ] **Kiểm tra Database**
  - [ ] Database đã được tạo chưa
  - [ ] Migrations đã được apply chưa
  - [ ] Admin user đã được tạo chưa (email: admin@giveaid.org, password: Admin123!)

### 10.4 Chạy Thử Ứng Dụng

- [ ] **Kiểm tra Backend API**
  - [ ] Mở browser: `http://localhost:5230/api/auth` → Phải trả về 404 (OK, vì route không tồn tại)
  - [ ] Mở Postman/Thunder Client → Test API: `POST http://localhost:5230/api/auth/register`

- [ ] **Kiểm tra Frontend**
  - [ ] Mở browser: `http://localhost:5173` → Phải hiển thị trang chủ
  - [ ] Kiểm tra các trang chính: Home, About, Donate, Login, Register

- [ ] **Kiểm tra Đăng Ký và Đăng Nhập**
  - [ ] Đăng ký tài khoản mới
  - [ ] Kiểm tra email xác thực (nếu có cấu hình SMTP)
  - [ ] Đăng nhập với tài khoản vừa tạo

- [ ] **Kiểm tra Chức Năng Donation**
  - [ ] Vào trang Donate
  - [ ] Điền form quyên góp
  - [ ] Kiểm tra donation đã được lưu vào database chưa

### 10.5 Đọc Tài Liệu

- [ ] **Đọc README.md**
  - [ ] Hiểu cách setup project
  - [ ] Hiểu cấu trúc project

- [ ] **Đọc README_MEMBER.md (file này)**
  - [ ] Đọc phần Architecture Overview
  - [ ] Đọc phần Backend Deep Dive
  - [ ] Đọc phần Frontend Deep Dive
  - [ ] Đọc phần Database Schema
  - [ ] Đọc phần Authentication Flow
  - [ ] Đọc phần Feature Flows
  - [ ] Đọc phần Debugging Guide

- [ ] **Xem Code**
  - [ ] Xem các file Controller chính (AuthController, DonationController)
  - [ ] Xem các file Service chính (AuthService, DonationService)
  - [ ] Xem các file Page chính (DonatePage, LoginPage, RegisterPage)

### 10.6 Làm Quen Với Workflow

- [ ] **Hiểu Git Workflow**
  - [ ] Tạo branch mới: `git checkout -b feature/ten-tinh-nang`
  - [ ] Commit changes: `git add . && git commit -m "message"`
  - [ ] Push branch: `git push origin feature/ten-tinh-nang`
  - [ ] Tạo Pull Request

- [ ] **Hiểu Code Review Process**
  - [ ] Code phải được review trước khi merge
  - [ ] Fix comments từ reviewer

- [ ] **Hiểu Testing Process**
  - [ ] Test local trước khi push code
  - [ ] Test trên staging environment (nếu có)

### 10.7 Bắt Đầu Phát Triển

- [ ] **Chọn Task/Feature để làm**
  - [ ] Xem issue/ticket trong project management tool (nếu có)
  - [ ] Hoặc hỏi team lead để được assign task

- [ ] **Tạo Branch Mới**
  ```bash
  git checkout -b feature/ten-tinh-nang
  ```

- [ ] **Develop Feature**
  - [ ] Viết code
  - [ ] Test local
  - [ ] Commit changes

- [ ] **Push và Tạo Pull Request**
  ```bash
  git push origin feature/ten-tinh-nang
  # Tạo Pull Request trên GitHub/GitLab
  ```

### 10.8 Tài Liệu Tham Khảo

- [ ] **ASP.NET Core Documentation**
  - [ASP.NET Core Docs](https://docs.microsoft.com/aspnet/core)

- [ ] **React Documentation**
  - [React Docs](https://react.dev)

- [ ] **Entity Framework Core**
  - [EF Core Docs](https://learn.microsoft.com/ef/core)

- [ ] **JWT Authentication**
  - [JWT.io](https://jwt.io)
  - [ASP.NET Core JWT Auth](https://learn.microsoft.com/aspnet/core/security/authentication/jwt-authn)

---

**Kết Thúc README_MEMBER.md**

**Chúc bạn thành công với dự án GIVE-AID! 🎉**

---
        // Convert C# PascalCase → JSON camelCase
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        
        // Case-insensitive property matching
        options.JsonSerializerOptions.PropertyNameCaseInsensitive = true;
        
        // Ignore circular references (prevent infinite loops)
        // VD: Donation.User.Donations.User.Donations...
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    });
```

**Tại sao cần:**
- Frontend React dùng camelCase (`userName`), Backend C# dùng PascalCase (`UserName`)
- Circular references: Entity có navigation properties → serialize sẽ loop vô tận

#### B. Database Context Registration (Dòng 24-27)

```csharp
var conn = builder.Configuration.GetConnectionString("DefaultConnection")
           ?? "Server=(localdb)\\MSSQLLocalDB;Database=GiveAidDB;Trusted_Connection=True;";

builder.Services.AddDbContext<GiveAidContext>(options => options.UseSqlServer(conn));
```

**Luồng hoạt động:**
1. Đọc connection string từ `appsettings.Development.json` (nếu có)
2. Nếu không có, dùng default LocalDB
3. Đăng ký `GiveAidContext` vào DI container với scope `Scoped` (1 instance per request)

#### C. Dependency Injection - Services (Dòng 29-41)

```csharp
builder.Services.AddScoped<AuthService>();
builder.Services.AddScoped<DonationService>();
builder.Services.AddScoped<ProgramService>();
// ... tất cả services
```

**Lifetime Options:**
- **Scoped**: 1 instance per HTTP request (recommended cho services)
- **Singleton**: 1 instance cho toàn bộ app lifetime
- **Transient**: New instance mỗi lần resolve

**Tại sao dùng Scoped:**
- Mỗi HTTP request có 1 instance service → thread-safe
- DbContext cũng Scoped → service và context cùng scope

#### D. JWT Authentication Configuration (Dòng 43-66)

```csharp
var jwtKey = builder.Configuration["Jwt:Key"] ?? "very_secret_key_change_me_please";
var jwtIssuer = builder.Configuration["Jwt:Issuer"] ?? "Give_AID";
var key = Encoding.ASCII.GetBytes(jwtKey);

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false; // Set true in production
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidIssuer = jwtIssuer,              // "Give_AID"
        ValidateAudience = false,             // Không validate audience
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(key),  // HS256
        ValidateLifetime = true,              // Check token expiry
    };
});
```

**Token Validation Flow:**
1. Request đến với header `Authorization: Bearer {token}`
2. JWT middleware validate:
   - Signature (dùng secret key)
   - Issuer (phải là "Give_AID")
   - Expiry (token chưa hết hạn)
3. Nếu valid → set `HttpContext.User` với claims
4. Controller có thể dùng `User.FindFirst(ClaimTypes.NameIdentifier)` để lấy userId

#### E. CORS Configuration (Dòng 71-80)

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:5173", "http://localhost:3000", "http://localhost:5174")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();  // Cho phép gửi cookies/credentials (JWT)
    });
});
```

**Tại sao cần CORS:**
- Frontend chạy `localhost:5173`, Backend chạy `localhost:5230` → khác origin
- Browser block cross-origin requests nếu không có CORS
- `AllowCredentials()` cần để gửi JWT token trong header

#### F. Middleware Pipeline (Dòng 111-115)

```csharp
app.UseHttpsRedirection();      // Redirect HTTP → HTTPS
app.UseCors("AllowFrontend");   // Apply CORS policy
app.UseAuthentication();        // JWT authentication (MUST before Authorization)
app.UseAuthorization();         // Role-based authorization
app.MapControllers();           // Map routes to controllers
```

**Thứ tự quan trọng:**
1. `UseCors()` phải trước `UseAuthentication()`
2. `UseAuthentication()` phải trước `UseAuthorization()`

#### G. Data Seeding (Dòng 84-103)

```csharp
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    var context = services.GetRequiredService<GiveAidContext>();
    var configuration = services.GetRequiredService<IConfiguration>();
    
    try
    {
        // Seed admin user (nếu chưa có)
        await DataSeeder.SeedAdminUserAsync(context, configuration);
        
        // Seed NGOs và Programs (nếu database rỗng)
        await DataSeeder.SeedNGOsAndProgramsAsync(context);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error seeding data: {ex.Message}");
    }
}
```

**Chạy khi nào:**
- Mỗi lần app start (development/production)
- Nếu admin đã tồn tại → skip
- Nếu database có NGOs/Programs → skip

**Check file**: `Backend/Helpers/DataSeeder.cs` (Dòng 1-309)

### 3.2 Controllers → Services → Models Pattern

**Kiến trúc Layered:**

```
HTTP Request
    ↓
Controller (Validation + HTTP handling)
    ↓
Service (Business Logic)
    ↓
DbContext (Data Access)
    ↓
Database (SQL Server)
```

**Ví dụ: Donation Creation Flow**

#### Step 1: Controller nhận request

**File**: `Backend/Controllers/DonationController.cs` **Dòng 22-83**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] DonationDTO? dto)
{
    // Dòng 26-29: Null check
    if (dto == null)
    {
        return BadRequest(new { message = "Request body is required" });
    }

    // Dòng 32: Log for debugging
    Console.WriteLine($"[Donation] Received: Amount={dto.Amount}, Cause={dto.Cause}, ...");

    // Dòng 34-44: ModelState validation
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value?.Errors.Count > 0)
            .Select(x => new { Field = x.Key, Errors = x.Value?.Errors.Select(e => e.ErrorMessage) })
            .ToList();
        
        Console.WriteLine($"[Donation] ModelState validation failed: ...");
        return BadRequest(new { message = "Validation failed", errors });
    }
    
    // Dòng 47-69: Manual validation
    if (dto.Amount <= 0)
        return BadRequest(new { message = "Amount must be greater than 0" });
    if (string.IsNullOrWhiteSpace(dto.Cause))
        return BadRequest(new { message = "Cause is required" });
    // ... more validations

    try
    {
        // Dòng 73: Delegate to service
        var donation = await _donationService.CreateAsync(dto);
        Console.WriteLine($"[Donation] Successfully created donation ID: {donation.Id}");
        return Ok(donation);
    }
    catch (Exception ex)
    {
        // Dòng 79-81: Error handling
        Console.WriteLine($"[Donation] Exception: {ex.Message}");
        return BadRequest(new { message = "Failed to create donation", error = ex.Message });
    }
}
```

**Trách nhiệm Controller:**
- ✅ Validate HTTP request (null check, ModelState)
- ✅ Delegate business logic cho Service
- ✅ Handle HTTP responses (200 OK, 400 BadRequest, 500 InternalServerError)
- ✅ Logging for debugging

#### Step 2: Service xử lý business logic

**File**: `Backend/Services/DonationService.cs` **Dòng 24-135**

```csharp
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    // Dòng 27-50: Business validation
    if (string.IsNullOrWhiteSpace(dto.Email))
        throw new ArgumentException("Email is required");
    if (string.IsNullOrWhiteSpace(dto.FullName))
        throw new ArgumentException("Full name is required");
    if (string.IsNullOrWhiteSpace(dto.Cause))
        throw new ArgumentException("Cause is required");

    // Dòng 43: Prepare donor name (anonymous check)
    var donorName = dto.Anonymous ? "Anonymous" : dto.FullName.Trim();
    var donorEmail = dto.Email.Trim();
    
    // Dòng 52-67: Create donation entity
    var donation = new Donation
    {
        Amount = dto.Amount,
        CauseName = string.IsNullOrWhiteSpace(dto.Cause) ? "General Donation" : dto.Cause.Trim(),
        PaymentStatus = "Success",  // Dummy payment luôn thành công
        PaymentMethod = string.IsNullOrWhiteSpace(dto.PaymentMethod) ? "Card" : dto.PaymentMethod.Trim(),
        UserId = dto.UserId,        // null nếu guest donate
        ProgramId = dto.ProgramId,  // Link to program if provided
        TransactionReference = "TRX-" + Guid.NewGuid().ToString("N").Substring(0, 12),
        DonorName = donorName,
        DonorEmail = donorEmail,
        DonorPhone = string.IsNullOrWhiteSpace(dto.Phone) ? null : dto.Phone.Trim(),
        DonorAddress = string.IsNullOrWhiteSpace(dto.Address) ? null : dto.Address.Trim(),
        IsAnonymous = dto.Anonymous,
        SubscribeNewsletter = dto.Newsletter
    };

    try
    {
        // Dòng 71-74: Save to database
        Console.WriteLine($"[DonationService] Creating donation: ...");
        _context.Donations.Add(donation);
        await _context.SaveChangesAsync();
        Console.WriteLine($"[DonationService] Donation created successfully with ID: {donation.Id}");

        // Dòng 77-111: Send email confirmation (fire-and-forget)
        if (!string.IsNullOrWhiteSpace(donorEmail) && !dto.Anonymous)
        {
            _ = Task.Run(async () =>
            {
                try
                {
                    var emailBody = EmailTemplate.DonationReceiptTemplate(
                        donorName,
                        donation.Amount,
                        donation.CauseName,
                        donation.TransactionReference,
                        donation.CreatedAt
                    );
                    await _emailService.SendEmailAsync(
                        donorEmail,
                        "Thank you for your donation",
                        emailBody
                    );
                    Console.WriteLine($"[DonationService] Confirmation email sent to {donorEmail}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"[DonationService] Failed to send email: {ex.Message}");
                    // Fail silently - không block donation success
                }
            });
        }
        
        return donation;
    }
    catch (DbUpdateException dbEx)
    {
        // Dòng 119-128: Database error handling
        Console.WriteLine($"[DonationService] Database error: {dbEx.InnerException?.Message}");
        throw new Exception($"Database error while creating donation: {dbEx.InnerException?.Message}", dbEx);
    }
}
```

**Trách nhiệm Service:**
- ✅ Business logic validation
- ✅ Create entity objects
- ✅ Save to database via DbContext
- ✅ Send emails (fire-and-forget)
- ✅ Error handling với specific exceptions

**Fire-and-Forget Pattern:**
- `_ = Task.Run(async () => { ... })` → không await
- Email gửi background, không block response
- Response time: ~100-200ms (thay vì ~1-2s nếu await email)

#### Step 3: Entity Framework save to database

**File**: `Backend/Models/Donation.cs`

```csharp
public class Donation
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public string CauseName { get; set; } = string.Empty;
    // ... other properties
}
```

**EF Core translates to SQL:**
```sql
INSERT INTO Donations 
  (Amount, CauseName, PaymentStatus, TransactionReference, DonorName, DonorEmail, UserId, ProgramId, ...)
VALUES 
  (100.00, 'Education for Children', 'Success', 'TRX-a3f9b2c1d4e5', 'John Doe', 'john@example.com', 10, 5, ...)
```

### 3.3 Tất Cả API Endpoints

| Controller | Route Base | Endpoints | Auth Required | File Lines |
|------------|-----------|-----------|---------------|------------|
| **AuthController** | `/api/auth` | `POST /register`<br>`POST /login`<br>`POST /verify-email`<br>`POST /resend-verification`<br>`POST /forgot-password`<br>`POST /reset-password` | ❌ No | 1-205 |
| **DonationController** | `/api/donation` | `POST /` (create)<br>`GET /my-donations` (user history)<br>`GET /` (admin: all)<br>`GET /{id}` (admin: by id) | Varies | 1-130 |
| **ProgramController** | `/api/program` | `GET /` (all programs)<br>`GET /{id}` (by id)<br>`GET /{id}/stats` (program stats)<br>`POST /{id}/register` (register interest)<br>`POST /` (admin: create)<br>`PUT /{id}` (admin: update)<br>`DELETE /{id}` (admin: delete) | Varies | - |
| **AdminController** | `/api/admin` | `GET /users`<br>`GET /users/{id}`<br>`PUT /users/{id}/role`<br>`DELETE /users/{id}`<br>`GET /donations`<br>`GET /donations/{id}`<br>`GET /queries`<br>`POST /queries/{id}/reply` | ✅ Admin | - |
| **ProfileController** | `/api/profile` | `GET /` (get profile)<br>`PUT /` (update profile)<br>`POST /change-password` | ✅ User | - |
| **NGOController** | `/api/ngo` | `GET /` (all NGOs)<br>`GET /{id}` (by id)<br>`POST /` (admin: create)<br>`PUT /{id}` (admin: update)<br>`DELETE /{id}` (admin: delete) | Varies | - |
| **PartnerController** | `/api/partner` | `GET /` (all partners)<br>`POST /` (admin: create)<br>`PUT /{id}` (admin: update)<br>`DELETE /{id}` (admin: delete) | Varies | - |
| **GalleryController** | `/api/gallery` | `GET /` (all images)<br>`POST /` (admin: upload)<br>`DELETE /{id}` (admin: delete) | Varies | - |
| **QueryController** | `/api/query` | `POST /` (submit query)<br>`GET /` (admin: all)<br>`POST /{id}/reply` (admin: reply) | Varies | - |
| **AboutController** | `/api/about` | `GET /{key}` (get section)<br>`POST /` (admin: create)<br>`PUT /{id}` (admin: update) | Varies | - |
| **InvitationController** | `/api/invitation` | `POST /send` (send invitation) | ✅ User | - |

**Legend:**
- ❌ No: Không cần authentication
- ✅ User: Cần login (bất kỳ user nào)
- ✅ Admin: Cần role "Admin"
- Varies: Tùy endpoint (public hoặc protected)

---

## 4. Frontend Deep Dive

### 4.1 App.jsx - Routes & Providers

**File**: `FRONTEND/src/App.jsx` **Dòng 1-120**

**Chức năng chính:**

#### A. AOS Initialization (Dòng 36-49)

```jsx
useEffect(() => {
  // Initialize AOS (Animate On Scroll) khi component mount
  import('aos').then(AOS => {
    AOS.init({
      duration: 700,
      easing: 'ease-in-out',
      once: true,        // Chỉ animate 1 lần khi scroll đến
      offset: 50         // Offset 50px từ top để trigger animation
    });
  });

  // Import AOS CSS
  import('aos/dist/aos.css');
}, []);
```

**Tại sao dùng dynamic import:**
- Giảm bundle size ban đầu
- AOS chỉ load khi cần (lazy loading)

#### B. Routes Structure (Dòng 52-101)

```jsx
<BrowserRouter>
  <AuthProvider>  {/* Global auth state */}
    <ScrollToTop />  {/* Restore scroll position on route change */}
    
    <Routes>
      {/* Admin Routes - Protected */}
      <Route path="/admin/*" element={
        <AdminRoute>  {/* Guard: check authentication & admin role */}
          <AdminLayout>  {/* Admin sidebar + header */}
            <Routes>
              <Route path="users" element={<UsersPage />} />
              <Route path="donations" element={<DonationsPageAdmin />} />
              <Route path="programs" element={<ProgramsPageAdmin />} />
              <Route path="ngos" element={<NGOsPageAdmin />} />
              <Route path="gallery" element={<GalleryPageAdmin />} />
              <Route path="partners" element={<PartnersPageAdmin />} />
              <Route path="about" element={<AboutPageAdmin />} />
              <Route path="queries" element={<QueriesPage />} />
              <Route path="*" element={<UsersPage />} />  {/* Default */}
            </Routes>
          </AdminLayout>
        </AdminRoute>
      } />
      
      {/* Public Routes */}
      <Route path="/*" element={
        <Layout>  {/* Navbar + Footer wrapper */}
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/donate" element={<DonatePage />} />
            <Route path="/login" element={<LoginPage />} />
            <Route path="/register" element={<RegisterPage />} />
            {/* ... more routes */}
          </Routes>
        </Layout>
      } />
    </Routes>
    
    {/* Global Toast Notifications */}
    <ToastContainer
      position="top-right"
      autoClose={3000}
      hideProgressBar={false}
      newestOnTop={false}
      closeOnClick
      rtl={false}
      pauseOnFocusLoss
      draggable
      pauseOnHover
      theme="light"
    />
  </AuthProvider>
</BrowserRouter>
```

**Route Nesting:**
- Admin routes: `/admin/*` → nested routes bên trong `AdminRoute`
- Public routes: `/*` → nested routes bên trong `Layout`

### 4.2 Authentication Context (Global State)

**File**: `FRONTEND/src/contexts/AuthContext.jsx` **Dòng 1-167**

**State Management:**

```jsx
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);        // User object
  const [isLoading, setIsLoading] = useState(true);  // Loading state

  // Decode JWT token để lấy user info
  const decodeToken = (token) => {
    try {
      const base64Url = token.split('.')[1];
      const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
      const jsonPayload = decodeURIComponent(atob(base64).split('').map((c) => {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
      }).join(''));
      const decoded = JSON.parse(jsonPayload);
      
      // Map JWT claims to user data (hỗ trợ nhiều format claims)
      return {
        id: decoded['nameid'] || decoded['http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier'] || decoded.sub,
        email: decoded['email'] || decoded['http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress'],
        fullName: decoded['fullName'] || decoded['http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name'],
        role: decoded['role'] || decoded['http://schemas.microsoft.com/ws/2008/06/identity/claims/role'] || 'User'
      };
    } catch (error) {
      return null;
    }
  };

  // Load user from localStorage on mount
  useEffect(() => {
    // Try localStorage user object first
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      try {
        const userData = JSON.parse(savedUser);
        if (userData && userData.email) {
          setUser(userData);
          setIsLoading(false);
          return;
        }
      } catch (error) {
        // Continue to decode token
      }
    }
    
    // If no saved user, decode token
    const token = localStorage.getItem('token');
    if (token) {
      const decoded = decodeToken(token);
      if (decoded && decoded.email) {
        const userData = {
          fullName: decoded.fullName || 'User',
          email: decoded.email,
          id: decoded.id,
          role: decoded.role || 'User'
        };
        setUser(userData);
        localStorage.setItem('user', JSON.stringify(userData));
      }
    }
    setIsLoading(false);
  }, []);

  const login = async (usernameOrEmail, password) => {
    const result = await loginService(usernameOrEmail, password);
    if (result.success) {
      const token = localStorage.getItem('token');
      if (token) {
        const decoded = decodeToken(token);
        if (decoded && decoded.email) {
          const userData = {
            fullName: decoded.fullName || 'User',
            email: decoded.email,
            id: decoded.id,
            role: decoded.role || 'User'
          };
          setUser(userData);
          localStorage.setItem('user', JSON.stringify(userData));
        }
      }
    }
    return result;
  };

  const logout = () => {
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    setUser(null);
    window.location.href = '/login';
  };

  return (
    <AuthContext.Provider value={{
      user,
      isLoading,
      isAuthenticated: !!user,
      isAdmin: user?.role === 'Admin',
      login,
      logout
    }}>
      {children}
    </AuthContext.Provider>
  );
};
```

**Key Functions:**
- `decodeToken(token)` — Decode JWT để lấy user info
- `login(usernameOrEmail, password)` — Login và save token + user
- `logout()` — Clear localStorage và redirect

**State Properties:**
- `user` — User object `{ id, email, fullName, role }`
- `isAuthenticated` — Boolean (`!!user`)
- `isAdmin` — Boolean (`user?.role === 'Admin'`)
- `isLoading` — Boolean (true khi đang init từ localStorage)

### 4.3 API Service Layer (Axios)

**File**: `FRONTEND/src/services/api.js`

```javascript
import axios from 'axios';

// Tạo Axios instance với base URL
const api = axios.create({
  baseURL: 'http://localhost:5230/api',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request Interceptor: Thêm JWT token vào mọi request
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response Interceptor: Handle 401 errors (auto-logout)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired hoặc invalid
      localStorage.removeItem('token');
      localStorage.removeItem('user');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

**Usage trong Service Files:**

```javascript
// authServices.js
import api from './api';

export const login = async (usernameOrEmail, password) => {
  const response = await api.post('/auth/login', { usernameOrEmail, password });
  if (response.data.token) {
    localStorage.setItem('token', response.data.token);
  }
  return response.data;
};

export const register = async (userData) => {
  const response = await api.post('/auth/register', userData);
  return response.data;
};
```

### 4.4 DonatePage.jsx - Donation Form

**File**: `FRONTEND/src/pages/DonatePage.jsx` **Dòng 1-767**

**Key Sections:**

#### A. State Management (Dòng 9-25)

```jsx
const [programs, setPrograms] = useState([]);  // Danh sách programs
const [programStats, setProgramStats] = useState({});  // Stats cho mỗi program
const [isSubmitting, setIsSubmitting] = useState(false);

const [formData, setFormData] = useState({
  amount: '',
  programId: '',
  fullName: '',
  email: '',
  phone: '',
  address: '',
  paymentMethod: 'credit',
  anonymous: false,
  newsletter: true,
});
```

#### B. Fetch Programs & Stats (Dòng 27-50)

```jsx
useEffect(() => {
  const fetchPrograms = async () => {
    try {
      // Fetch all programs
      const res = await programService.getAll();
      const programsData = Array.isArray(res) ? res : [];
      setPrograms(programsData);
      
      // Fetch stats cho mỗi program (parallel)
      const statsPromises = programsData.map(async (program) => {
        try {
          const stats = await programService.getStats(program.id);
          return { programId: program.id, stats };
        } catch (error) {
          console.error(`Failed to load stats for program ${program.id}:`, error);
          return { programId: program.id, stats: null };
        }
      });
      
      const statsResults = await Promise.all(statsPromises);
      const statsMap = {};
      statsResults.forEach(({ programId, stats }) => {
        statsMap[programId] = stats;
      });
      setProgramStats(statsMap);
    } catch (error) {
      console.error('Failed to load programs:', error);
      toast.error('Failed to load programs');
    }
  };
  
  fetchPrograms();
}, []);
```

#### C. Form Submit Handler (Dòng 150-250)

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  // Client-side validation
  if (!formData.amount || parseFloat(formData.amount) <= 0) {
    toast.error('Amount must be greater than 0');
    return;
  }
  
  if (!formData.fullName || !formData.email) {
    toast.error('Full name and email are required');
    return;
  }
  
  // Find selected program
  const selectedProgram = programs.find(p => p.id === parseInt(formData.programId));
  const causeName = selectedProgram ? selectedProgram.title : "General Donation";
  
  // Prepare payload
  const donationData = {
    amount: parseFloat(formData.amount),
    cause: causeName,
    programId: parseInt(formData.programId) || null,
    fullName: formData.fullName,
    email: formData.email,
    phone: formData.phone || null,
    address: formData.address || null,
    paymentMethod: formData.paymentMethod,
    anonymous: formData.anonymous,
    newsletter: formData.newsletter,
    userId: user?.id || null  // null nếu guest
  };
  
  setIsSubmitting(true);
  
  try {
    // Call API
    const result = await donationService.create(donationData);
    
    // Success
    toast.success('Donation successful! Check your email for confirmation.');
    navigate('/donation-history');
  } catch (error) {
    // Error handling
    const message = error.response?.data?.message || 'Donation failed';
    toast.error(message);
    console.error('Donation error:', error);
  } finally {
    setIsSubmitting(false);
  }
};
```

#### D. Program Selection Cards (Dòng 250-350)

```jsx
{programs.slice(0, 6).map((program, index) => {
  const stats = programStats[program.id];
  const isSelected = formData.programId === program.id.toString();

  return (
    <div
      key={program.id}
      className={`category-card h-100 ${isSelected ? 'selected' : ''}`}
      onClick={() => handleProgramCardClick(program.id)}
    >
      <h5 className="category-title">{program.title}</h5>
      <p className="category-description" style={{
        display: '-webkit-box',
        WebkitLineClamp: 2,
        WebkitBoxOrient: 'vertical',
        overflow: 'hidden',
        textOverflow: 'ellipsis',
        minHeight: '3em',
        lineHeight: '1.5em'
      }}>
        {program.description || "Supporting meaningful charitable activities."}
      </p>
      
      {/* Progress Bar */}
      {stats && stats.goalAmount && stats.goalAmount > 0 ? (
        <div className="mb-2">
          <div className="progress" style={{ height: '8px' }}>
            <div
              className="progress-bar bg-success"
              style={{ width: `${Math.min(stats.progressPercentage || 0, 100)}%` }}
            />
          </div>
          <small className="text-muted">
            {formatCurrency(stats.totalDonations)} / {formatCurrency(stats.goalAmount)}
          </small>
        </div>
      ) : null}
      
      {/* Registration Count */}
      {stats && stats.registrationCount !== undefined && (
        <small className="text-muted">
          <i className="fas fa-users me-1"></i>
          <strong>{stats.registrationCount || 0}</strong> registered
        </small>
      )}
    </div>
  );
})}
```

### 4.5 Admin Route Guard

**File**: `FRONTEND/src/components/admin/AdminRoute.jsx`

```jsx
import { useAuth } from '../../contexts/AuthContext';
import { Navigate } from 'react-router-dom';

export default function AdminRoute({ children }) {
  const { isAuthenticated, isAdmin, isLoading } = useAuth();
  
  if (isLoading) {
    return <div className="text-center py-5">Loading...</div>;
  }
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  if (!isAdmin) {
    return (
      <div className="container py-5">
        <div className="alert alert-danger">
          <h4>Access Denied</h4>
          <p>You don't have permission to access this page.</p>
        </div>
      </div>
    );
  }
  
  return children;
}
```

**Flow:**
1. Check `isLoading` → show loading
2. Check `isAuthenticated` → redirect to login
3. Check `isAdmin` → show 403 error
4. All passed → render children (admin pages)

---

## 5. Database Schema & Migrations

### 5.1 Entity Relationship Diagram

```
┌─────────────┐       ┌──────────────┐       ┌──────────────┐
│   Users     │──────<│  Donations   │>──────│  NgoPrograms │
│             │ 1:N   │              │  N:1  │              │
│ Id (PK)     │       │ Id (PK)      │       │ Id (PK)      │
│ Email       │       │ Amount       │       │ Title        │
│ PasswordHash│       │ CauseName    │       │ Description  │
│ Role        │       │ UserId (FK)  │       │ NGOId (FK)   │
│ EmailVerified│       │ ProgramId(FK)│       │ GoalAmount   │
└─────────────┘       │ DonorName    │       └──────────────┘
       │              │ DonorEmail   │              │
       │              │ PaymentStatus│              │
       │              └──────────────┘              │
       │                                            │
       └────────────<│ProgramReg.   │>─────────────┘
                  1:N │              │ N:1
                      │ Id (PK)      │
                      │ UserId (FK)  │
                      │ ProgramId(FK)│
                      │ Email        │
                      └──────────────┘

┌──────────────┐
│   NGOs       │──────────┐
│              │ 1:N      │
│ Id (PK)      │          │
│ Name         │          ▼
│ Description  │    ┌──────────────┐
│ ContactEmail │    │  NgoPrograms │
└──────────────┘    └──────────────┘
```

### 5.2 Key Tables

#### Users Table

**File**: `Backend/Models/User.cs` **Dòng 1-45**

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `Id` | int | PK, IDENTITY(1,1) | Auto-increment |
| `Username` | string(50) | UNIQUE, NOT NULL | Login username |
| `Email` | string(255) | UNIQUE, NOT NULL, INDEX | User email |
| `PasswordHash` | string(255) | NOT NULL | BCrypt hashed password |
| `FullName` | string(100) | NOT NULL | Display name |
| `Phone` | string(50) | NULL | Contact number |
| `Address` | string(500) | NULL | User address |
| `Role` | string(20) | NOT NULL, DEFAULT='User' | 'User' or 'Admin' |
| `EmailVerified` | bit | NOT NULL, DEFAULT=0 | Email confirmation status |
| `EmailVerificationToken` | string(255) | NULL | Hashed verification token |
| `EmailVerificationTokenExpiry` | datetime2 | NULL | Token expiry (24h from creation) |
| `PasswordResetToken` | string(255) | NULL | Hashed reset token |
| `PasswordResetTokenExpiry` | datetime2 | NULL | Token expiry (1h from creation) |
| `CreatedAt` | datetime2 | NOT NULL, DEFAULT(GETUTCDATE()) | Account creation time |

**Indexes:**
- `IX_Users_Email` (UNIQUE)
- `IX_Users_Username` (UNIQUE)

**Migrations:**
- `20251021161050_InitialCreate.cs` — Creates table
- `20251031044158_AddUsernameToUser.cs` — Adds Username column
- `20251031092945_AddEmailVerificationToUser.cs` — Adds email verification
- `20251101120000_AddPasswordResetToUser.cs` — Adds password reset

#### Donations Table

**File**: `Backend/Models/Donation.cs`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `Id` | int | PK, IDENTITY(1,1) | Auto-increment |
| `Amount` | decimal(18,2) | NOT NULL | Donation amount |
| `CauseName` | string(200) | NOT NULL | Program/cause name |
| `PaymentStatus` | string(20) | NOT NULL, DEFAULT='Pending' | 'Success'/'Failed'/'Pending' |
| `PaymentMethod` | string(50) | NULL | 'Credit Card'/'Debit Card'/'Bank Transfer' |
| `UserId` | int | FK→Users.Id, NULL | Donor (if logged in) |
| `ProgramId` | int | FK→NgoPrograms.Id, NULL | Linked program |
| `TransactionReference` | string(50) | UNIQUE, NOT NULL | TRX-xxxxxxxxxxxxx |
| `DonorName` | string(100) | NOT NULL | Donor name (or "Anonymous") |
| `DonorEmail` | string(255) | NOT NULL | Donor email |
| `DonorPhone` | string(50) | NULL | Donor phone |
| `DonorAddress` | string(500) | NULL | Donor address |
| `IsAnonymous` | bit | NOT NULL, DEFAULT=0 | Hide donor name |
| `SubscribeNewsletter` | bit | NOT NULL, DEFAULT=0 | Newsletter opt-in |
| `CreatedAt` | datetime2 | NOT NULL, DEFAULT(GETUTCDATE()) | Donation timestamp |

**Foreign Keys:**
- `FK_Donations_Users_UserId` → `Users.Id` (ON DELETE SET NULL)
- `FK_Donations_NgoPrograms_ProgramId` → `NgoPrograms.Id` (ON DELETE SET NULL)

**Indexes:**
- `IX_Donations_TransactionReference` (UNIQUE)
- `IX_Donations_UserId` (non-unique)
- `IX_Donations_ProgramId` (non-unique)
- `IX_Donations_CreatedAt` (non-unique, for sorting)

**Migrations:**
- `20251021161050_InitialCreate.cs` — Creates table
- `20251105155757_AddProgramGoalAndDonationLink.cs` — Adds ProgramId FK

#### NgoPrograms Table

**File**: `Backend/Models/NgoProgram.cs`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `Id` | int | PK, IDENTITY(1,1) | Auto-increment |
| `Title` | string(200) | NOT NULL | Program name |
| `Description` | nvarchar(MAX) | NULL | Program details (HTML allowed) |
| `Location` | string(200) | NULL | Where program runs |
| `StartDate` | datetime2 | NULL | Program start date |
| `EndDate` | datetime2 | NULL | Program end date |
| `Status` | string(20) | NOT NULL, DEFAULT='Active' | 'Active'/'Completed'/'Cancelled' |
| `NGOId` | int | FK→NGOs.Id, NULL | Owning NGO |
| `GoalAmount` | decimal(18,2) | NULL | Fundraising goal |
| `CreatedAt` | datetime2 | NOT NULL, DEFAULT(GETUTCDATE()) | Creation timestamp |

**Foreign Keys:**
- `FK_NgoPrograms_NGOs_NGOId` → `NGOs.Id` (ON DELETE SET NULL)

**Migrations:**
- `20251021161050_InitialCreate.cs` — Creates table
- `20251105155757_AddProgramGoalAndDonationLink.cs` — Adds GoalAmount

#### ProgramRegistrations Table

**File**: `Backend/Models/ProgramRegistration.cs`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `Id` | int | PK, IDENTITY(1,1) | Auto-increment |
| `ProgramId` | int | FK→NgoPrograms.Id, NOT NULL | Program ID |
| `UserId` | int | FK→Users.Id, NULL | User ID (if logged in) |
| `Email` | string(255) | NOT NULL | Registrant email |
| `FullName` | string(100) | NOT NULL | Registrant name |
| `Phone` | string(50) | NULL | Registrant phone |
| `Message` | nvarchar(MAX) | NULL | Additional message |
| `CreatedAt` | datetime2 | NOT NULL, DEFAULT(GETUTCDATE()) | Registration timestamp |

**Foreign Keys:**
- `FK_ProgramRegistrations_NgoPrograms_ProgramId` → `NgoPrograms.Id` (ON DELETE CASCADE)
- `FK_ProgramRegistrations_Users_UserId` → `Users.Id` (ON DELETE SET NULL)

**Unique Constraint:**
- `UQ_ProgramRegistrations_ProgramId_Email` — Prevent duplicate registration (same email + same program)

**Migrations:**
- `20251103131053_AddProgramRegistration.cs` — Creates table

### 5.3 Migrations

**Location**: `Backend/Migrations/`

**How to Create Migration:**
```bash
cd Backend
dotnet ef migrations add MigrationName
```

**How to Apply Migration:**
```bash
dotnet ef database update
```

**How to Rollback:**
```bash
dotnet ef database update PreviousMigrationName
```

**Important Migrations:**

1. **`20251021161050_InitialCreate.cs`**
   - Creates all base tables: Users, Donations, NGOs, NgoPrograms, Partners, Gallery, Queries, Invitations, Causes, AboutSections

2. **`20251031044158_AddUsernameToUser.cs`**
   - Adds `Username` column to Users table
   - Creates unique index on Username

3. **`20251031092945_AddEmailVerificationToUser.cs`**
   - Adds `EmailVerificationToken`, `EmailVerificationTokenExpiry`, `EmailVerified` columns
   - For email verification flow

4. **`20251101120000_AddPasswordResetToUser.cs`**
   - Adds `PasswordResetToken`, `PasswordResetTokenExpiry` columns
   - For password reset flow

5. **`20251103131053_AddProgramRegistration.cs`**
   - Creates `ProgramRegistrations` table
   - Adds unique constraint on (ProgramId, Email)

6. **`20251105155757_AddProgramGoalAndDonationLink.cs`**
   - Adds `GoalAmount` to NgoPrograms
   - Adds `ProgramId` FK to Donations

### 5.4 Data Seeding

**File**: `Backend/Helpers/DataSeeder.cs` **Dòng 1-309**

**Chạy khi nào:**
- Mỗi lần app start (trong `Program.cs` Dòng 84-103)

**Seed Admin User:**
```csharp
public static async Task SeedAdminUserAsync(GiveAidContext context, IConfiguration config)
{
    // Check if admin exists
    var adminExists = await context.Users.AnyAsync(u => u.Email == adminEmail);
    if (adminExists) return;
    
    // Create admin user
    var admin = new User
    {
        Username = adminUsername,
        Email = adminEmail,
        PasswordHash = PasswordHasher.Hash(adminPassword),
        FullName = adminFullName,
        Role = "Admin",
        EmailVerified = true,
        CreatedAt = DateTime.UtcNow
    };
    
    context.Users.Add(admin);
    await context.SaveChangesAsync();
}
```

**Seed NGOs & Programs:**
```csharp
public static async Task SeedNGOsAndProgramsAsync(GiveAidContext context)
{
    // Only seed if database is empty
    if (await context.NGOs.AnyAsync()) return;
    
    // Create NGOs
    var ngo1 = new NGO { Name = "Education Foundation", ... };
    context.NGOs.Add(ngo1);
    
    // Create Programs
    var program1 = new NgoProgram { Title = "Scholarship Program", NGOId = ngo1.Id, ... };
    context.NgoPrograms.Add(program1);
    
    await context.SaveChangesAsync();
}
```

**Default Admin Credentials:**
- Email: `admin@giveaid.org`
- Username: `admin`
- Password: `Admin123!`

---

## 6. Authentication & Authorization Flow

### 6.1 Registration → Email Verification → Login Flow

**Luồng hoàn chỉnh:**

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: User Registration                                    │
└─────────────────────────────────────────────────────────────┘
Frontend: RegisterPage.jsx
  │
  ├─> User điền form (email, username, password, fullName, ...)
  │
  ├─> POST /api/auth/register
  │   Body: { email, username, password, fullName, phone, address }
  │
Backend: AuthController.cs Dòng 24-61
  │
  ├─> AuthController.Register() (Dòng 24)
  │   ├─ Dòng 26-29: Null check
  │   ├─ Dòng 35-44: ModelState validation
  │   └─ Dòng 47: Delegate to service
  │
  ├─> AuthService.RegisterAsync() (Dòng 29)
  │   ├─ Dòng 32-43: Check email/username exists
  │   │   └─> Query: SELECT * FROM Users WHERE Email = ? OR Username = ?
  │   │
  │   ├─ Dòng 46-56: Create user entity
  │   │   ├─ Dòng 51: Hash password → PasswordHasher.Hash(req.Password)
  │   │   │   └─> BCrypt hash với work factor 12
  │   │   │
  │   │   ├─ Dòng 59: Generate verification token
  │   │   │   └─> Random 32-char token
  │   │   │
  │   │   ├─ Dòng 60: Hash token before saving
  │   │   │   └─> PasswordHasher.Hash(verificationToken)
  │   │   │
  │   │   ├─ Dòng 61: Set expiry = UtcNow + 24 hours
  │   │   └─ Dòng 62: EmailVerified = false
  │   │
  │   ├─ Dòng 64-65: Save to database
  │   │   └─> INSERT INTO Users (...)
  │   │
  │   └─ Dòng 74-111: Send verification email (fire-and-forget)
  │       └─> EmailService.SendEmailAsync()
  │           └─> Email body: VerificationEmailTemplate()
  │           └─> Link: http://localhost:5173/verify-email?token=xxx
  │
  └─> Return: { message: "Registration successful! Please check your email..." }

┌─────────────────────────────────────────────────────────────┐
│ STEP 2: User clicks verification link in email              │
└─────────────────────────────────────────────────────────────┘
Frontend: VerifyEmailPage.jsx
  │
  ├─> Extract token from URL: ?token=xxx
  │
  ├─> POST /api/auth/verify-email
  │   Body: { token: "xxx..." }
  │
Backend: AuthController.cs Dòng 100-130
  │
  ├─> AuthController.VerifyEmail() (Dòng 100)
  │   └─> AuthService.VerifyEmailAsync() (Dòng 139)
  │       │
  │       ├─ Dòng 164-193: Find user by hashed token
  │       │   └─> Query: SELECT * FROM Users WHERE EmailVerificationToken IS NOT NULL
  │       │   └─> Loop through users, verify token với BCrypt.Verify()
  │       │   └─> Match found → user = matched user
  │       │
  │       ├─ Dòng 202-204: Check token expiry
  │       │   └─> if (user.EmailVerificationTokenExpiry < DateTime.UtcNow)
  │       │       → return (false, "Token expired")
  │       │
  │       ├─ Dòng 211: Set EmailVerified = true
  │       ├─ Dòng 212-213: Clear verification token
  │       └─ Dòng 214: SaveChangesAsync()
  │
  └─> Return: { message: "Email verified successfully!" }
      Frontend: Show success → Redirect to login

┌─────────────────────────────────────────────────────────────┐
│ STEP 3: User Login                                          │
└─────────────────────────────────────────────────────────────┘
Frontend: LoginPage.jsx
  │
  ├─> User nhập usernameOrEmail + password
  │
  ├─> POST /api/auth/login
  │   Body: { usernameOrEmail, password }
  │
Backend: AuthController.cs Dòng 62-98
  │
  ├─> AuthController.Login() (Dòng 62)
  │   └─> AuthService.LoginAsync() (Dòng 292)
  │       │
  │       ├─ Dòng 295: Check if input is email or username
  │       │   └─> isEmail = req.UsernameOrEmail.Contains("@")
  │       │
  │       ├─ Dòng 298-300: Find user
  │       │   └─> if (isEmail)
  │       │       → SELECT * FROM Users WHERE Email = ?
  │       │   └─> else
  │       │       → SELECT * FROM Users WHERE Username = ?
  │       │
  │       ├─ Dòng 302-303: User not found → return error
  │       │
  │       ├─ Dòng 306-308: Verify password
  │       │   └─> PasswordHasher.Verify(req.Password, user.PasswordHash)
  │       │   └─> BCrypt.Verify() → true/false
  │       │
  │       ├─ Dòng 311-312: Check email verified
  │       │   └─> if (!user.EmailVerified) → return error
  │       │
  │       └─ Dòng 315: Generate JWT token
  │           └─> JwtHelper.GenerateToken(user, _config)
  │               ├─ Create claims:
  │               │   - ClaimTypes.NameIdentifier = user.Id
  │               │   - ClaimTypes.Email = user.Email
  │               │   - ClaimTypes.Name = user.FullName
  │               │   - ClaimTypes.Role = user.Role
  │               │
  │               ├─ Sign với secret key (HS256)
  │               └─ Set expiry = UtcNow + 7 days (default)
  │
  └─> Return: { message: "Login successful", token: "eyJhbGci..." }

Frontend: LoginPage.jsx
  │
  ├─> Receive token
  │
  ├─> AuthContext.login(token)
  │   ├─ localStorage.setItem('token', token)
  │   ├─ Decode JWT → extract user info
  │   ├─ setUser({ id, email, fullName, role })
  │   └─ localStorage.setItem('user', JSON.stringify(user))
  │
  └─> Navigate to home/dashboard
```

### 6.2 JWT Token Structure

**Generated by**: `Backend/Helpers/JwtHelper.cs` **Dòng 13-47**

**Token Format:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1laWQiOiIxMjMiLCJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJyb2xlIjoiVXNlciIsImlzcyI6IkdpdmVfQUlEIiwiZXhwIjoxNjk5OTk5OTk5fQ.signature
```

**Header (Base64 decoded):**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload (Base64 decoded):**
```json
{
  "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "123",
  "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress": "user@example.com",
  "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "John Doe",
  "http://schemas.microsoft.com/ws/2008/06/identity/claims/role": "User",
  "iss": "Give_AID",
  "exp": 1699999999
}
```

**Signature:**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**Usage trong Backend Controller:**
```csharp
// Lấy user ID từ JWT token
var userIdClaim = User.FindFirst(ClaimTypes.NameIdentifier);
if (userIdClaim == null || !int.TryParse(userIdClaim.Value, out int userId))
{
    return Unauthorized(new { message = "Invalid token" });
}

// Lấy role
var role = User.FindFirst(ClaimTypes.Role)?.Value;
```

### 6.3 Protected Routes (Backend)

**Middleware**: JWT Bearer Authentication (configured in `Program.cs` Dòng 48-66)

**Usage trong Controllers:**

```csharp
// Require any authenticated user
[Authorize]
[HttpGet("my-donations")]
public async Task<IActionResult> GetMyDonations()
{
    var userIdClaim = User.FindFirst(ClaimTypes.NameIdentifier);
    int userId = int.Parse(userIdClaim.Value);
    // ...
}

// Require Admin role
[Authorize(Roles = "Admin")]
[HttpGet("users")]
public async Task<IActionResult> GetAllUsers()
{
    // Only admins can access
    // ...
}
```

**How it works:**
1. Request đến với header `Authorization: Bearer {token}`
2. JWT Bearer middleware validate token:
   - Check signature với secret key
   - Check issuer (phải là "Give_AID")
   - Check expiry (token chưa hết hạn)
3. Nếu valid → set `HttpContext.User` với claims
4. `[Authorize]` attribute check `HttpContext.User.Identity.IsAuthenticated`
5. `[Authorize(Roles = "Admin")]` check role claim

### 6.4 Protected Routes (Frontend)

**Guard Component**: `FRONTEND/src/components/admin/AdminRoute.jsx`

**Flow:**
```
User navigates to /admin/users
  ↓
AdminRoute component mounts
  ↓
Check isLoading → Show loading if true
  ↓
Check isAuthenticated → Redirect to /login if false
  ↓
Check isAdmin → Show 403 if false
  ↓
All passed → Render children (admin pages)
```

**Code:**
```jsx
export default function AdminRoute({ children }) {
  const { isAuthenticated, isAdmin, isLoading } = useAuth();
  
  if (isLoading) return <div>Loading...</div>;
  if (!isAuthenticated) return <Navigate to="/login" replace />;
  if (!isAdmin) return <div>403 Forbidden</div>;
  
  return children;
}
```

---

## 7. Luồng Xử Lý Các Tính Năng Chính

### 7.1 Donation Creation (Complete Flow với File References)

**Files Involved:**
1. `FRONTEND/src/pages/DonatePage.jsx` (Dòng 1-767)
2. `FRONTEND/src/services/donationServices.js`
3. `Backend/Controllers/DonationController.cs` (Dòng 22-83)
4. `Backend/Services/DonationService.cs` (Dòng 24-135)
5. `Backend/Helpers/EmailTemplate.cs` (Dòng 64-135)
6. `Backend/Services/EmailService.cs`

**Step-by-Step với Line Numbers:**

**1. User Interaction (Frontend)**

**File**: `FRONTEND/src/pages/DonatePage.jsx`

**Dòng 150-250 (handleSubmit function):**
```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  // Client-side validation
  if (!formData.amount || parseFloat(formData.amount) <= 0) {
    toast.error('Amount must be greater than 0');
    return;
  }
  
  // Find program title from programId
  const selectedProgram = programs.find(p => p.id === parseInt(formData.programId));
  const causeName = selectedProgram ? selectedProgram.title : "General Donation";
  
  // Prepare payload
  const donationData = {
    amount: parseFloat(formData.amount),
    cause: causeName,
    programId: parseInt(formData.programId) || null,
    fullName: formData.fullName,
    email: formData.email,
    phone: formData.phone || null,
    address: formData.address || null,
    paymentMethod: formData.paymentMethod,
    anonymous: formData.anonymous,
    newsletter: formData.newsletter,
    userId: user?.id || null
  };
  
  setIsSubmitting(true);
  
  try {
    // Call API service
    const result = await donationService.create(donationData);
    
    // Success
    toast.success('Donation successful! Check your email for confirmation.');
    navigate('/donation-history');
  } catch (error) {
    const message = error.response?.data?.message || 'Donation failed';
    toast.error(message);
    console.error('Donation error:', error);
  } finally {
    setIsSubmitting(false);
  }
};
```

**2. API Call (Frontend Service)**

**File**: `FRONTEND/src/services/donationServices.js`

```javascript
import api from './api';

export const create = async (donationData) => {
  const response = await api.post('/donation', donationData);
  return response.data;
};
```

**3. Request Received (Backend Controller)**

**File**: `Backend/Controllers/DonationController.cs` **Dòng 22-83**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] DonationDTO? dto)
{
    // Dòng 26-29: Null check
    if (dto == null)
    {
        return BadRequest(new { message = "Request body is required" });
    }

    // Dòng 32: Log for debugging
    Console.WriteLine($"[Donation] Received: Amount={dto.Amount}, Cause={dto.Cause}, ...");

    // Dòng 34-44: ModelState validation
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value?.Errors.Count > 0)
            .Select(x => new { Field = x.Key, Errors = x.Value?.Errors.Select(e => e.ErrorMessage) })
            .ToList();
        
        Console.WriteLine($"[Donation] ModelState validation failed: ...");
        return BadRequest(new { message = "Validation failed", errors });
    }
    
    // Dòng 47-69: Manual validation
    if (dto.Amount <= 0)
        return BadRequest(new { message = "Amount must be greater than 0" });
    if (string.IsNullOrWhiteSpace(dto.Cause))
        return BadRequest(new { message = "Cause is required" });
    if (string.IsNullOrWhiteSpace(dto.FullName))
        return BadRequest(new { message = "Full name is required" });
    if (string.IsNullOrWhiteSpace(dto.Email))
        return BadRequest(new { message = "Email is required" });

    try
    {
        // Dòng 73: Delegate to service
        var donation = await _donationService.CreateAsync(dto);
        Console.WriteLine($"[Donation] Successfully created donation ID: {donation.Id}");
        return Ok(donation);
    }
    catch (Exception ex)
    {
        // Dòng 79-81: Error handling
        Console.WriteLine($"[Donation] Exception: {ex.Message}");
        return BadRequest(new { message = "Failed to create donation", error = ex.Message });
    }
}
```

**4. Business Logic (Service Layer)**

**File**: `Backend/Services/DonationService.cs` **Dòng 24-135**

```csharp
public async Task<Donation> CreateAsync(DonationDTO dto)
{
    // Dòng 27-50: Business validation
    if (string.IsNullOrWhiteSpace(dto.Email))
        throw new ArgumentException("Email is required");
    if (string.IsNullOrWhiteSpace(dto.FullName))
        throw new ArgumentException("Full name is required");
    if (string.IsNullOrWhiteSpace(dto.Cause))
        throw new ArgumentException("Cause is required");

    // Dòng 43: Prepare donor name (anonymous check)
    var donorName = dto.Anonymous ? "Anonymous" : dto.FullName.Trim();
    var donorEmail = dto.Email.Trim();
    
    // Dòng 52-67: Create donation entity
    var donation = new Donation
    {
        Amount = dto.Amount,
        CauseName = string.IsNullOrWhiteSpace(dto.Cause) ? "General Donation" : dto.Cause.Trim(),
        PaymentStatus = "Success",  // Dummy payment luôn thành công
        PaymentMethod = string.IsNullOrWhiteSpace(dto.PaymentMethod) ? "Card" : dto.PaymentMethod.Trim(),
        UserId = dto.UserId,        // null nếu guest donate
        ProgramId = dto.ProgramId,  // Link to program if provided
        TransactionReference = "TRX-" + Guid.NewGuid().ToString("N").Substring(0, 12),
        DonorName = donorName,
        DonorEmail = donorEmail,
        DonorPhone = string.IsNullOrWhiteSpace(dto.Phone) ? null : dto.Phone.Trim(),
        DonorAddress = string.IsNullOrWhiteSpace(dto.Address) ? null : dto.Address.Trim(),
        IsAnonymous = dto.Anonymous,
        SubscribeNewsletter = dto.Newsletter
    };

    try
    {
        // Dòng 71-74: Save to database
        Console.WriteLine($"[DonationService] Creating donation: ...");
        _context.Donations.Add(donation);
        await _context.SaveChangesAsync();
        Console.WriteLine($"[DonationService] Donation created successfully with ID: {donation.Id}");

        // Dòng 77-111: Send email confirmation (fire-and-forget)
        if (!string.IsNullOrWhiteSpace(donorEmail) && !dto.Anonymous)
        {
            _ = Task.Run(async () =>
            {
                try
                {
                    // Dòng 85-95: Generate email body
                    var emailBody = EmailTemplate.DonationReceiptTemplate(
                        donorName,
                        donation.Amount,
                        donation.CauseName,
                        donation.TransactionReference,
                        donation.CreatedAt
                    );
                    
                    // Dòng 96-103: Send email
                    await _emailService.SendEmailAsync(
                        donorEmail,
                        "Thank you for your donation",
                        emailBody
                    );
                    Console.WriteLine($"[DonationService] Confirmation email sent to {donorEmail}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"[DonationService] Failed to send email: {ex.Message}");
                    // Fail silently - không block donation success
                }
            });
        }
        
        return donation;
    }
    catch (DbUpdateException dbEx)
    {
        // Dòng 119-128: Database error handling
        Console.WriteLine($"[DonationService] Database error: {dbEx.InnerException?.Message}");
        throw new Exception($"Database error while creating donation: {dbEx.InnerException?.Message}", dbEx);
    }
}
```

**5. Database Save (Entity Framework)**

**EF Core translates to SQL:**
```sql
INSERT INTO Donations 
  (Amount, CauseName, PaymentStatus, PaymentMethod, UserId, ProgramId, 
   TransactionReference, DonorName, DonorEmail, DonorPhone, DonorAddress, 
   IsAnonymous, SubscribeNewsletter, CreatedAt)
VALUES 
  (100.00, 'Education for Children', 'Success', 'Credit Card', 10, 5, 
   'TRX-a3f9b2c1d4e5', 'John Doe', 'john@example.com', '+1234567890', '123 Main St', 
   0, 1, GETUTCDATE())
```

**6. Email Sent (Background Task)**

**File**: `Backend/Services/EmailService.cs`

```csharp
public async Task<bool> SendEmailAsync(string to, string subject, string body)
{
    using var smtpClient = new SmtpClient(_smtpHost, _smtpPort)
    {
        EnableSsl = true,
        Credentials = new NetworkCredential(_smtpUser, _smtpPass)
    };
    
    var mailMessage = new MailMessage
    {
        From = new MailAddress(_smtpFrom),
        Subject = subject,
        Body = body,
        IsBodyHtml = true
    };
    mailMessage.To.Add(to);
    
    await smtpClient.SendMailAsync(mailMessage);
    return true;
}
```

**7. Response Returned & UI Update**

**Frontend receives response:**
```json
{
  "id": 42,
  "amount": 100.00,
  "causeName": "Education for Children",
  "paymentStatus": "Success",
  "transactionReference": "TRX-a3f9b2c1d4e5",
  "donorName": "John Doe",
  "donorEmail": "john@example.com",
  "createdAt": "2024-11-07T10:30:00Z"
}
```

**Frontend shows success toast và navigate:**
```jsx
toast.success('Donation successful! Check your email for confirmation.');
navigate('/donation-history');
```

### 7.2 Program Registration Flow (Prevent Duplicates)

**Files:**
1. `FRONTEND/src/pages/ProgramsPage.jsx`
2. `FRONTEND/src/services/programServices.js`
3. `Backend/Controllers/ProgramController.cs`
4. `Backend/Services/ProgramService.cs` **Dòng 69-105**

**Flow:**

```
User clicks "Register Interest" → Modal opens
  ↓
User fills form: { fullName, email, phone, message }
  ↓
POST /api/program/{programId}/register
  Body: { fullName, email, phone, message }
  ↓
ProgramController.Register(programId, request) (Dòng 98-115)
  ↓
ProgramService.RegisterAsync(programId, request) (Dòng 69-105)
  │
  ├─ Dòng 71-73: Check program exists
  │   └─> SELECT * FROM NgoPrograms WHERE Id = programId
  │
  ├─ Dòng 76-79: Check duplicate registration
  │   └─> SELECT * FROM ProgramRegistrations
  │       WHERE ProgramId = programId 
  │       AND Email.ToLower() = request.Email.ToLower()
  │
  ├─ Dòng 80-82: If duplicate found
  │   └─> return (false, "You have already registered interest in this program")
  │
  ├─ Dòng 85-95: Create new registration
  │   └─> var registration = new ProgramRegistration {
  │         ProgramId = programId,
  │         UserId = request.UserId,  // null if guest
  │         Email = request.Email,
  │         FullName = request.FullName,
  │         Phone = request.Phone,
  │         Message = request.Message
  │       };
  │
  ├─ Dòng 97-98: Save to database
  │   └─> INSERT INTO ProgramRegistrations (...)
  │
  └─ Dòng 100: Return success
      └─> return (true, "Successfully registered interest")
  ↓
Frontend shows toast: "Successfully registered!"
```

**Duplicate Prevention Logic:**
```csharp
// ProgramService.cs Dòng 76-82
var existingRegistration = await _context.ProgramRegistrations
    .FirstOrDefaultAsync(r => 
        r.ProgramId == programId && 
        r.Email != null && 
        r.Email.ToLower() == req.Email.ToLower());

if (existingRegistration != null)
{
    return (false, "You have already registered interest in this program");
}
```

### 7.3 Get Program Statistics Flow

**Files:**
1. `FRONTEND/src/pages/ProgramsPage.jsx` (fetch stats)
2. `FRONTEND/src/services/programServices.js` (getStats API call)
3. `Backend/Controllers/ProgramController.cs` (Dòng 39-61)
4. `Backend/Services/ProgramService.cs` (GetStatsAsync method)

**Flow:**

```
Frontend: ProgramsPage.jsx useEffect()
  ↓
programService.getStats(programId)
  ↓
GET /api/program/{programId}/stats
  ↓
ProgramController.GetProgramStats(programId) (Dòng 39-61)
  ↓
ProgramService.GetStatsAsync(programId)
  │
  ├─ Calculate total donations
  │   └─> SELECT SUM(Amount) FROM Donations WHERE ProgramId = programId AND PaymentStatus = 'Success'
  │
  ├─ Get goal amount
  │   └─> SELECT GoalAmount FROM NgoPrograms WHERE Id = programId
  │
  ├─ Calculate progress percentage
  │   └─> progressPercentage = (totalDonations / goalAmount) * 100
  │
  ├─ Count registrations
  │   └─> SELECT COUNT(*) FROM ProgramRegistrations WHERE ProgramId = programId
  │
  └─ Return stats object
      {
        programId: 5,
        totalDonations: 5000.00,
        goalAmount: 10000.00,
        progressPercentage: 50.0,
        registrationCount: 42
      }
  ↓
Frontend: Update programStats state
  ↓
Render progress bar: width = {progressPercentage}%
```

---

## 8. Email System

### 8.1 Email Service Configuration

**File**: `Backend/appsettings.json` hoặc `appsettings.Development.json`

```json
{
  "Smtp": {
    "Host": "smtp.gmail.com",
    "Port": "587",
    "User": "your-email@gmail.com",
    "Pass": "your-app-password",
    "From": "your-email@gmail.com"
  }
}
```

**For Gmail:**
1. Enable 2-Factor Authentication on Google account
2. Generate App Password: [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
3. Use 16-character App Password (không phải regular password)

**For Outlook:**
```json
{
  "Smtp": {
    "Host": "smtp-mail.outlook.com",
    "Port": "587",
    "User": "your-email@outlook.com",
    "Pass": "your-password",
    "From": "your-email@outlook.com"
  }
}
```

### 8.2 Email Templates

**File**: `Backend/Helpers/EmailTemplate.cs` **Dòng 1-302**

**Available Templates:**

1. **DonationReceiptTemplate** (Dòng 64-135)
   - Purpose: Donation confirmation email
   - Parameters: `donorName`, `amount`, `cause`, `transactionRef`, `donationDate`
   - HTML template với styling đẹp

2. **VerificationEmailTemplate** (Dòng 140-188)
   - Purpose: Email verification
   - Parameters: `fullName`, `verificationLink`, `verificationToken`
   - Link: `http://localhost:5173/verify-email?token=xxx`

3. **PasswordResetEmailTemplate** (Dòng 193-244)
   - Purpose: Password reset
   - Parameters: `fullName`, `resetLink`, `resetToken`
   - Link: `http://localhost:5173/reset-password?token=xxx`

4. **ContactConfirmationEmailTemplate** (Dòng 249-300)
   - Purpose: Contact form confirmation
   - Parameters: `fullName`, `subject`, `queryId`

5. **QueryReplyTemplate** (Dòng 40-59)
   - Purpose: Admin replies to user query
   - Parameters: `subject`, `reply`, `recipientName`

6. **InvitationTemplate** (Dòng 11-35)
   - Purpose: User invites friend
   - Parameters: `fromName`, `message`, `inviteToken`

**Example: Donation Receipt Email**

**File**: `Backend/Helpers/EmailTemplate.cs` **Dòng 64-135**

```csharp
public static string DonationReceiptTemplate(
    string? donorName, 
    decimal amount, 
    string? cause, 
    string? transactionRef, 
    DateTime? donationDate)
{
    var sb = new StringBuilder();
    sb.AppendLine("<!DOCTYPE html>");
    sb.AppendLine("<html>");
    sb.AppendLine("<head>");
    sb.AppendLine("    <meta charset='UTF-8'>");
    sb.AppendLine("    <style>");
    // ... CSS styling
    sb.AppendLine("    </style>");
    sb.AppendLine("</head>");
    sb.AppendLine("<body>");
    sb.AppendLine("    <div class='container'>");
    sb.AppendLine("        <div class='header'>");
    sb.AppendLine("            <h1>❤️ Thank You for Your Donation!</h1>");
    sb.AppendLine("        </div>");
    sb.AppendLine("        <div class='content'>");
    sb.AppendLine($"            <p>Dear <strong>{donorName}</strong>,</p>");
    sb.AppendLine("            <div class='thank-you'>Thank you for your generous contribution!</div>");
    sb.AppendLine($"            <div class='amount'>{amount:C}</div>");
    sb.AppendLine($"            <p>Cause: {cause}</p>");
    sb.AppendLine($"            <p>Transaction Reference: {transactionRef}</p>");
    // ... more HTML
    sb.AppendLine("    </div>");
    sb.AppendLine("</body>");
    sb.AppendLine("</html>");
    
    return sb.ToString();
}
```

### 8.3 Email Sending Strategy (Fire-and-Forget)

**Pattern được sử dụng:**
```csharp
// Don't await - returns response immediately
_ = Task.Run(async () =>
{
    try
    {
        await _emailService.SendEmailAsync(...);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Email sending failed: {ex.Message}");
        // Fail silently - don't block user flow
    }
});
```

**Benefits:**
- Response time: ~100-200ms (thay vì ~1-2s nếu await email)
- User không phải đợi email gửi xong
- Email failures không block donation success

**Trade-offs:**
- Email có thể fail mà user không biết (nhưng donation vẫn thành công)
- Không có retry mechanism (có thể implement sau)

---

## 9. Hướng Dẫn Debugging

### 9.1 Debugging Failed Donation

**Triệu chứng**: User click "Donate Now" → Error "Donation failed"

**Step 1: Check Frontend Console (F12)**

Mở browser DevTools (F12) → Console tab, tìm errors:

```javascript
// DonatePage.jsx Dòng 1928-1932
catch (error) {
  const message = error.response?.data?.message || 'Donation failed';
  toast.error(message);
  console.error('Donation error:', error);  // ← Check log này
}
```

**Check request payload:**
```javascript
// Thêm log trước khi submit
console.log('Donation data:', donationData);
// Expected: { amount: 100, cause: "Education", fullName: "John", ... }
```

**Step 2: Check Backend Logs (Terminal)**

Trong terminal chạy backend, tìm logs:

```
[Donation] Received: Amount=100, Cause=Education, FullName=John, Email=john@example.com
[Donation] ModelState validation failed: Amount: must be greater than 0
OR
[DonationService] Creating donation: Amount=100, CauseName=Education, ...
[DonationService] Donation created successfully with ID: 42
OR
[DonationService] Database error: Cannot insert NULL into column 'DonorEmail'
```

**Step 3: Check Database (SSMS)**

Mở SQL Server Management Studio:

```sql
-- Check if donation was created
SELECT TOP 10 * FROM Donations ORDER BY CreatedAt DESC;

-- Check last inserted ID
SELECT IDENT_CURRENT('Donations');

-- Check constraints
EXEC sp_helpconstraint 'Donations';

-- Check if program exists
SELECT * FROM NgoPrograms WHERE Id = 5;
```

**Step 4: Common Issues & Solutions**

| Error Message | Cause | Solution |
|---------------|-------|----------|
| "Amount must be greater than 0" | Client validation failed | Check `formData.amount` value trong console |
| "Cause is required" | Empty cause field | Ensure program selected hoặc có default "General Donation" |
| "Database error: Cannot insert NULL" | Missing required field | Check DTO mapping, ensure DonorEmail không null |
| "Email sending failed" | SMTP config wrong | Check `appsettings.Development.json` SMTP settings |
| "Program not found" | Invalid programId | Check programId exists trong database |

### 9.2 Debugging Login Issues

**Issue**: User không thể login

**Check 1: User exists?**

```sql
-- Open SSMS
SELECT * FROM Users WHERE Email = 'user@example.com' OR Username = 'username';
```

**Check 2: Email verified?**

```sql
-- Nếu EmailVerified = 0, user không thể login
SELECT Email, EmailVerified, EmailVerificationToken FROM Users WHERE Email = 'user@example.com';

-- Temporary fix (set verified manually)
UPDATE Users SET EmailVerified = 1 WHERE Email = 'user@example.com';
```

**Check 3: Password correct?**

```csharp
// Trong AuthService.cs Dòng 306, thêm debug log
var valid = PasswordHasher.Verify(req.Password, user.PasswordHash);
Console.WriteLine($"[AuthService] Password valid: {valid}");  // Debug
```

**Check 4: JWT token valid?**

```javascript
// Frontend console
const token = localStorage.getItem('token');
console.log('Token:', token);

// Decode token tại jwt.io
// Check claims: id, email, role, exp
```

**Common Login Errors:**

| Error | Cause | Solution |
|-------|-------|----------|
| "Invalid username/email or password" | Wrong password hoặc user không tồn tại | Check password, check user exists |
| "Please verify your email" | Email chưa verified | Set EmailVerified = 1 trong DB hoặc verify email |
| "Token expired" | JWT token hết hạn | Login lại để get new token |

### 9.3 Debugging Email Not Sending

**Check SMTP Config:**

```json
// appsettings.Development.json
{
  "Smtp": {
    "Host": "smtp.gmail.com",      // Correct?
    "Port": "587",                 // 587 for TLS, 465 for SSL
    "User": "your-email@gmail.com",
    "Pass": "app-password-16chars",  // App Password, NOT regular password!
    "From": "your-email@gmail.com"
  }
}
```

**Check Backend Logs:**

```
[Info] Verification email sent successfully to user@example.com
OR
[Warning] Failed to send email: Authentication failed
OR
[Error] SMTP connection timeout
```

**Common SMTP Errors:**

| Error | Cause | Solution |
|-------|-------|----------|
| "Authentication failed" | Wrong password | Use Gmail App Password (not regular password) |
| "Connection refused" | Wrong port | Use 587 for TLS, 465 for SSL |
| "Timeout" | Network/firewall | Try different network, check firewall |

**Test SMTP manually:**

```csharp
// Tạo test endpoint trong TestController.cs
[HttpPost("send-email")]
public async Task<IActionResult> TestEmail()
{
    await _emailService.SendEmailAsync("test@example.com", "Test", "This is a test");
    return Ok("Email sent");
}
```

### 9.4 Debugging Database Connection Issues

**Issue**: "Cannot connect to server" hoặc "Login failed"

**Check 1: SQL Server đang chạy?**

- Mở Task Manager → Services → Tìm "SQL Server (MSSQLSERVER)" hoặc "SQL Server (SQLEXPRESS)"
- Nếu không chạy → Start service

**Check 2: Connection string đúng?**

```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=Give_AID_API_Db;..."
  }
}
```

**Check server name:**
- Mở SSMS → Connect → Server name hiển thị ở đây
- VD: `DESKTOP-ABC123\SQLEXPRESS` hoặc `localhost\SQLEXPRESS`
- Nếu có dấu `\` → dùng `\\` trong JSON

**Check 3: Database exists?**

```sql
-- Check database exists
SELECT name FROM sys.databases WHERE name = 'Give_AID_API_Db';

-- If not exists, create it
CREATE DATABASE Give_AID_API_Db;
```

**Check 4: User có quyền?**

```sql
-- Check user permissions
EXEC sp_helplogins 'your_username';
```

### 9.5 Breakpoint Recommendations

**Backend (Visual Studio / VS Code):**

1. **DonationController.Create** (Dòng 22)
   - Check DTO received
   - Check validation errors

2. **DonationService.CreateAsync** (Dòng 24)
   - Check entity creation
   - Check database save

3. **AuthService.LoginAsync** (Dòng 292)
   - Check user found
   - Check password verification
   - Check token generation

**Frontend (Chrome DevTools):**

1. **DonatePage.jsx handleSubmit** (Dòng 150)
   - Check formData values
   - Check API call payload

2. **donationServices.js create** function
   - Check request config
   - Check response data

3. **AuthContext.jsx login** function
   - Check token received
   - Check user decoded

---

## 10. Best Practices & Onboarding Checklist

### 10.1 Naming Conventions

**Backend (C#):**
- **Classes**: `PascalCase` (`DonationService`, `AuthController`)
- **Methods**: `PascalCase` (`CreateAsync`, `GetAllAsync`)
- **Variables**: `camelCase` (`userId`, `donationService`)
- **Private fields**: `_camelCase` (`_context`, `_donationService`)
- **Constants**: `PascalCase` (`DefaultConnection`)
- **Namespaces**: `PascalCase` (`Backend.Services`, `Backend.Controllers`)

**Frontend (JavaScript/React):**
- **Components**: `PascalCase` (`DonatePage`, `Navbar`)
- **Functions**: `camelCase` (`handleSubmit`, `fetchPrograms`)
- **Variables**: `camelCase` (`formData`, `isSubmitting`)
- **Constants**: `UPPER_SNAKE_CASE` (`API_BASE_URL`)
- **Hooks**: `camelCase` với "use" prefix (`useAuth`, `useState`)
- **Files**: `PascalCase` cho components (`DonatePage.jsx`), `camelCase` cho utilities (`helpers.js`)

### 10.2 Error Handling Pattern

**Backend (C#):**

```csharp
try
{
    // Business logic
    var result = await _service.DoSomething();
    return Ok(result);
}
catch (ArgumentException ex)
{
    // Client error (400)
    Console.WriteLine($"[Service] Validation error: {ex.Message}");
    return BadRequest(new { message = ex.Message });
}
catch (DbUpdateException dbEx)
{
    // Database error (500)
    Console.WriteLine($"[Service] Database error: {dbEx.InnerException?.Message}");
    return StatusCode(500, new { message = "Database error", error = dbEx.Message });
}
catch (Exception ex)
{
    // Unexpected error (500)
    Console.WriteLine($"[Service] Unexpected error: {ex.Message}");
    return StatusCode(500, new { message = "Internal server error" });
}
```

**Frontend (JavaScript):**

```javascript
try {
  const result = await api.post('/endpoint', data);
  toast.success('Success!');
  return result.data;
} catch (error) {
  const message = error.response?.data?.message || 'An error occurred';
  toast.error(message);
  console.error('API error:', error);  // Log for debugging
  throw error;
}
```

### 10.3 Where to Add New Features

**Scenario: Thêm tính năng "Event"**

**Backend:**

1. **Create Model**: `Backend/Models/Event.cs`
   ```csharp
   public class Event
   {
       public int Id { get; set; }
       public string Title { get; set; } = string.Empty;
       // ... other properties
   }
   ```

2. **Add DbSet**: `Backend/Data/GiveAidContext.cs`
   ```csharp
   public DbSet<Event> Events { get; set; }
   ```

3. **Create Migration:**
   ```bash
   cd Backend
   dotnet ef migrations add AddEventTable
   dotnet ef database update
   ```

4. **Create DTO**: `Backend/DTOs/EventDTO.cs`
   ```csharp
   public class EventDTO
   {
       public string Title { get; set; } = string.Empty;
       // ... other properties
   }
   ```

5. **Create Service**: `Backend/Services/EventService.cs`
   ```csharp
   public class EventService
   {
       private readonly GiveAidContext _context;
       // ... methods
   }
   ```

6. **Register Service**: `Backend/Program.cs` Dòng 29-41
   ```csharp
   builder.Services.AddScoped<EventService>();
   ```

7. **Create Controller**: `Backend/Controllers/EventController.cs`
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class EventController : ControllerBase
   {
       private readonly EventService _eventService;
       // ... endpoints
   }
   ```

**Frontend:**

1. **Create Service**: `FRONTEND/src/services/eventServices.js`
   ```javascript
   import api from './api';
   
   export const getAll = async () => {
     const response = await api.get('/event');
     return response.data;
   };
   ```

2. **Create Page**: `FRONTEND/src/pages/EventsPage.jsx`
   ```jsx
   export default function EventsPage() {
     // ... component code
   }
   ```

3. **Add Route**: `FRONTEND/src/App.jsx` Dòng 80-97
   ```jsx
   <Route path="/events" element={<EventsPage />} />
   ```

4. **Add Nav Link**: `FRONTEND/src/components/layout/Navbar.jsx`
   ```jsx
   <li className="nav-item">
     <Link className="nav-link" to="/events">Events</Link>
   </li>
   ```

### 10.4 Code Review Checklist

**Trước khi tạo Pull Request:**

- [ ] Code compiles without errors
- [ ] No console.log statements left (use Console.WriteLine in backend)
- [ ] Error handling implemented
- [ ] Validation added (client-side + server-side)
- [ ] Database migrations tested
- [ ] API endpoints tested (Swagger hoặc Postman)
- [ ] Frontend pages tested (manual testing)
- [ ] No hardcoded values (use config)
- [ ] Comments added for complex logic
- [ ] Follows naming conventions
- [ ] No sensitive data in code (passwords, API keys)

### 10.5 Onboarding Checklist cho Dev Mới

**Prerequisites:**
- [ ] Install Git
- [ ] Install .NET 8.0 SDK
- [ ] Install SQL Server + SSMS
- [ ] Install Node.js 18+
- [ ] Install VS Code hoặc Visual Studio 2022

**Repository Setup:**
- [ ] Clone repository: `git clone <repo-url>`
- [ ] Checkout dev branch: `git checkout dev` (nếu có)
- [ ] Read README.md (user-facing setup)
- [ ] Read README_MEMBER.md (technical docs - file này)

**Backend Setup:**
- [ ] Navigate to `GIVE-AID/Backend`
- [ ] Run `dotnet restore`
- [ ] Copy `appsettings.example.json` → `appsettings.Development.json`
- [ ] Configure SQL Server connection string
- [ ] Configure SMTP settings (optional, for email features)
- [ ] Run migrations: `dotnet ef database update`
- [ ] Verify database created trong SSMS
- [ ] Run backend: `dotnet run`
- [ ] Test Swagger: `http://localhost:5230/swagger`
- [ ] Verify admin user seeded (check console logs)

**Frontend Setup:**
- [ ] Navigate to `GIVE-AID/FRONTEND`
- [ ] Run `npm install`
- [ ] Verify `package-lock.json` created
- [ ] Check `src/services/api.js` có correct base URL
- [ ] Run frontend: `npm run dev`
- [ ] Open browser: `http://localhost:5173`
- [ ] Test login với admin (admin@giveaid.org / Admin123!)
- [ ] Browse pages, check console for errors

**Code Familiarity:**
- [ ] Read `Backend/Program.cs` — Understand DI & middleware
- [ ] Read `FRONTEND/src/App.jsx` — Understand routes & providers
- [ ] Explore `AuthController` + `AuthService` — Understand auth flow
- [ ] Explore `DonationController` + `DonationService` — Understand donation flow
- [ ] Check database tables trong SSMS — Understand schema
- [ ] Read `AuthContext.jsx` — Understand global state
- [ ] Test API endpoints trong Swagger

**Testing:**
- [ ] Register new user via frontend
- [ ] Verify email (nếu SMTP configured, else manually set EmailVerified=1 trong DB)
- [ ] Login với new user
- [ ] Create a donation
- [ ] Check donation trong database
- [ ] Check donation history page
- [ ] Login as admin
- [ ] Access admin panel
- [ ] View users, donations, programs

**Development Workflow:**
- [ ] Create feature branch: `git checkout -b feature/your-feature`
- [ ] Make changes
- [ ] Test locally
- [ ] Run linter: `npm run lint` (frontend)
- [ ] Commit: `git commit -m "feat: your feature description"`
- [ ] Push: `git push origin feature/your-feature`
- [ ] Create Pull Request
- [ ] Request code review

**Debugging Setup:**
- [ ] Install browser DevTools extensions (React DevTools)
- [ ] Configure VS Code debugger cho .NET (launch.json)
- [ ] Enable verbose logging trong `appsettings.Development.json`
- [ ] Learn to use Swagger for API testing
- [ ] Set up Postman collection (optional)

### 10.6 Quick Reference

**Common Commands:**

**Backend:**
```bash
cd GIVE-AID/Backend
dotnet restore              # Restore NuGet packages
dotnet build               # Build project
dotnet run                 # Run backend (port 5230)
dotnet ef migrations add MigrationName   # Create migration
dotnet ef database update  # Apply migrations
dotnet ef database drop    # Drop database (careful!)
```

**Frontend:**
```bash
cd GIVE-AID/FRONTEND
npm install                # Install dependencies
npm run dev               # Run dev server (port 5173)
npm run build             # Build for production
npm run preview           # Preview production build
npm run lint              # Run ESLint
```

**Important URLs:**
- Frontend Dev: http://localhost:5173
- Backend API: http://localhost:5230
- Swagger Docs: http://localhost:5230/swagger
- Admin Panel: http://localhost:5173/admin/users

**Default Credentials:**
- Admin: admin@giveaid.org / Admin123!

**Key Environment Variables:**

**Backend (appsettings.Development.json):**
- `ConnectionStrings:DefaultConnection` — SQL Server connection
- `Jwt:Key` — JWT secret key
- `Smtp:*` — Email configuration

**Frontend:**
- Base URL hardcoded trong `src/services/api.js` (http://localhost:5230/api)

---

## Tổng Kết

Tài liệu này đã bao phủ:
- ✅ Tổng quan kiến trúc hệ thống
- ✅ Cấu trúc thư mục & file quan trọng
- ✅ Backend deep dive (Controllers, Services, Models)
- ✅ Frontend deep dive (Pages, Components, Services)
- ✅ Database schema & migrations
- ✅ Authentication & authorization flow
- ✅ Luồng xử lý các tính năng chính
- ✅ Email system
- ✅ Hướng dẫn debugging
- ✅ Best practices & onboarding checklist

**Lưu ý quan trọng:**
- File này sẽ được update thường xuyên khi có thay đổi
- Nếu có thắc mắc, hỏi team lead hoặc xem code comments
- Luôn test code trước khi commit
- Follow naming conventions và error handling patterns

---

**Last Updated**: Tháng 11/2024  
**Version**: 1.0.0  
**Maintained by**: Give-AID Development Team
