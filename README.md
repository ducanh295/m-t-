# DETAILED SYSTEM FUNCTIONS DESCRIPTION

This document describes in detail the functions of the GIVE-AID system following the Input-Process-Output structure.

---

## **COMMON FUNCTIONS**

### **Function 1: Register Account (Register)**

| | |
|---|---|
| **Input** | • User enters information: Full name, Username, Email, Password, Phone (optional), Address (optional) |
| **Process** | • System checks if input data is valid<br>• System checks if Email and Username already exist in the database<br>• If already exists, system displays error message to user<br>• If not exists, system encrypts password and creates email verification token<br>• System creates new account with unverified email status<br>• System saves user information to database<br>• System sends verification email to user (sent asynchronously, does not block response)<br>• System returns login token and success message |
| **Output** | • Result is a success message or error message for the user |

---

### **Function 2: Login**

| | |
|---|---|
| **Input** | • User enters Username or Email and Password |
| **Process** | • System checks if input data is valid<br>• System determines if user entered Email or Username<br>• System searches for user in database by Email or Username<br>• If not found, system displays login error<br>• If found, system verifies password<br>• If password is incorrect, system displays login error<br>• If password is correct, system checks if email has been verified<br>• If email is not verified, system requires user to verify email first<br>• If email is verified, system creates login token and returns it to user |
| **Output** | • Result is login token and success message, or error message for the user |

---

### **Function 3: Verify Email**

| | |
|---|---|
| **Input** | • User enters verification token from link in email |
| **Process** | • System checks if token is valid<br>• System searches for user with matching verification token that has not expired<br>• If not found or token expired, system displays error<br>• If found, system marks email as verified<br>• System removes verification token and token expiry<br>• System saves changes to database<br>• System displays verification success message |
| **Output** | • Result is verification success message or error message for the user |

---

### **Function 4: Forgot Password**

| | |
|---|---|
| **Input** | • User enters Email |
| **Process** | • System checks if Email is valid<br>• System searches for user in database by Email<br>• If found, system creates password reset token and sets expiry time<br>• System saves token to database<br>• System sends email containing password reset link to user (sent asynchronously)<br>• System always returns success message (security, does not reveal if email exists or not) |
| **Output** | • Result is success message for the user (if email exists, password reset link has been sent) |

---

### **Function 5: Reset Password**

| | |
|---|---|
| **Input** | • User enters password reset token from link in email and new password |
| **Process** | • System checks if token and new password are valid<br>• System searches for user with matching password reset token that has not expired<br>• If not found or token expired, system displays error<br>• If found, system encrypts new password<br>• System updates new password to database<br>• System removes password reset token and token expiry<br>• System displays password reset success message |
| **Output** | • Result is password reset success message or error message for the user |

---

### **Function 6: Make Donation (Donate)**

| | |
|---|---|
| **Input** | • User enters donation information: Amount, Cause, Full name, Email, Phone (optional), Address (optional), Payment method (optional), Select program (optional), Select anonymous (optional), Subscribe newsletter (optional) |
| **Process** | • System checks if input data is valid<br>• System checks amount is greater than 0, cause is not empty, full name is not empty, email is not empty<br>• System creates unique transaction reference for donation<br>• If user selects anonymous, system sets donor name as "Anonymous"<br>• System creates donation record with successful payment status<br>• System saves donation information to database<br>• If not anonymous and has email, system sends donation confirmation email to user (sent asynchronously)<br>• System returns created donation information |
| **Output** | • Result is created donation information or error message for the user |

---

### **Function 7: View Programs List (View Programs)**

| | |
|---|---|
| **Input** | • User accesses programs list page |
| **Process** | • System queries database to get all programs<br>• System sorts programs list<br>• System displays programs list to user |
| **Output** | • Result is list of programs displayed to user |

---

### **Function 8: View Program Statistics (View Program Stats)**

