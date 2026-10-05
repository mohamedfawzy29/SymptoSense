# Functional Requirements Document (FRD)

**Document Version:** 1.0

**Status:** Approved

**Last Updated:** 2026-09-16

---

## 1. Overview & Scope

This document specifies the Functional Requirements for the Minimum Viable Product (MVP) of the **Smart Waste Bin System**. All functional requirements herein are strictly derived from the approved Roles & Permissions, Technical Specification Document (TSD), and User Journeys.

Each requirement follows the format:
`[Module Code]-[Sequential Number]: Requirement Description`

---

## 2. System Functional Modules

---

### Module 1: Identity & Access Management (IAM)

* **IAM-001 (Citizen Registration & Authentication):** The system MUST support user registration and authentication via Email/Password or third-party providers (Google/Apple Sign-In), issuing JWT-based access tokens.


* **IAM-002 (Admin Authentication):** The system MUST provide a secure authentication gateway for administrative users accessing the web dashboard.


* **IAM-003 (Profile Management):** The system MUST allow authenticated Citizens to view and update their personal profile details (display name, avatar, contact info).


* **IAM-004 (Account Deletion & Anonymization):** When a Citizen requests account deletion, the system MUST anonymize user-identifying references in historical transactions and ledgers rather than performing physical hard deletes.


* **IAM-005 (User Moderation):** The system MUST allow Admins to search users, inspect their disposal and dispute history, and suspend or ban accounts exhibiting fraudulent behavior.


* **IAM-006 (Account Status Enforcement):** The system MUST automatically block suspended or banned accounts from claiming points, submitting disputes, or redeeming vouchers.



---

### Module 2: Ingestion & AI Classification Engine (AIE)

* **AIE-001 (Hardware Image Ingestion):** The Backend API MUST accept `multipart/form-data` requests from registered Smart Bins containing captured waste images and telemetry fill-level data.


* **AIE-002 (In-Memory Inference):** The system MUST process ingested images using the embedded ML.NET inference engine to predict the waste category (`WasteType`) and assign a `ConfidenceScore`.


* **AIE-003 (Points Calculation):** The system MUST evaluate the predicted `WasteType` against a points matrix to determine calculated base reward points.


* **AIE-004 (Disposal Session Creation):** The system MUST generate a `DisposalSession` record containing the classification output, calculated points, and an explicit 90-second Time-To-Live expiration timestamp (`ExpiresAt = UtcNow + 90s`).


* **AIE-005 (Dynamic QR Generation):** The system MUST generate a cryptographically random `SessionToken` payload and return it to the smart bin display as a dynamic QR code.



---

### Module 3: Session Claiming & Point Validation (CLAIM)

* **CLAIM-001 (QR Token Scanning):** The Mobile App MUST allow authenticated Citizens to scan the dynamic QR code displayed on the bin and submit the `SessionToken` to the API.


* **CLAIM-002 (TTL Expiration Verification):** The Backend MUST validate that the scanned session token exists and that the current UTC time is strictly less than `ExpiresAt`.


* **CLAIM-003 (Result Inspection):** Upon successful token validation, the system MUST return the detected waste type, captured image, and estimated reward points to the Mobile App for user review.


* **CLAIM-004 (Atomic Claim Execution):** When a Citizen accepts the result, the system MUST execute an ACID transaction that marks the session as `Claimed`, binds the session to the authenticated `userId`, and credits the calculated points to the user's wallet.


* **CLAIM-005 (Anti-Double Claiming & Concurrency):** The system MUST enforce database concurrency controls (optimistic locking) to reject concurrent or duplicate claims against the same `SessionToken`.



---

### Module 4: Dispute Management & Active Learning (DISP)

* **DISP-001 (Immediate Dispute Submission):** The system MUST allow Citizens to contest an AI classification on the spot during the session claim window by submitting a `DisputeTicket` containing the claimed waste category and optional notes.


* **DISP-002 (Transaction Parking):** Upon dispute submission, the system MUST transition the session status to `Disputed` and lock the transaction state pending human review.


* **DISP-003 (Dispute Rate Limiting):** The system MUST enforce a sliding-window rate limit on dispute submissions (e.g., maximum 3 disputes per 24 hours per user) to mitigate spam.


* **DISP-004 (Admin Review Queue):** The system MUST provide Admins with an inspection dashboard displaying the camera snapshot, ML inference metadata, and the user's claimed category side-by-side.


* **DISP-005 (Dispute Approval & Retraining Export):** When an Admin approves a dispute, the system MUST credit base points plus compensatory points (`OriginalPoints + compPoints`), resolve the session, and move the image to the `/retraining-dataset/` directory with the corrected label.


* **DISP-006 (Dispute Rejection):** When an Admin rejects a dispute, the system MUST record a mandatory rejection reason code, credit only the original base points, and close the dispute ticket.



