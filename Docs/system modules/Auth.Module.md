# API Specification — Module 1: Authentication & Guest Session Management

**Document Version:** 1.0
**Module:** AUTH
**Base URL:** `/api/v1/auth`

---

## 1. Module Overview

The **AUTH Module** is responsible for:

* Creating temporary Guest Sessions.
* Registering user accounts.
* Migrating eligible anonymous assessments into registered accounts.
* Authenticating registered users.
* Refreshing access tokens.
* Logging users out and revoking refresh tokens.
* Retrieving the currently authenticated user's profile.
* Handling password reset requests and confirmation.

The module supports both:

* **Anonymous / Guest Users**
* **Registered / Authenticated Users**

---

## 2. Authentication Model

| Client State    | Authentication Mechanism               |
| --------------- | -------------------------------------- |
| Guest User      | `X-Session-ID`                         |
| Registered User | `Authorization: Bearer <access_token>` |
| Token Refresh   | `refresh_token`                        |
| Logout          | Authenticated session + refresh token  |

---

# 3. API Endpoints

---

## 3.1 Create Guest Session

### Use Case

**Create Guest Session**

### Requirement References

`AUTH-001`, `BR-001`, `BR-003`

### Endpoint

```http
POST /api/v1/auth/guest-session
```

### Authentication

None.

### Request Body

```json
{
  "client_info": {
    "preferred_language": "ar"
  }
}
```

### Response — `201 Created`

```json
{
  "status": "success",
  "data": {
    "guest_session_id": "gst_9f8e7d6c5b4a3120",
    "created_at": "2026-09-22T15:28:15Z",
    "expires_at": "2026-09-23T15:28:15Z",
    "assessment_limit": 1,
    "remaining_assessments": 1
  }
}
```

### Purpose

Creates a temporary anonymous session that allows the user to start an assessment without creating an account.

---

## 3.2 Register User Account

### Use Case

**Register User Account with Guest Session Migration**

### Requirement References

`AUTH-003`, `AUTH-005`, `BR-004`, `BR-005`

### Endpoint

```http
POST /api/v1/auth/register
```

### Authentication

None.

### Request Body

```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "guest_session_id": "gst_9f8e7d6c5b4a3120"
}
```

### Response — `201 Created`

```json
{
  "status": "success",
  "message": "User registered successfully and guest assessment migrated.",
  "data": {
    "user_id": "usr_1122334455",
    "email": "user@example.com",
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "d7a8f9e0b1c2d3e4f5a6b7c8d9e0f1a2",
    "token_type": "Bearer",
    "expires_in": 3600,
    "migrated_assessment": {
      "assessment_id": "rpt_9988776655",
      "status": "ASSOCIATED"
    }
  }
}
```

### Purpose

Creates a registered user account and, when a valid eligible guest session is supplied, associates the guest assessment with the newly created account.

### Important Rule

Guest assessment migration is treated as part of the registration transaction rather than as a separate public API endpoint.

---

## 3.3 Authenticate User

### Use Case

**Authenticate User**

### Requirement References

`AUTH-004`, `SEC-001`, `SEC-003`

### Endpoint

```http
POST /api/v1/auth/login
```

### Authentication

None.

### Request Body