| | |
|---|---|
| **Input** | • User selects program to view statistics |
| **Process** | • System retrieves program information from database<br>• If not found, system displays error<br>• If found, system calculates total donation amount for program<br>• System calculates goal completion percentage<br>• System counts number of program registrations<br>• System calculates remaining amount needed (if goal exists)<br>• System displays statistics to user |
| **Output** | • Result is program statistics information displayed to user, or error message if program not found |

---

### **Function 9: Register for Program (Register for Program)**

| | |
|---|---|
| **Input** | • User enters information: Full name, Email, Phone (optional) and selects program to register |
| **Process** | • System checks if input data is valid<br>• System checks if program exists in database<br>• If not exists, system displays error<br>• If exists, system checks if user has already registered for this program (if logged in)<br>• If already registered, system displays error<br>• If not registered, system creates new registration record<br>• System saves registration information to database<br>• System displays registration success message |
| **Output** | • Result is registration success message or error message for the user |

---

### **Function 10: Submit Query/Request (Submit Query)**

| | |
|---|---|
| **Input** | • User enters information: Subject, Query content, Email, Full name (optional) |
| **Process** | • System checks if input data is valid<br>• System creates new query record<br>• System saves query information to database<br>• System returns created query information |
| **Output** | • Result is query information successfully submitted |

---

### **Function 11: View Profile** - Authenticated User Only

| | |
|---|---|
| **Input** | • Logged-in user accesses profile page |
| **Process** | • System checks if login token is valid<br>• System retrieves user information from token<br>• If token is invalid, system denies access<br>• System queries database to get user's profile information<br>• If no profile exists, system returns empty information<br>• If profile exists, system displays profile information to user |
| **Output** | • Result is user profile information displayed to user, or error message if no access permission |

---

### **Function 12: Update Profile** - Authenticated User Only

| | |
|---|---|
| **Input** | • Logged-in user enters information to update: Full name, Phone, Address, Date of birth, Gender (all optional) |
| **Process** | • System checks if login token is valid<br>• System checks if input data is valid<br>• System retrieves user information from token<br>• System searches for user's profile in database<br>• If no profile exists, system creates new profile<br>• If profile exists, system updates information fields from user input data<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is update success message or error message for the user |

---

### **Function 13: Change Password** - Authenticated User Only

| | |
|---|---|
| **Input** | • Logged-in user enters old password and new password |
| **Process** | • System checks if login token is valid<br>• System checks if input data is valid<br>• System retrieves user information from token<br>• System searches for user in database<br>• System verifies if old password is correct<br>• If old password is incorrect, system displays error<br>• If old password is correct, system encrypts new password<br>• System updates new password to database<br>• System displays password change success message |
| **Output** | • Result is password change success message or error message for the user |

---

### **Function 14: View Donation History** - Authenticated User Only

| | |
|---|---|
| **Input** | • Logged-in user accesses donation history page |
| **Process** | • System checks if login token is valid<br>• System retrieves user information from token<br>• System queries database to get all user's donations<br>• System sorts donations list by time (newest first)<br>• System displays donations list to user |
| **Output** | • Result is list of user's donations displayed on page |

---

## **ADMIN FUNCTIONS**

### **Function 1: User Management - View List (Get All Users)**

| | |
|---|---|
| **Input** | • Administrator accesses user management page |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• If no permission, system denies access<br>• System queries database to get all users<br>• System displays users list to administrator (does not display passwords) |
| **Output** | • Result is list of all users displayed to administrator, or error message if no permission |

---

### **Function 2: User Management - View Details (Get User by ID)**

| | |
|---|---|
| **Input** | • Administrator selects user to view details |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves user ID<br>• System queries database to search for user by ID<br>• If not found, system displays error<br>• If found, system displays user details to administrator |
| **Output** | • Result is user details displayed to administrator, or error message if not found or no permission |

---

### **Function 3: User Management - Update Role (Update User Role)**

