# MFA-Implementation
AI-Powered MFA Implementation

Demo : https://www.awesomescreenshot.com/video/38359043?key=c66557ec74c61966a526e0968c081070

Username : Aman
Password : Test@1234


### **Multi-Factor Authentication (MFA) Documentation**  
**Methods:** SMS-based OTP and TOTP (Authenticator App)  

---

## **1. Overview**
Multi-Factor Authentication (MFA) enhances security by requiring additional verification beyond passwords. The two MFA methods implemented in this project are:  
1. **SMS-based OTP:** Users receive a one-time passcode via SMS after login, which they must enter to authenticate.  
2. **TOTP (Time-based One-Time Password) Authenticator App:** Users scan a QR code using an authenticator app (Google Authenticator, Microsoft Authenticator) and enter the generated TOTP code.

---

## **2. Development Steps**
### **Step 1: Define MFA Methods and Database Schema**
- Create a database schema to store user MFA preferences.
- Modify the `Users` table to store:
  - `MFAMethod` (values: `SMS`, `AuthenticatorApp`)
  - `MFASecret` (for storing TOTP secret keys)
  - `OTP` (temporary SMS OTP)
  - `ExpiryTime` (for OTP expiration)

#### **Database Schema**
```sql
CREATE TABLE Users (
    UserId INT PRIMARY KEY IDENTITY(1,1),
    Username NVARCHAR(50) NOT NULL,
    PasswordHash NVARCHAR(255) NOT NULL,
    MFAEnabled BIT NOT NULL DEFAULT 0,
    MFAMethod NVARCHAR(50), -- Example: "SMS" or "AuthenticatorApp"
    MFASecret NVARCHAR(255), -- For storing TOTP secret key
    OTP NVARCHAR(6), -- Stores temporary OTP (if using SMS)
    ExpiryTime DATETIME -- Expiry time for OTP
);
```

### **Step 2: Implement Login API**
- Validate user credentials.
- Check `MFAMethod` and return relevant authentication steps.
- If MFA is `SMS`, generate OTP and store it in the database.
- If MFA is `AuthenticatorApp`, generate a TOTP secret and send a QR code.

### **Step 3: Send OTP for SMS-based MFA**
- Generate a **6-digit OTP** and set an **expiry time (5 minutes)**.
- Store OTP in the database and send it via Twilio SMS API.

### **Step 4: Verify OTP**
- Retrieve OTP from the database.
- Validate against user input and check expiry time.

### **Step 5: Setup Authenticator App (TOTP)**
- Generate a **TOTP secret** for the user.
- Display a **QR code** with the `otpauth://` URL format for scanning.

### **Step 6: Verify TOTP Code**
- Retrieve `MFASecret` from the database.
- Verify TOTP against the current time window using the `OtpNet` library.

### **Step 7: Secure Authentication Workflow**
- Store secrets securely (encrypt sensitive fields).
- Implement **rate limiting** to prevent brute-force attacks.
- Log MFA verification attempts in the database.

---

## **3. AI Prompts for Development**
During the development process, the following AI prompts were useful to generate APIs, security logic, and integrations:

### **Prompt to Generate Login API**
```
Generate a login API that supports MFA. If MFA method is "SMS", send OTP and verify via a separate endpoint. If MFA method is "AuthenticatorApp", verify using TOTP after login.
```

### **Prompt to Generate Database Schema**
```
Create SQL tables to store user MFA preferences, including fields for MFA method, status, backup codes, and OTP expiration time.
```

### **Prompt to Encrypt and Secure Data**
```
Write code to securely encrypt/decrypt MFA preferences and backup codes before storing them in the database.
```

### **Prompt to Integrate Twilio for OTPs**
```
Integrate Twilio into a .NET API to send OTPs to users via SMS. Generate a new OTP on login and verify user input.
```

### **Prompt to Generate TOTP QR Code**
```
Generate a QR code for TOTP authentication in .NET using the `otpauth://totp/{Issuer}:{AccountName}?secret={Secret}&issuer={Issuer}` format.
```

### **Prompt to Verify TOTP**
```
Use a TOTP library to generate and verify codes for authenticator apps after login.
```

---

## **4. MFA Login Flow**
### **Step 1: User Logs In**
```http
POST /api/auth/login
```
#### **Request Body**
```json
{
  "username": "john_doe",
  "password": "password123"
}
```
#### **Response Example**
```json
{
  "userId": 123,
  "mfaMethod": "SMS",
  "message": "OTP sent via SMS. Please verify."
}
```

### **Step 2A: SMS OTP Verification**
```http
POST /api/auth/verify-otp
```
#### **Request Body**
```json
{
  "userId": 123,
  "otp": "123456"
}
```

### **Step 2B: Authenticator App (TOTP) Verification**
```http
POST /api/auth/verify-totp
```
#### **Request Body**
```json
{
  "userId": 123,
  "totpCode": "123456"
}
```

---

## **5. Security Considerations**
✔ **Secure Storage:** Encrypt stored MFA secrets using AES encryption.  
✔ **Rate Limiting:** Apply request rate limits for MFA verification attempts.  
✔ **Logging & Auditing:** Track MFA attempts for security monitoring.  
✔ **Session Management:** Ensure JWT tokens expire correctly for user sessions.  

---

## **6. Summary**
✔️ Supports **SMS-based OTP** and **Authenticator App (TOTP)** for MFA.  
✔️ Securely stores **TOTP secret keys** for verification.  
✔️ Sends **QR codes** for easy authenticator app setup.  
✔️ Logs MFA verification attempts for auditing.  

Let me know if you'd like to refine any sections or add additional details! 🚀
