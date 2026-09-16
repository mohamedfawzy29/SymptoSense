Admin Module Specification



Module: Platform Administration

Version: 1

Status: Draft — Aligned with Approved User Roles \& Permissions, Business Rules, and Data \& Privacy documents



1\. Module Overview



The Admin Module provides the operational tools required to manage and monitor the SymptoSense platform. It lets a privileged Admin user search and manage registered accounts, view aggregated platform usage statistics, and review operational error logs.



The module does not perform medical assessment, does not view individual assessment content, and does not grant the Admin any special medical authority over the assessment flow.



Its primary responsibility is:



&#x20;   Expose a restricted, auditable set of operations for account management and platform monitoring, without exposing sensitive user or medical data beyond what is explicitly authorized.



2\. Main Responsibilities



The Admin Module is responsible for:



&#x20;   Allowing an Admin to search and view registered user accounts (ADM-001).

&#x20;   Allowing an Admin to activate or deactivate a user account (ADM-001).

&#x20;   Providing aggregated platform usage statistics — total assessments, completion rates, registration rates (ADM-002).

&#x20;   Providing access to operational/technical error logs (ADM-003).

&#x20;   Recording an audit trail of privileged administrative actions (ROLE-010).

&#x20;   Enforcing that every administrative action is authorized against the Admin role before execution (ROLE-005).



The module is not responsible for:



&#x20;   Viewing the content of any user's assessment — symptoms, answers, or AI conversation (ADM-002, ROLE-006).

&#x20;   Diagnosing, evaluating, or influencing any assessment outcome.

&#x20;   Accessing user passwords or authentication secrets (User Roles, Section 3 — Admin Restrictions).

&#x20;   Impersonating a user without an explicitly defined and audited support mechanism (User Roles, Section 3).

&#x20;   Modifying medical or safety rules (BR-075–BR-089 remain outside Admin's authority).

&#x20;   Managing user data without authorization, or bypassing security controls.

&#x20;   Acting as the source of truth for user account or assessment data — it reads and updates through the modules that own that data (Section 9).



3\. Admin Permission Model



Per the approved User Roles \& Permissions document, authorization must follow the principle of least privilege (ROLE-009). The role model is designed to be extensible rather than hard-coded, since a future Medical Reviewer role is already anticipated (User Roles, Section 4) even though it is out of scope for the MVP.



UserRole (enum)

├── Guest

├── RegisteredUser

├── Admin

└── MedicalReviewer   — reserved, not implemented in MVP



Only Admin may access any endpoint in this module. This is enforced by a dedicated authorization guard (Section 14), not by convention in individual controller methods.



4\. User Management Flow



Admin

&#x20; |

&#x20; v

Search / View Users

&#x20; |

&#x20; v

Admin Module: AdminRoleGuard passes?

&#x20; |

&#x20; +---- NO ----> 403 Forbidden (recorded as a security event, Data \& Privacy Section 33)

&#x20; |

&#x20;YES

&#x20; |

&#x20; v

IUserDirectoryService.SearchUsersAsync (Section 9)

&#x20; |

&#x20; v

Return user summaries (no passwords, no assessment content)



5\. Account Status Change Flow



Admin

&#x20; |

&#x20; v

Request: Deactivate / Activate User {userId}

&#x20; |

&#x20; v

AdminRoleGuard passes?

&#x20; |

&#x20; +---- NO ----> 403 Forbidden

&#x20; |

&#x20;YES

&#x20; |

&#x20; v

IUserDirectoryService.UpdateUserStatusAsync

&#x20; |

&#x20; v

Audit Log: record { adminId, action, targetUserId, timestamp } (Section 8)

&#x20; |

&#x20; v

Confirm to Admin



Every account status change is written to the Audit Log before the response is returned to the Admin — this is not an optional side effect (ROLE-010, Data \& Privacy Section 33).



6\. Statistics \& Monitoring Flow



Admin

&#x20; |

&#x20; v

Request Platform Stats

&#x20; |

&#x20; v

IAssessmentStatsService.GetAggregatedStatsAsync (Section 9)

&#x20; |

&#x20; v

{ totalStarted, totalCompleted, totalAbandoned, totalDrafts, registrationRate }

&#x20; |

&#x20; v

Return to Admin



No individual assessment identifier, symptom, or result ever appears in this response — only counts.



7\. Constraints



&#x20;   Every query the Admin Module issues against assessment data must return an aggregate (count, rate, or similar) — never a row-level assessment record. This constraint is enforced at the interface boundary exposed by the Assessment Module (Section 9), not merely by convention inside the Admin Module.

&#x20;   Every action that modifies state (e.g., account status) must produce exactly one Audit Log entry — no modifying action is allowed to complete without one.

&#x20;   The Admin Module never queries the Users table directly if a shared database is used — it goes through the Auth Module's exposed interface (Section 9), to avoid two modules independently owning access logic for the same data.



8\. Contracts



Search Users — Response



