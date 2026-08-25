# User Journeys & System Flows

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-19  

---

## 1. Overview & Purpose

This document outlines the core user journeys for **SymptoSense**. The goal of defining these journeys before technical implementation is to:

1. **Map System Behavior to User Intent:** Clearly visualize screen flow, backend interactions, and state persistence for both unauthenticated and authenticated users.
2. **Ensure Smooth Guest-to-User Transition:** Guarantee that guest users can receive immediate value without losing their initial assessment data when registering.
3. **Embed Safety Mechanics:** Explicitly define where emergency detection logic interrupts normal workflows to prioritize user safety.

---



## 2. Core User Journeys

---

### Journey 1: Guest User Assessment Flow
> **Primary Goal:** Provide immediate, high-value symptom assessment without requiring upfront registration.
>
> [Start Assessment]
│
▼
[Initial Health Context Form]
│
▼
[3D Body Symptom Localization]
│
▼
[Conversational AI Assessment] ──(Red Flag Detected?)──► [Emergency Alert Screen]
│ (No Emergency)
▼
[Symptom Assessment Report]
│
▼
[Prompt: Register to Save History]




> #### Detailed Steps:
1. **Entry & Session Initialization:**
   * User navigates to the application and clicks **"Start Assessment"**.
   * The system generates a temporary `Guest Session ID` (stored in `localStorage` / session cookie).
2. **Initial Health Context Input:**
   * User provides basic mandatory demographics: **Age** and **Gender**.
   * User optionally enters: **Chronic Conditions**, **Current Medications**, and **Allergies**.
3. **Anatomical Symptom Localization:**
   * User opens the interactive **3D Human Body Model**.
   * Progressive navigation path: `Whole Body` $\rightarrow$ `Region` $\rightarrow$ `Sub-region` $\rightarrow$ `Specific Location`.
   * *Alternative:* User describes symptom via natural language text or voice input.
4. **Conversational AI & Dynamic Follow-ups:**
   * User describes symptoms in natural language (Arabic / Egyptian Arabic) or via voice.
   * AI processes context and asks dynamic, medically relevant follow-up questions.
   * **Safety Guardrail Check:** If emergency criteria (Red Flags) are met at any point, the flow halts immediately, directing the user to the **Emergency Alert Screen**.
5. **Report Generation:**
   * System generates a structured assessment report containing:
     * Possible health conditions.
     * Urgency classification (Low, Moderate, High).
     * Recommended medical specialty.
     * Medical disclaimers.
6. **Registration Trigger:**
   * A call-to-action prompts the guest user to create an account to permanently save the report to their profile.

---



### Journey 2: Registered User Assessment & Profile Flow
> **Primary Goal:** Maintain persistent health history, speed up new assessments using saved context, and review past reports.

[Login / Account Creation]
│
▼
[User Dashboard]
├──► [View Assessment History / Past Reports]
└──► [Start New Assessment]
│
▼
[Auto-fill Saved Health Profile]
│
▼
[3D Body & AI Assessment Flow]
│
▼
[Save Report to Account History]




#### Detailed Steps:
1. **Authentication / Registration Migration:**
   * User completes registration after a guest assessment (system links the `Guest Session ID` to the new `User ID`).
   * Returning user logs in with credentials.
2. **Dashboard Access:**
   * User views their personal **Dashboard** with two main capabilities:
     * **Assessment History:** Browse, search, and view previous assessment reports.
     * **Health Profile Management:** Update saved chronic conditions, medications, age, and basic health metrics.
3. **New Assessment Execution:**
   * User clicks **"Start New Assessment"**.
   * System automatically loads the pre-existing **Health Context** to streamline the assessment.
   * User follows the standard 3D localization and AI conversation steps.
4. **Persistent Report Storage:**
   * Upon completion, the new assessment report is linked to the user's permanent account history.

---



### Journey 3: Admin Operational Flow
> **Primary Goal:** Platform management, system monitoring, and operational maintenance.

[Admin Login] ──► [Admin Dashboard] ──► [User Management & System Logs]


#### Detailed Steps:
1. **Authentication:**
   * Admin logs in via a secure, separate administration portal.
