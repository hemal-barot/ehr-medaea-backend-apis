# API Reference — Authentication
## Base path: `/api/v1/auth`
## Updated: April 2026

---

## POST /auth/signup

Register a new provider account.

**Content-Type**: `application/json`  
**Auth**: None required

### Request Body

```json
{
  "email": "provider@hospital.org",     // Required — must be unique
  "password": "SecurePass1!",           // Required — min 8 chars, upper, lower, digit
  "first_name": "Jane",                 // Required
  "last_name": "Smith",                 // Optional
  "phone": "+15555555555",              // Optional — E.164 format
  "role": "doctor",                     // Optional — doctor|nurse|admin|staff (default: doctor)
  "specialty": "Internal Medicine"      // Optional
}
```

### Responses

| Code | Description |
|---|---|
| 201 | Account created. Returns JWT or "check your email" message |
| 409 | Email already registered |
| 422 | Validation error (password requirements, missing required fields) |

### Response (no email verification)

```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "user": {
    "id": "uuid",
    "email": "provider@hospital.org",
    "first_name": "Jane",
    "last_name": "Smith",
    "role": "doctor",
    "mfa_enabled": false,
    "is_verified": true
  }
}
```

### Response (email verification required)

```json
{
  "message": "Account created. Please check your email to verify your account."
}
```

---

## POST /auth/login

Authenticate and receive a JWT token.

**Content-Type**: `application/x-www-form-urlencoded`  
**Auth**: None

> Note: Uses OAuth2 form format (not JSON) for compatibility with OAuth2 standards.

### Request Body (form-encoded)

```
username=provider@hospital.org&password=SecurePass1!
```

### Responses

| Code | Description |
|---|---|
| 200 | Success — returns JWT or MFA challenge |
| 401 | Invalid credentials |
| 403 | Email not verified |

### Response (success, no MFA)

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "user": { ... }
}
```

### Response (MFA required)

```json
{
  "mfa_required": true,
  "mfa_method": "authenticator",
  "message": "Enter your 6-digit authentication code."
}
```

---

## POST /auth/verify-email

Verify a provider's email address.

**Content-Type**: `application/json`  
**Auth**: None

```json
{ "token": "48-char-url-safe-token-from-email" }
```

| Code | Description |
|---|---|
| 200 | Email verified |
| 400 | Token invalid or expired |

---

## POST /auth/resend-verification

Resend the email verification link.

```json
{ "email": "provider@hospital.org" }
```

| Code | Description |
|---|---|
| 200 | Email sent (always 200 — never reveals whether email exists) |

---

## POST /auth/forgot-password

Initiate password reset.

```json
{ "email": "provider@hospital.org" }
```

| Code | Description |
|---|---|
| 200 | Always 200 (HIPAA — never reveals if email exists) |

---

## POST /auth/reset-password

Set a new password using a reset token.

```json
{
  "token": "reset-token-from-email",
  "new_password": "NewSecurePass1!"
}
```

| Code | Description |
|---|---|
| 200 | Password reset successfully |
| 400 | Token invalid or expired |
| 422 | Password doesn't meet requirements |

---

## POST /auth/mfa/setup

Initiate MFA enrollment. **Requires JWT.**

```json
{ "method": "authenticator" }  // authenticator | sms | email
```

### Response (authenticator)

```json
{
  "qr_url": "otpauth://totp/Medaea:provider@hospital.org?secret=BASE32SECRET&issuer=Medaea",
  "secret": "BASE32TOTP SECRET",
  "message": "Scan the QR code with your authenticator app, then call /mfa/finalize."
}
```

---

## POST /auth/mfa/finalize

Confirm MFA enrollment. **Requires JWT.**

```json
{ "code": "123456" }
```

| Code | Description |
|---|---|
| 200 | MFA enabled — backup codes returned |
| 400 | Invalid TOTP code |

---

## POST /auth/mfa/verify

Verify MFA code during login (second factor).

```json
{
  "email": "provider@hospital.org",
  "code": "123456"
}
```

| Code | Description |
|---|---|
| 200 | Verified — returns JWT access_token |
| 401 | Invalid or expired code |

---

## GET /users/me

Get current user profile. **Requires JWT.**

### Response

```json
{
  "id": "uuid",
  "email": "provider@hospital.org",
  "first_name": "Jane",
  "last_name": "Smith",
  "phone": "+15555555555",
  "role": "doctor",
  "specialty": "Internal Medicine",
  "npi": "1234567890",
  "provider_type": "MD",
  "avatar_url": null,
  "mfa_enabled": true,
  "mfa_method": "authenticator",
  "is_verified": true,
  "organizations": [
    {
      "id": "org-uuid",
      "name": "Phoenix Medical Center",
      "org_type": "clinic",
      "role": "doctor",
      "department": "Internal Medicine"
    }
  ]
}
```