{

&#x20; "users": \[

&#x20;   {

&#x20;     "userId": "u-9001",

&#x20;     "emailMasked": "a\*\*\*\*@example.com",

&#x20;     "status": "active",

&#x20;     "createdAt": "2026-07-02T10:00:00Z"

&#x20;   }

&#x20; ]

}



Update User Status — Request



{

&#x20; "status": "disabled"

}



Platform Stats — Response



{

&#x20; "totalAssessmentsStarted": 4210,

&#x20; "totalAssessmentsCompleted": 3190,

&#x20; "totalAssessmentsAbandoned": 740,

&#x20; "totalDrafts": 280,

&#x20; "registrationRate": 0.42

}



Audit Log Entry (internal / write side)



{

&#x20; "adminId": "admin-01",

&#x20; "action": "user\_status\_updated",

&#x20; "targetResourceId": "u-9001",

&#x20; "timestamp": "2026-09-14T09:12:00Z"

}



No contract in this module ever includes assessment content, symptom data, or authentication secrets.



9\. Relationship With Other Modules



+---------------+

|  Auth Module  |  <---- user search / status updates (via IUserDirectoryService)

+---------------+

&#x20;       ^

&#x20;       |

+----------------+          +---------------------+

|  Admin Module   | -------> |  Assessment Module   |  ---- aggregate stats only

+----------------+          +---------------------+

&#x20;       |

&#x20;       v

+------------------+

|  Audit Log Store  |  (owned by Admin Module)

+------------------+

&#x20;       |

&#x20;       v

+-------------------+

| Error Log Reader   |  (read-only, shared logging store)

+-------------------+



The Admin Module never communicates with the AI Module or the 3D Module — it has no operational reason to. Its only data dependencies are the Auth Module (account data), the Assessment Module (aggregate statistics only), and its own Audit Log store.



10\. Assessment Data Boundary



This is the most important boundary in this module, mirroring the AI Module's Safety boundary (AI Module Specification, Section 2).



Per ROLE-006: "Administrative access does not automatically grant unrestricted access to user medical data." Per the User Roles document's Open Questions, whether an Admin may ever view a specific user's assessment content is explicitly unresolved by the team.



Until that question is formally resolved:



&#x20;   The Admin Module implements zero capability to retrieve individual assessment content, by design — not merely by omission.

&#x20;   IAssessmentStatsService (Section 9) is defined so that it structurally cannot return row-level data — its return type is a fixed aggregate DTO (Section 17), not a queryable assessment repository.

&#x20;   If the team later approves a controlled, audited exception (e.g., for a support investigation), it should be added as a distinct, separately authorized capability — not as a relaxation of this module's default queries.



11\. Data Ownership \& Statelessness



The Admin Module does not own user account data or assessment data — it reads and updates them through the owning modules' interfaces (Section 9). It owns exactly one thing directly: the Audit Log.



Data                        Owned By              Admin Module Access

User accounts                Auth Module            Read + limited update (status only), via interface

Assessment records            Assessment Module       Aggregate read only, via interface

Error logs                    Shared logging store    Read only

Audit log                     Admin Module             Read + write (owns this data)



12\. Database



The Admin Module requires its own storage for exactly one thing: the Audit Log (e.g., an `audit\_logs` table). It does not require a database for users, assessments, or error logs — those are read through the modules that own them.



audit\_logs

├── id

├── admin\_id

├── action

├── target\_resource\_id

└── timestamp



13\. Backend Responsibility



The backend enforces authorization before any business logic runs, and enforces the aggregate-only boundary described in Section 10 at the interface level, not only in the controller.



Admin Request

&#x20;     |

&#x20;     v

AdminRoleGuard (Section 14)

&#x20;     |

&#x20;     v

Application Layer (Section 18)

&#x20;     |

&#x20;     v

Auth Module / Assessment Module interfaces (read/update, aggregate-only)

&#x20;     |

&#x20;     v

Audit Log (for any modifying action)



14\. Authorization Guard



public interface IAdminRoleGuard

{

&#x20;   bool IsAuthorized(ClaimsPrincipal user);

}



AdminRoleGuard checks that the authenticated user's role claim equals Admin before any controller method in this module executes. This is a dedicated guard rather than a shared generic authentication check, so that admin-specific authorization logic stays in one place and is easy to audit (ROLE-005, ROLE-009).



15\. Backend API



GET /api/admin/users?query={search term}



Response: Search Users response shown in Section 8.



PATCH /api/admin/users/{userId}/status



Request: Update User Status request shown in Section 8.

Effect: Updates status via the Auth Module's interface, then writes one Audit Log entry.



GET /api/admin/stats



Response: Platform Stats response shown in Section 8.



GET /api/admin/logs



Response: recent operational error log entries (ADM-003) — technical information only, no sensitive user or medical content (Data \& Privacy, Section 31).



Final route naming and payload shape remain an implementation detail, consistent with the AI and 3D Module specifications.



16\. Clean Architecture



Admin Module

│

├── Domain

│   ├── Entities

│   └── Value Objects

│

├── Application

│   ├── DTOs

│   ├── Interfaces

│   ├── Use Cases

│   └── Services

│