2. **User & System Administration:**
   * View user account statuses and non-sensitive operational metrics.
   * Monitor system logs, error reports, and assessment session diagnostics without violating user medical data isolation.

---



## 3. Data & State Transition Rules

| Step / Action | Guest State | Registered State | System Action |
| :--- | :--- | :--- | :--- |
| **Start Assessment** | Creates `Guest Session ID` | Uses active JWT Token | Allocates temporary vs. authenticated session state |
| **Health Context Entry** | Stored in temporary memory | Fetched from database | Auto-populates forms for registered users |
| **Emergency Trigger** | Display emergency guidance | Display emergency guidance | Immediately terminates assessment flow |
| **Complete Assessment** | Shows report; temporary access | Saves report to DB | Prompts guest to register |
| **Account Registration** | Converts session to User ID | N/A | Migrates guest assessment record to permanent DB record |

---

## 4. Architectural Rules for Implementation

* **ROLE-001 (Guest Flow):** Guest users MUST be able to reach the Assessment Report without mandatory registration.
* **ROLE-002 (Data Isolation):** Authenticated users MUST only query assessment history matching their authenticated `User ID`.
* **ROLE-003 (Emergency Guardrail):** Emergency detection logic MUST take precedence over standard dynamic follow-up questioning.
* **ROLE-004 (Session Migration):** The backend MUST provide an atomic endpoint to migrate guest assessment records upon user sign-up.




Markdown
---

## Journey 4: Medical Reviewer / Clinical Auditor Flow (Future Design)

### Primary Actor
* **Medical Reviewer / Clinical Specialist** (طبيب / مراجع طبي معتمد)

### Core Objective
Audit anonymized assessment logs, validate AI specialty mapping accuracy, flag unsafe or inaccurate clinical prompts, and maintain clinical safety standards without compromising user privacy.

---

### Step-by-Step Flow

[Login via Medical Portal] ──> [Access Clinical Dashboard]
│
▼
[Select Anonymized Assessment]
│
▼
[Review AI Reasoning & Outputs]
│
┌──────────────────────┴──────────────────────┐
▼                                             ▼
[Approved / Accurate]                      [Flagged / Incorrect Mapping]
│                                             │
▼                                             ▼
[Mark as Clinically Valid]                   [Add Clinical Feedback & Correct Taxonomy]


### Detailed Interaction Table

| Step | User Action (Medical Reviewer) | System Response | Data State & Safety Rules |
| :--- | :--- | :--- | :--- |
| **1. Authentication** | Log in using clinical staff credentials with MFA (Multi-Factor Auth). | Validates credentials and grants access to Medical Reviewer Portal. | Enforces strictly defined Clinical Reviewer RBAC permissions. |
| **2. Queue Filtering** | Filter assessments by criteria: Emergency Flagged, Low AI Confidence, or Random Sample. | Displays a list of fully anonymized assessment transcripts and generated reports. | **Privacy Boundary:** All PII (Name, Email, Phone, IP) MUST be redacted. |
| **3. Clinical Audit** | Inspect inputs (3D anatomical points, symptoms, demographics) and AI outputs (Conditions, Urgency, Specialty). | Shows raw AI prompt logs, symptom extraction breakdown, and final output report. | Read-only view of health parameters. No edit rights over live user data. |
| **4. Feedback & Calibration** | Rate accuracy (1-5) and provide override corrections if specialty or urgency was misclassified. | Saves feedback log directly to the AI Prompt Calibration Dataset for developer review. | Creates an immutable Audit Trail entry linked to `Reviewer_ID`. |

---

### Key Requirements for Medical Reviewer:
* **REVI-001 (Anonymized Log Access):** The system MUST strip all Personally Identifiable Information (PII) before displaying transcripts to the Medical Reviewer.
* **REVI-002 (Prompt Feedback Loop):** The Reviewer MUST be able to submit structured clinical corrections (e.g., correct medical specialty, severity level adjustment) for AI prompt tuning.
* **REVI-003 (Safety Audit Logs):** All emergency red-flag triggers MUST be accessible in a priority queue for clinical safety compliance audits.