| | |
|---|---|
| **Input** | • Administrator selects user and chooses new role (User or Admin) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if role is valid<br>• System retrieves user ID and new role<br>• System searches for user in database<br>• If not found, system displays error<br>• If found, system updates new role<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is role update success message or error message for administrator |

---

### **Function 4: User Management - Delete User (Delete User)**

| | |
|---|---|
| **Input** | • Administrator selects user to delete and confirms deletion |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves user ID<br>• System searches for user in database<br>• If not found, system displays error<br>• If found, system deletes user from database (may delete related records such as donations, queries)<br>• System displays deletion success message |
| **Output** | • Result is deletion success message or error message for administrator |

---

### **Function 5: Donation Management - View All (Get All Donations)**

| | |
|---|---|
| **Input** | • Administrator accesses donation management page |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System queries database to get all donations (including user information if available)<br>• System sorts donations list by time (newest first)<br>• System displays donations list to administrator |
| **Output** | • Result is list of all donations displayed to administrator, or error message if no permission |

---

### **Function 6: Donation Management - View Details (Get Donation by ID)**

| | |
|---|---|
| **Input** | • Administrator selects donation to view details |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves donation ID<br>• System queries database to search for donation by ID (including user information)<br>• If not found, system displays error<br>• If found, system displays donation details to administrator |
| **Output** | • Result is donation details displayed to administrator, or error message if not found or no permission |

---

### **Function 7: Program Management - Create New (Create Program)**

