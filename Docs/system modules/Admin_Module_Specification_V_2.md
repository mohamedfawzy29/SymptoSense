# API Specification — Module 5: System Administration & Operational Management

**Document Version:** 1.0  
**Module:** ADMIN  
**Base URL:** `/api/v1/admin`

---

## 1. Module Overview

The **ADMIN Module** is responsible for:
* Managing registered user account statuses (activation and deactivation).
* Searching and querying registered user accounts with non-clinical profile metadata.
* Retrieving aggregated, anonymized platform metrics, conversion rates, and safety escalation counts.
* Inspecting operational error logs, API downtime events, and component failure traces.
* Automatically capturing immutable audit log records for all administrative operations.
* Enforcing strict medical data privacy isolation and clinical separation boundaries.

The module supports:
* **System Administrators (`Admin` Role)** with backend-enforced Role-Based Access Control (RBAC).

---

## 2. Authorization & Security Model

| Client State / Role | Authorization Mechanism | Access Scope & Constraints |
| :--- | :--- | :--- |
| **System Admin** | `Authorization: Bearer <access_token>` | Full access to administrative endpoints; strictly prohibited from viewing raw user medical history or symptom chats (`BR-008`). |
| **Audit Interceptor** | Internal Backend Middleware | Automatic append-only logging of any state-changing administrative action (`ROLE-010`, `Req-33`). |

---

## 3. API Endpoints

### 3.1 Search & List Users

#### Use Case
Search & List Users

#### Requirement References
`ADM-001`, `BR-008`, `BR-009`

#### Endpoint
`GET /api/v1/admin/users`

#### Authentication
`Authorization: Bearer <access_token>` (Role: `Admin`)

#### Request Parameters (Query)
```json
{
  "query": "ahmed",
  "status": "active",
  "page": 1,
  "limit": 20
}
```

#### Response — 200 OK
```json
{
  "status": "success",
  "data": {
    "users": [
      {
        "user_id": "usr_1122334455",
        "email": "user@example.com",
        "display_name": "Ahmed Ali",
        "status": "active",
        "created_at": "2026-08-10T14:22:00Z",
        "last_login_at": "2026-08-22T09:15:30Z",
        "total_assessments_completed": 5
      }
    ],
    "pagination": {
      "total": 150,
      "page": 1,
      "limit": 20,
      "total_pages": 8
    }
  }
}
```

#### Purpose
Retrieves a paginated list of registered user accounts with basic status metadata while excluding sensitive medical data.

---

### 3.2 Update User Account Status

#### Use Case
Update User Account Status

#### Requirement References
`ADM-001`, `BR-009`, `ROLE-010`, `BR-ADM-001`

#### Endpoint
`PATCH /api/v1/admin/users/{userId}/status`

#### Authentication
`Authorization: Bearer <access_token>` (Role: `Admin`)

#### Request Body
```json
{
  "status": "deactivated",
  "reason": "Administrative suspension due to terms of service violation"
}
```

#### Response — 200 OK
```json
{
  "status": "success",
  "message": "User status updated successfully.",
  "data": {
    "user_id": "usr_1122334455",
    "previous_status": "active",
    "new_status": "deactivated",
    "updated_at": "2026-09-22T18:00:00Z",
    "audit_log_id": "audit_99887766"
  }
}
```

#### Purpose
Activates or deactivates a user account and triggers an immutable audit log entry.

#### Important Rule
Administrators cannot deactivate their own administrative accounts (`BR-ADM-001`).

---

### 3.3 Retrieve Platform Aggregated Statistics

#### Use Case
Retrieve Platform Aggregated Statistics

#### Requirement References
`ADM-002`, `BR-052`, `BR-053`, `BR-008`

#### Endpoint
`GET /api/v1/admin/stats`

#### Authentication
`Authorization: Bearer <access_token>` (Role: `Admin`)

#### Request Parameters (Query)
```json
{
  "period": "7d"
}
```

#### Response — 200 OK
```json
{
  "status": "success",
  "data": {
    "timeframe": "7d",
    "overview": {
      "total_assessments_started": 1250,
      "total_assessments_completed": 1050,
      "successful_completion_rate": 84.0,
      "guest_assessments": 450,
      "registered_assessments": 800,
      "guest_to_registration_conversions": 95,
      "conversion_rate": 21.1
    },
    "safety": {
      "emergency_escalations_triggered": 32,
      "safety_interruption_rate": 2.56
    },
    "performance": {
      "average_completion_time_seconds": 185,
      "ai_response_latency_ms": 420
    }
  }
}
```