---

### Module 5: Wallet, Ledger & Gamification (WLET)

* **WLET-001 (Triple-Bucket Balance Architecture):** The system MUST aggregate and display user points partitioned into three distinct buckets: `Available`, `Pending/In-Dispute`, and `Lifetime Earned`.


* **WLET-002 (Append-Only Ledger):** The system MUST maintain an immutable, append-only financial ledger tracking transaction UUID, timestamp, bin identifier, item breakdown, points delta, and a running balance snapshot.


* **WLET-003 (Manual Ledger Adjustments):** The system MUST allow Admins to perform manual ledger credits or debits, requiring an explicit, non-empty justification reason code.


* **WLET-004 (Gamification & Leaderboard):** The system MUST compile milestone badges and dynamic leaderboards (weekly, monthly, all-time) based on lifetime earned points and recycling volume.



---

### Module 6: Rewards Marketplace (RWRD)

* **RWRD-001 (Catalog Browsing & Filtering):** The system MUST display an active rewards catalog seeded in the database, allowing Citizens to filter items by category and affordability based on their `Available` balance.


* **RWRD-002 (Immediate Point Deduction):** Upon voucher redemption confirmation, the system MUST immediately deduct the item cost from the Citizen's available points and append a debit entry to the ledger.


* **RWRD-003 (Voucher Issuance):** The system MUST generate an active digital voucher containing a displayable code/barcode/QR stored in the user's "My Vouchers" repository.


* **RWRD-004 (Catalog Administration):** The system MUST allow Admins to create, update, price, and archive reward catalog items.


* **RWRD-005 (Stock Depletion Handling):** The system MUST automatically deactivate or mark rewards out-of-stock when available inventory hits zero.



---

### Module 7: Fleet Management & Administrative Auditing (FLT)

* **FLT-001 (Device Registration & Metadata):** The system MUST allow Admins to register and edit physical bin metadata, including name, geographical coordinates, capacity limits, and district assignments.


* **FLT-002 (Device Operational Telemetry):** The system MUST track live bin fill-levels and operational states (`AVAILABLE`, `≥80% FULL`, `≥95% FULL / MAINTENANCE`).


* **FLT-003 (Public Map Visualization):** The system MUST expose a public read-only endpoint displaying operational bin coordinates and status indicators to mobile users.


* **FLT-004 (Remote Bin Controls):** The system MUST allow Admins to manually transition bins to `OUT_OF_SERVICE` or dispatch simulated reboot commands.


* **FLT-005 (Immutable Admin Audit Logging):** The system MUST record every administrative mutation (dispute settlements, account bans, ledger adjustments, fleet changes) into an immutable audit table capturing Admin ID, action type, resource ID, timestamp, and justification reason.


* **FLT-006 (Operational Analytics):** The system MUST aggregate and display core operational metrics (Daily Active Users, points issued vs. redeemed, recycling volume by material, contamination percentages).



---

## 3. Requirements Traceability Matrix (RTM Summary)

| Module Code | Module Description | Primary Actor | Associated User Journey |
| --- | --- | --- | --- |
| **IAM** | Identity & Access Management | Citizen / Admin | Journey 1 & Journey 5

 |
| **AIE** | Ingestion & AI Classification Engine | Smart Bin (IoT) / Backend | Journey 1

 |
| **CLAIM** | Session Claiming & Point Validation | Citizen | Journey 1

 |
| **DISP** | Dispute Management & Active Learning | Citizen / Admin | Journey 1 & Journey 4

 |
| **WLET** | Wallet, Ledger & Gamification | Citizen / Admin | Journey 2 & Journey 5

 |
| **RWRD** | Rewards Marketplace | Citizen / Admin | Journey 3

 |
| **FLT** | Fleet Management & Auditing | Admin / Citizen (Map) | Journey 5

 |

---

## 4. Architectural Rules & Data Boundaries

1. **FR-RULE-001 (Session TTL Exclusivity):** A disposal session token MUST expire permanently 90 seconds after generation, preventing asynchronous or secondary claims.


2. **FR-RULE-002 (Atomic Financial Mutations):** Every point adjustment, claim, or redemption MUST execute within an ACID database transaction to prevent race conditions and ledger discrepancies.


3. **FR-RULE-003 (Data Isolation):** Citizens MUST NOT query or modify ledgers, dispute tickets, or voucher codes belonging to other users.


4. **FR-RULE-004 (Active Learning Pipeline Integrity):** Captured waste images MUST NOT be transferred into retraining directories without explicit administrative approval of a dispute.


5. **FR-RULE-005 (Audit Trail Non-Repudiation):** Privileged mutations on balances, accounts, and disputes MUST require an explicit reason code and generate an immutable audit log entry.