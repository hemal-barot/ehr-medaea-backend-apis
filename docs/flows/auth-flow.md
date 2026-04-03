# Authentication Flow — Medaea EHR

## Overview

The Medaea EHR authentication system follows a HIPAA-compliant multi-step flow designed for healthcare providers. It supports:

- Email + password authentication
- Email verification (enforced in production)
- Multi-Factor Authentication (TOTP app, SMS via Twilio, or email OTP)
- Secure password reset via email
- JWT-based session management

---

## Full Authentication Sequence

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PROVIDER SIGNUP FLOW                         │
└─────────────────────────────────────────────────────────────────────┘

Provider                   Frontend              FastAPI               Database
   │                           │                    │                      │
   │  Fill signup form         │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/signup │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Check email unique  │
   │                           │                    │─────────────────────▶│
   │                           │                    │  Hash password       │
   │                           │                    │  Create User row     │
   │                           │                    │─────────────────────▶│
   │                           │                    │                      │
   │                           │                    │  (if VERIFY=true)    │
   │                           │                    │  Send welcome email  │
   │                           │◀───────────────────│  201 Created         │
   │  "Check your email"       │                    │                      │
   │◀──────────────────────────│                    │                      │


┌─────────────────────────────────────────────────────────────────────┐
│                      EMAIL VERIFICATION FLOW                        │
└─────────────────────────────────────────────────────────────────────┘

Provider                   Frontend              FastAPI               Database
   │                           │                    │                      │
   │  Click email link         │                    │                      │
   │  /verify-email?token=xyz  │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  verify-email      │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Validate token      │
   │                           │                    │  Check expiry        │
   │                           │                    │  Set is_verified=true│
   │                           │                    │─────────────────────▶│
   │                           │◀───────────────────│  200 OK              │
   │  Redirect to /login       │                    │                      │
   │◀──────────────────────────│                    │                      │


┌─────────────────────────────────────────────────────────────────────┐
│                       LOGIN FLOW (No MFA)                           │
└─────────────────────────────────────────────────────────────────────┘

Provider                   Frontend              FastAPI               Database
   │                           │                    │                      │
   │  Enter email + password   │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/login  │                      │
   │                           │  form: username,   │                      │
   │                           │        password    │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Find user by email  │
   │                           │                    │─────────────────────▶│
   │                           │                    │  Verify bcrypt hash  │
   │                           │                    │  Check is_verified   │
   │                           │                    │  Check mfa_enabled   │
   │                           │                    │  (mfa_enabled=false) │
   │                           │                    │  Create JWT (8h TTL) │
   │                           │                    │  Update last_login   │
   │                           │◀───────────────────│  200 + access_token  │
   │  Dashboard loaded         │                    │                      │
   │◀──────────────────────────│                    │                      │


┌─────────────────────────────────────────────────────────────────────┐
│                       LOGIN FLOW (With MFA)                         │
└─────────────────────────────────────────────────────────────────────┘

Provider                   Frontend              FastAPI               Twilio/TOTP
   │                           │                    │                      │
   │  Enter email + password   │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/login  │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Verify credentials  │
   │                           │                    │  mfa_enabled = true  │
   │                           │                    │                      │
   │                           │                    │  (if SMS) Send OTP──▶│ Twilio SMS
   │                           │◀───────────────────│  { mfa_required:     │
   │                           │                    │    true,             │
   │                           │                    │    mfa_method: "sms"}│
   │  Show MFA code input      │                    │                      │
   │◀──────────────────────────│                    │                      │
   │  Enter 6-digit code       │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  mfa/verify        │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Verify TOTP/SMS     │
   │                           │                    │  code (30s window)   │
   │                           │                    │  Create JWT (8h TTL) │
   │                           │◀───────────────────│  200 + access_token  │
   │  Dashboard loaded         │                    │                      │
   │◀──────────────────────────│                    │                      │


┌─────────────────────────────────────────────────────────────────────┐
│                      PASSWORD RESET FLOW                            │
└─────────────────────────────────────────────────────────────────────┘

Provider                   Frontend              FastAPI               Email
   │  "Forgot Password"        │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  forgot-password   │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Find user (silent)  │
   │                           │                    │  Gen reset token     │
   │                           │                    │  Store (1h expiry)   │
   │                           │                    │  Send reset email───▶│
   │                           │◀───────────────────│  200 (always)        │
   │  "Check your email"       │                    │                      │
   │◀──────────────────────────│                    │                      │
   │  Click reset link         │                    │                      │
   │  /reset-password?token=   │                    │                      │
   │──────────────────────────▶│                    │                      │
   │  Enter new password       │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  reset-password    │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Validate token      │
   │                           │                    │  Check expiry        │
   │                           │                    │  Hash new password   │
   │                           │                    │  Clear reset token   │
   │                           │◀───────────────────│  200 OK              │
   │  Redirect to /login       │                    │                      │
   │◀──────────────────────────│                    │                      │
```

---

## MFA Enrollment Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MFA TOTP SETUP FLOW                              │
└─────────────────────────────────────────────────────────────────────┘

Provider (logged in)       Frontend              FastAPI             TOTP Library
   │                           │                    │                      │
   │  Enable MFA (Settings)    │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  mfa/setup         │                      │
   │                           │  {method: "auth.."}│                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Generate TOTP secret│
   │                           │                    │─────────────────────▶│
   │                           │                    │  Encrypt + store     │
   │                           │                    │  Build QR URL        │
   │                           │◀───────────────────│  { qr_url, secret }  │
   │  Show QR code to user     │                    │                      │
   │◀──────────────────────────│                    │                      │
   │  Scan with app            │                    │                      │
   │  Get 6-digit code         │                    │                      │
   │──────────────────────────▶│                    │                      │
   │                           │  POST /auth/       │                      │
   │                           │  mfa/finalize      │                      │
   │                           │  {code: "123456"}  │                      │
   │                           │───────────────────▶│                      │
   │                           │                    │  Verify TOTP code    │
   │                           │                    │  Set mfa_enabled=true│
   │                           │◀───────────────────│  200 + backup codes  │
   │  "MFA is now enabled"     │                    │                      │
   │  Save backup codes        │                    │                      │
   │◀──────────────────────────│                    │                      │
```

---

## JWT Token Structure

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user-uuid-here",
    "exp": 1743800000,
    "iat": 1743771200
  }
}
```

- **Algorithm**: HS256 (HMAC-SHA256)
- **TTL**: 8 hours (configurable via `ACCESS_TOKEN_EXPIRE_MINUTES`)
- **Claims**: `sub` (user UUID), `exp`, `iat`

---

## Security Considerations (HIPAA §164.312)

| Requirement | Implementation |
|---|---|
| Unique user identification | UUID primary key per user |
| Automatic logoff | JWT expiry (8h), frontend idle timer |
| Encryption and decryption | bcrypt password hashing (cost=12), AES for MFA secrets |
| Audit controls | All logins logged in `audit_logs` |
| Transmission security | HTTPS in production (TLS 1.2+) |
| MFA | TOTP + SMS support |
| Password complexity | Min 8 chars, uppercase, lowercase, digit |