├── Infrastructure

│   ├── Auth Module Client

│   ├── Assessment Module Client

│   └── Audit Log Repository

│

└── Presentation

&#x20;   └── API Controllers



17\. Domain Layer



UserSummary

├── UserId

├── EmailMasked

├── Status

└── CreatedAt



PlatformStats

├── TotalAssessmentsStarted

├── TotalAssessmentsCompleted

├── TotalAssessmentsAbandoned

├── TotalDrafts

└── RegistrationRate



AuditLogEntry

├── AdminId

├── Action

├── TargetResourceId

└── Timestamp



No dependency on ASP.NET, Entity Framework, HTTP, controllers, or any specific database technology.



18\. Application Layer



public class UserSummaryDto

{

&#x20;   public string UserId { get; set; }

&#x20;   public string EmailMasked { get; set; }

&#x20;   public string Status { get; set; } // "active" | "disabled"

&#x20;   public DateTime CreatedAt { get; set; }

}



public class UpdateUserStatusRequestDto

{

&#x20;   public string Status { get; set; } // "active" | "disabled"

}



public class PlatformStatsDto

{

&#x20;   public int TotalAssessmentsStarted { get; set; }

&#x20;   public int TotalAssessmentsCompleted { get; set; }

&#x20;   public int TotalAssessmentsAbandoned { get; set; }

&#x20;   public int TotalDrafts { get; set; }

&#x20;   public double RegistrationRate { get; set; }

}



public class AuditLogEntryDto

{

&#x20;   public string AdminId { get; set; }

&#x20;   public string Action { get; set; }

&#x20;   public string TargetResourceId { get; set; }

&#x20;   public DateTime Timestamp { get; set; }

}



public interface IUserDirectoryService

{

&#x20;   Task<List<UserSummaryDto>> SearchUsersAsync(string query);

&#x20;   Task UpdateUserStatusAsync(string userId, UpdateUserStatusRequestDto request);

}



public interface IAssessmentStatsService

{

&#x20;   Task<PlatformStatsDto> GetAggregatedStatsAsync();

}



public interface IAuditLogService

{

&#x20;   Task RecordAsync(AuditLogEntryDto entry);

}



public interface IErrorLogReaderService

{

&#x20;   Task<List<string>> GetRecentErrorsAsync(int limit);

}



IUserDirectoryService and IAssessmentStatsService are implemented by adapters over the Auth and Assessment Modules respectively (Section 19) — the Admin Module's Application layer never talks to their underlying data stores directly.



19\. Infrastructure Layer



AuthModuleClient : IUserDirectoryService

&#x20;   — calls the Auth Module's own exposed interface/API for search and status updates.



AssessmentModuleClient : IAssessmentStatsService

&#x20;   — calls the Assessment Module's exposed aggregate-stats endpoint only; this client has no method capable of returning a single assessment record.



AuditLogRepository : IAuditLogService

&#x20;   — the only data store this module owns directly (Section 12).



ErrorLogReader : IErrorLogReaderService

&#x20;   — reads from the platform's shared logging store, filtered to technical/operational entries only.



20\. Presentation Layer



AdminController



GET    /api/admin/users

PATCH  /api/admin/users/{userId}/status

GET    /api/admin/stats

GET    /api/admin/logs



Every method is decorated with AdminRoleGuard (Section 14). The controller delegates directly to the Application layer; no authorization or business logic lives in the controller itself beyond the guard.



21\. Open Items Requiring Team Confirmation



&#x20;   Admin visibility into individual assessments. Still explicitly unresolved by the User Roles document. This design assumes "no" by default (Section 10) until the team formally decides otherwise.

&#x20;   Cross-module data access pattern. Confirm whether the Auth and Assessment Modules will expose internal service interfaces/APIs for the Admin Module to call (assumed here), or whether all modules will share direct database access — the former is recommended to preserve module boundaries.

&#x20;   Audit Log access control. Confirm who, if anyone, can read the Audit Log in the MVP — this document assumes it is write-heavy and not yet exposed through any Admin-facing read endpoint.

&#x20;   Content Management scope. The User Roles document states the Admin "may eventually" manage non-clinical content — confirm whether this is in MVP scope; it is excluded from this design until confirmed.

&#x20;   Error log sensitivity filtering. Confirm the exact boundary between "technical information useful for debugging" and information that must be excluded per Data \& Privacy Section 31.



22\. Summary



Admin Authenticates

&#x20;     ↓

AdminRoleGuard Authorizes

&#x20;     ↓

&#x20;  ┌──────────────┬───────────────────┬──────────────────┐

&#x20;  ↓              ↓                   ↓                  ↓

Search/Manage   View Aggregated    View Error Logs   (any modifying

Users (via      Stats (via                            action writes

Auth Module)    Assessment Module,                     an Audit Log

&#x20;               aggregate only)                        entry)



The Admin Module never becomes a second source of truth for user or assessment data, and never gains a path — direct or indirect — to an individual user's assessment content. Its authority is limited to account operations, aggregate visibility, and operational monitoring, all of it auditable.

