Markdown
# Functional Requirements Document (FRD)

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-19  

---

## 1. Overview & Scope

This document specifies the Functional Requirements for the Minimum Viable Product (MVP) of **SymptoSense**. All functional requirements herein are strictly derived from the approved Product Scope, Stakeholder Analysis, User Roles, and User Journeys.

Each requirement follows the format:
`[Module Code]-[Sequential Number]: Requirement Description`

---

## 2. System Functional Modules

---

### Module 1: Guest & Authentication Management (AUTH)

* **AUTH-001 (Guest Session Creation):** The system MUST generate a temporary, unique Guest Session ID upon starting an unauthenticated assessment.
* **AUTH-002 (Guest Assessment Capability):** The system MUST allow Guest Users to complete a full symptom assessment and view the final report without forcing registration.
* **AUTH-003 (User Registration):** The system MUST allow Guest Users to register an account using Email/Password after completing an assessment or from the main interface.
* **AUTH-004 (User Authentication):** The system MUST authenticate Registered Users securely and issue JWT-based access tokens.
* **AUTH-005 (Guest Session Migration):** Upon registration or login immediately following a guest assessment, the system MUST automatically associate the temporary guest assessment report with the new user's permanent account.
* **AUTH-006 (Password Management):** The system MUST provide standard user credential management (login, logout, password reset flow).

---

### Module 2: Health Context & Profile (HCTX)

* **HCTX-001 (Initial Context Collection):** The system MUST collect mandatory demographic inputs (Age, Gender) prior to or during the symptom assessment.
* **HCTX-002 (Optional Context Inputs):** The system MUST support optional entry of height, weight, chronic diseases, current medications, allergies, and pregnancy status (when applicable).
* **HCTX-003 (Persistent Health Profile):** The system MUST allow Registered Users to save, view, update, and reuse their Health Profile across future assessments.
* **HCTX-004 (Context Injection):** The system MUST pass the user's saved Health Context to the AI Assessment Engine to personalize follow-up questions and condition evaluations.

---

### Module 3: 3D Body Interaction & Localization (3DLOC)

* **3DLOC-001 (Interactive 3D Body Representation):** The system MUST render an interactive 3D human body model as a primary entry point for symptom selection.
* **3DLOC-002 (Progressive Anatomical Navigation):** The system MUST support multi-level progressive selection:
  `Whole Body` $\rightarrow$ `Body Region` $\rightarrow$ `Sub-Region` $\rightarrow$ `Specific Location`.
* **3DLOC-003 (Localization Payload):** The system MUST convert the selected 3D anatomical coordinates/region into a standardized text context payload for the AI engine.
* **3DLOC-004 (RTL & Labeling Support):** All 3D anatomical body region labels MUST be presented in Arabic.

---

### Module 4: Conversational AI & Dynamic Questioning (AICHAT)

* **AICHAT-001 (Multi-Modal Symptom Input):** The system MUST allow users to describe symptoms using natural language text and voice input (Speech-to-Text).
* **AICHAT-002 (Arabic / Egyptian Dialect Processing):** The AI Engine MUST comprehend descriptions written or spoken in standard Arabic and Egyptian Arabic.
* **AICHAT-003 (Dynamic Follow-up Generation):** The AI Engine MUST dynamically generate relevant follow-up questions based on the user's previous answers, health context, and localized body region.
* **AICHAT-004 (Answer Revision):** The system MUST allow users to review and modify their answers prior to finalizing the assessment.

---

### Module 5: Emergency Detection & Safety Guardrails (SAFE)

* **SAFE-001 (Red Flag Symptom Monitoring):** The system MUST continuously evaluate user inputs against high-risk emergency symptom patterns (e.g., severe chest pain, sudden numbness, acute dyspnea).
* **SAFE-002 (Immediate Flow Interruption):** Upon detecting a potential medical emergency, the system MUST immediately halt the regular assessment sequence.
* **SAFE-003 (Emergency Alert Display):** The system MUST display an urgent, prominent medical warning advising the user to seek immediate emergency care.
* **SAFE-004 (Non-Diagnosis Framing):** The system MUST explicitly frame all non-emergency outputs as "possible conditions" and NEVER present definitive medical diagnoses or prescriptions.

---

### Module 6: Assessment Report & Specialty Guidance (RPT)

* **RPT-001 (Report Generation):** Upon assessment completion, the system MUST generate a structured report including:
  * Summary of reported symptoms.
  * Possible health explanations.
  * Urgency level indicator (Urgent, Non-Urgent).
  * Recommended medical specialty (e.g., Cardiology, ENT, Neurology).
  * Prominent medical disclaimers.
* **RPT-002 (Specialty Mapping):** The AI Engine MUST accurately map the evaluated symptom cluster to the most appropriate clinical specialty.
* **RPT-003 (Assessment History Storage):** The system MUST save completed assessment reports to the database for Registered Users.
* **RPT-004 (Assessment History Retrieval):** Registered Users MUST be able to view a chronological list of their past assessments and inspect detailed past reports.

---

### Module 7: Administrative & Operational Capabilities (ADM)

* **ADM-001 (User Management):** The Admin MUST be able to search, view, activate, and deactivate registered user accounts.
* **ADM-002 (System Activity Monitoring):** The Admin MUST be able to view aggregated platform usage statistics (e.g., total assessments completed, registration rates) without viewing private user chat logs.
* **ADM-003 (Operational Error Logging):** The system MUST log technical failures, API downtime, and system errors for administrative review.

---

## 3. Requirements Traceability Matrix (RTM Summary)

| Module | Primary Actor | Associated User Journey |
| :--- | :--- | :--- |
| **AUTH** | Guest User / Registered User | Journey 1 & 2 |
| **HCTX** | Guest User / Registered User | Journey 1 & 2 |
| **3DLOC** | Guest User / Registered User | Journey 1 & 2 |
| **AICHAT** | Guest User / Registered User | Journey 1 & 2 |
| **SAFE** | All Users | Journey 1 & 2 (Guardrail) |
| **RPT** | Guest User / Registered User | Journey 1 & 2 |
| **ADM** | Admin | Journey 3 |

---

## 4. Architectural Rules & Data Boundaries

1. **FR-RULE-001 (Data Isolation):** Users MUST NOT be able to access assessment reports belonging to other users.
2. **FR-RULE-002 (Guest Data Expiration):** Guest assessment sessions that are not migrated to a user account within 24 hours SHOULD be automatically purged or anonymized.
3. **FR-RULE-003 (Safety Primacy):** Safety rules (`SAFE-*`) override all conversational flows (`AICHAT-*`).