#### Purpose
Retrieves high-level usage, completion, and safety escalation metrics in anonymized format without exposing user PII or symptom inputs.

---

### 3.4 Retrieve Operational Logs

#### Use Case
Retrieve Operational Logs

#### Requirement References
`ADM-003`, `Req-31`, `Req-32`, `BR-ADM-004`

#### Endpoint
`GET /api/v1/admin/logs`

#### Authentication
`Authorization: Bearer <access_token>` (Role: `Admin`)

#### Request Parameters (Query)
```json
{
  "severity": "error",
  "service": "ai-engine",
  "limit": 50
}
```

#### Response — 200 OK
```json
{
  "status": "success",
  "data": {
    "logs": [
      {
        "log_id": "log_0019283",
        "timestamp": "2026-09-22T17:45:12Z",
        "severity": "error",
        "service": "ai-engine",
        "error_code": "AI_TIMEOUT_EXCEEDED",
        "message": "Downstream LLM provider failed to respond within 5000ms",
        "context": {
          "session_id": "sess_anon_991823",
          "retry_attempt": 2
        }
      }
    ]
  }
}
```

#### Purpose
Inspects system error logs and operational health events. All personal identifiers (emails, passwords, JWT tokens) are automatically redacted prior to response.

---

## 4. Internal / Cross-Module Use Cases

Not every ADMIN-related Use Case requires a standalone public HTTP request.

| Requirement ID | Use Case | API Endpoint | Implementation |
| :--- | :--- | :--- | :--- |
| `ROLE-010`, `Req-33` | Audit Trail Logging | N/A | Executed automatically via backend `AuditLoggingInterceptor` on state-changing operations |
| `ROLE-006`, `BR-008` | Enforce Medical Data Isolation | N/A | Applied via DTO filters and repository queries that exclude clinical tables |
| `BR-036`, `BR-037` | Assessment Engine Separation | N/A | Strict boundary preventing Admin APIs from mutating clinical logic or AI assessment outputs |
| `BR-ADM-001` | Enforce Self-Deactivation Limit | N/A | Internal Use Case gate throwing `400 Bad Request` if `admin_id == target_user_id` |

#### Operational & Privacy Rules

##### Medical Isolation Gate
The backend strictly prevents any database joins or queries connecting Admin endpoints to `chat_histories`, `symptom_inputs`, or `assessment_reports`.

##### Audit Immutability Gate
Audit entries recorded in the `audit_logs` table are append-only. Database permissions prevent `UPDATE` or `DELETE` queries on audit records.

---

## 5. ADMIN Use Case → API Mapping

| Requirement ID | Use Case | Direct API Endpoint | Method | Context |
| :--- | :--- | :--- | :--- | :--- |
| `ADM-001` | Search & List Users | `/api/v1/admin/users` | `GET` | Searches registered users with non-clinical metadata |
| `ADM-001` | Update User Account Status | `/api/v1/admin/users/{userId}/status` | `PATCH` | Activates/deactivates accounts and logs audit record |
| `ADM-002` | Retrieve Aggregated Statistics | `/api/v1/admin/stats` | `GET` | Aggregates system metrics & safety escalation counts |
| `ADM-003` | Retrieve Operational Logs | `/api/v1/admin/logs` | `GET` | Fetches operational error logs with auto-masked PII |
| `ROLE-010` | Audit Trail Logging | N/A | — | Internal interceptor capturing administrative actions |
| `ROLE-006` | Enforce Medical Data Isolation | N/A | — | Domain-level security filter blocking access to medical data |
| `BR-ADM-001` | Enforce Self-Targeting Limit | N/A | — | Internal backend check preventing self-deactivation |

---

## 6. ADMIN Open Questions

The following decisions should be finalized before implementation:
1. **Token Invalidation on Deactivation:** Should deactivating a user account immediately invalidate all active JWT refresh tokens in Redis/Database?
2. **Log Retention Policy:** What retention policy should apply to operational logs (e.g., auto-purge after 90 days vs. cold storage archiving)?
3. **Real-time Alerting:** Should system administrators receive real-time notifications/webhooks for `fatal` severity logs or critical service downtime?
4. **Audit Log Access:** Should audit logs be queryable via an internal `GET /api/v1/admin/audit-logs` endpoint or reserved exclusively for compliance database exports?
5. **Error Contract Consistency:** What exact HTTP error contract should be returned when an Admin attempts a forbidden action (e.g., self-deactivation returning `400 Bad Request` vs `403 Forbidden`)?