```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

### Response — `200 OK`

```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "d7a8f9e0b1c2d3e4f5a6b7c8d9e0f1a2",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": "usr_1122334455",
      "email": "user@example.com"
    }
  }
}
```

### Purpose

Validates user credentials and issues authentication tokens.

---

## 3.4 Refresh Access Token

### Use Case

**Refresh Access Token**

### Requirement References

`AUTH-004`, `SEC-005`

### Endpoint

```http
POST /api/v1/auth/refresh-token
```

### Authentication

None.

### Request Body

```json
{
  "refresh_token": "d7a8f9e0b1c2d3e4f5a6b7c8d9e0f1a2"
}
```

### Response — `200 OK`

```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "e8b9f0a1b2c3d4e5f6a7b8c9d0e1f2a3",
    "token_type": "Bearer",
    "expires_in": 3600
  }
}
```

### Purpose

Issues a new access token using a valid refresh token.

---

## 3.5 Logout User

### Use Case

**Logout User**

### Requirement References

`AUTH-006`

### Endpoint

```http
POST /api/v1/auth/logout
```

### Headers

```http
Authorization: Bearer <access_token>
```

### Request Body

```json
{
  "refresh_token": "e8b9f0a1b2c3d4e5f6a7b8c9d0e1f2a3"
}
```

### Response — `200 OK`

```json
{
  "status": "success",
  "message": "User logged out successfully and tokens revoked."
}
```

### Purpose

Terminates the authenticated session and revokes the associated refresh token.

---

## 3.6 Retrieve Current User Profile

### Use Case

**Retrieve Current User Profile**

### Requirement References

`AUTH-004`, `SEC-005`

### Endpoint

```http
GET /api/v1/auth/me
```

### Headers

```http
Authorization: Bearer <access_token>
```

### Request Body

None.

### Response — `200 OK`

```json
{
  "status": "success",
  "data": {
    "user_id": "usr_1122334455",
    "email": "user@example.com",
    "is_active": true,
    "created_at": "2026-09-22T15:28:15Z"
  }
}
```

### Purpose

Retrieves the profile of the currently authenticated user.

---

## 3.7 Request Password Reset

### Use Case

**Request Password Reset**

### Requirement References

`AUTH-006`

### Endpoint

```http
POST /api/v1/auth/password-reset/request
```

### Authentication

None.

### Request Body

```json
{
  "email": "user@example.com"
}
```

### Response — `200 OK`

```json
{
  "status": "success",
  "message": "If the email is registered, a password reset link has been sent."
}
```

### Purpose

Starts the password recovery process without revealing whether the supplied email is registered.

---

## 3.8 Confirm Password Reset

### Use Case

**Confirm Password Reset**

### Requirement References

`AUTH-006`, `SEC-003`

### Endpoint

```http
POST /api/v1/auth/password-reset/confirm
```

### Authentication

None.

### Request Body

```json
{
  "reset_token": "rst_token_abc123xyz",
  "new_password": "NewSecurePassword456!"
}
```

### Response — `200 OK`

```json
{
  "status": "success",
  "message": "Password updated successfully. Please login with your new credentials."
}
```

### Purpose

Validates the password-reset token and updates the user's password.

---

# 4. Internal / Cross-Module Use Cases

Not every AUTH-related Use Case requires its own HTTP endpoint.

| Requirement | Use Case                       | API Endpoint | Implementation                          |
| ----------- | ------------------------------ | ------------ | --------------------------------------- |
| `AUTH-002`  | Complete Anonymous Assessment  | N/A          | Cross-module flow using `X-Session-ID`  |
| `AUTH-005`  | Migrate Guest Assessment       | N/A          | Executed atomically during registration |
| `BR-006`    | Enforce Guest Assessment Limit | N/A          | Internal backend gate                   |

### Guest Assessment Limit

The backend evaluates the guest assessment limit when an assessment is started.

If the guest session has exceeded its allowed assessment count, the request is rejected.

Expected result:

```http
403 Forbidden
```

---

# 5. AUTH Use Case → API Mapping

| Requirement ID | Use Case                       | Direct API Endpoint                   | Method | Context                          |
| -------------- | ------------------------------ | ------------------------------------- | ------ | -------------------------------- |
| `AUTH-001`     | Create Guest Session           | `/api/v1/auth/guest-session`          | POST   | Creates temporary guest session  |
| `AUTH-002`     | Complete Anonymous Assessment  | N/A                                   | —      | Cross-module flow                |
| `AUTH-003`     | Register User Account          | `/api/v1/auth/register`               | POST   | Creates registered account       |
| `AUTH-004`     | Authenticate User              | `/api/v1/auth/login`                  | POST   | Validates credentials            |
| `AUTH-004`     | Refresh Access Token           | `/api/v1/auth/refresh-token`          | POST   | Refreshes authentication         |
| `AUTH-004`     | Retrieve Current User Profile  | `/api/v1/auth/me`                     | GET    | Retrieves authenticated user     |
| `AUTH-005`     | Migrate Guest Assessment       | N/A                                   | —      | Part of registration transaction |
| `AUTH-006`     | Logout User                    | `/api/v1/auth/logout`                 | POST   | Revokes refresh token            |
| `AUTH-006`     | Request Password Reset         | `/api/v1/auth/password-reset/request` | POST   | Starts password recovery         |
| `AUTH-006`     | Confirm Password Reset         | `/api/v1/auth/password-reset/confirm` | POST   | Updates password                 |
| `BR-006`       | Enforce Guest Assessment Limit | N/A                                   | —      | Internal backend rule            |

---

# 6. AUTH Open Questions

The following decisions should be finalized before implementation:

1. Should logout require both the access token and refresh token, or should the refresh token alone identify the session to revoke?
2. Should refresh tokens be rotated on every successful refresh?
3. Should password reset invalidate all existing sessions/tokens?
4. Should registering with an expired or already-migrated `guest_session_id` fail, or simply create the account without migration?
5. What exact HTTP error contract should be used for duplicate email, invalid credentials, expired guest sessions, and invalid refresh tokens?