| | |
|---|---|
| **Input** | • Administrator enters program information: Title, Description, Start date (optional), End date (optional), Location (optional), Goal amount (optional), Select NGO organization (optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System creates new program<br>• System saves program information to database<br>• System returns created program information |
| **Output** | • Result is created program information or error message for administrator |

---

### **Function 8: Program Management - Update (Update Program)**

| | |
|---|---|
| **Input** | • Administrator selects program and enters information to update: Title, Description, Start date, End date, Location, Goal amount, NGO organization (all optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System retrieves program ID<br>• System searches for program in database<br>• If not found, system displays error<br>• If found, system updates information fields from administrator input data<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is update success message or error message for administrator |

---

### **Function 9: Program Management - Delete (Delete Program)**

| | |
|---|---|
| **Input** | • Administrator selects program to delete and confirms deletion |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves program ID<br>• System searches for program in database<br>• If not found, system displays error<br>• If found, system deletes program from database (may delete related records such as registrations, donations, gallery images)<br>• System displays deletion success message |
| **Output** | • Result is deletion success message or error message for administrator |

---

### **Function 10: NGO Management - Create New (Create NGO)**

| | |
|---|---|
| **Input** | • Administrator enters NGO organization information: Organization name, Description (optional), Logo URL (optional), Website (optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System creates new NGO organization<br>• System saves NGO organization information to database<br>• System returns created NGO organization information |
| **Output** | • Result is created NGO organization information or error message for administrator |

---

### **Function 11: NGO Management - Update (Update NGO)**

| | |
|---|---|
| **Input** | • Administrator selects NGO organization and enters information to update: Organization name, Description, Logo URL, Website (all optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System retrieves NGO organization ID<br>• System searches for NGO organization in database<br>• If not found, system displays error<br>• If found, system updates information fields from administrator input data<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is update success message or error message for administrator |

---

### **Function 12: NGO Management - Delete (Delete NGO)**

| | |
|---|---|
| **Input** | • Administrator selects NGO organization to delete and confirms deletion |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves NGO organization ID<br>• System searches for NGO organization in database<br>• System checks if NGO organization is being used by any program<br>• If being used, system displays error and does not allow deletion<br>• If not being used, system deletes NGO organization from database<br>• System displays deletion success message |
| **Output** | • Result is deletion success message or error message for administrator |

---

### **Function 13: Gallery Management - Add Image (Create Gallery Item)**

| | |
|---|---|
| **Input** | • Administrator selects image file or enters image URL, enters caption (optional), selects related program (optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if image file or image URL is provided (at least one of them)<br>• If image file is provided, system checks file type (only images allowed), checks file size (maximum 5MB)<br>• If file is valid, system saves file to folder and creates image URL<br>• If image URL is provided, system uses URL directly<br>• System creates new image record<br>• System saves image information to database<br>• System returns created image information |
| **Output** | • Result is created image information or error message for administrator (file type error, file size error, or no permission) |

---

### **Function 14: Gallery Management - Delete Image (Delete Gallery Item)**

| | |
|---|---|
| **Input** | • Administrator selects image to delete and confirms deletion |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves image ID<br>• System searches for image in database<br>• If not found, system displays error<br>• If found, system deletes image file from folder (if it is an uploaded file)<br>• System deletes image record from database<br>• System displays deletion success message |
| **Output** | • Result is deletion success message or error message for administrator |

---

### **Function 15: Partner Management - Create New (Create Partner)**

| | |
|---|---|
| **Input** | • Administrator enters partner information: Partner name, Logo URL (optional), Website (optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System creates new partner<br>• System saves partner information to database<br>• System returns created partner information |
| **Output** | • Result is created partner information or error message for administrator |

---

### **Function 16: Partner Management - Update (Update Partner)**

| | |
|---|---|
| **Input** | • Administrator selects partner and enters information to update: Partner name, Logo URL, Website (all optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System retrieves partner ID<br>• System searches for partner in database<br>• If not found, system displays error<br>• If found, system updates information fields from administrator input data<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is update success message or error message for administrator |

---

### **Function 17: Partner Management - Delete (Delete Partner)**

| | |
|---|---|
| **Input** | • Administrator selects partner to delete and confirms deletion |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System retrieves partner ID<br>• System searches for partner in database<br>• If not found, system displays error<br>• If found, system deletes partner from database<br>• System displays deletion success message |
| **Output** | • Result is deletion success message or error message for administrator |

---

### **Function 18: About Content Management - Create New (Create About Section)**

| | |
|---|---|
| **Input** | • Administrator enters content section information: Key (unique), Title, Content, Additional data (optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System creates new content section<br>• System saves content section information to database<br>• System returns created content section information |
| **Output** | • Result is created content section information or error message for administrator |

---

### **Function 19: About Content Management - Update (Update About Section)**

| | |
|---|---|
| **Input** | • Administrator selects content section and enters information to update: Title, Content, Additional data (all optional) |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if input data is valid<br>• System retrieves content section ID<br>• System searches for content section in database<br>• If not found, system displays error<br>• If found, system updates information fields from administrator input data<br>• System saves changes to database<br>• System displays update success message |
| **Output** | • Result is update success message or error message for administrator |

---

### **Function 20: Query Management - View All (Get All Queries)**

| | |
|---|---|
| **Input** | • Administrator accesses query management page |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System queries database to get all queries (including user information if available)<br>• System sorts queries list by time (newest first)<br>• System displays queries list to administrator |
| **Output** | • Result is list of all queries displayed to administrator, or error message if no permission |

---

### **Function 21: Query Management - Reply (Reply to Query)**

| | |
|---|---|
| **Input** | • Administrator selects query and enters reply content |
| **Process** | • System checks if administrator is logged in and has Admin role<br>• System checks if reply content is valid<br>• System retrieves query ID<br>• System searches for query in database<br>• If not found, system displays error<br>• If found, system updates reply content and reply date<br>• System saves changes to database<br>• If query has user and user has email, system sends reply email to user (sent asynchronously)<br>• System displays reply success message |
| **Output** | • Result is reply success message or error message for administrator |

---

## **Notes**

- All functions requiring login use login token in header
- Admin functions require user to have Admin role
- Emails are sent asynchronously to avoid slowing down system response
- All passwords are encrypted before saving to database
- Email verification tokens and password reset tokens have expiration time
