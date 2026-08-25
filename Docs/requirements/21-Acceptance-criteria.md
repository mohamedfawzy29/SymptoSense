# Acceptance Criteria Document (ACD)

**Document Version:** 1.1  
**Status:** Approved  
**Last Updated:** Aug 25, 2026  

---

## 1. Module 1: Guest & Authentication Management (AUTH)

### AC-AUTH-001 (Anonymous Initial Assessment Flow)
* **Scenario:** A user initiates an evaluation session without creating an account.
* **Given:** A user navigates to the SymptoSense application and is unauthenticated.
* **When:** The user clicks the "Start Assessment" button.
* **Then:** The backend system MUST automatically generate a unique, temporary Guest Session ID stored via the client browser (`AUTH-001`).
* **And:** The system MUST allow the user to complete the entire symptom assessment questionnaire and view the final report results without forcing user registration (`AUTH-002`, `BR-002`).
* **And:** The system MUST strictly enforce that no personal identifiers (Name, Email, Phone Number) are required to complete this initial session (`BR-003`).

### AC-AUTH-002 (Guest-to-User Account Migration)
* **Scenario:** A guest user registers an account immediately after completing an assessment.
* **Given:** A user has completed a guest assessment session and is viewing the final report.
* **When:** The user clicks the registration prompt and successfully submits the Email/Password sign-up form.
* **Then:** The backend system MUST execute an atomic database operation that links the temporary Guest Session ID and its associated assessment report to the newly created permanent User ID (`AUTH-005`, `BR-005`).
* **And:** The system MUST verify that the migrated report is accessible within the user's persistent Assessment History dashboard (`BR-005`, `BR-090`).

### AC-AUTH-003 (Guest Assessment Session Limit)
* **Scenario:** A guest user attempts to start a second assessment within the same unauthenticated session.
* **Given:** A guest user has already completed one single assessment and generated a report.
* **When:** The user attempts to initiate a new, separate symptom check.
* **Then:** The system MUST block the execution loop and present a clear authentication enforcement prompt requiring the user to login or sign up before proceeding (`BR-006`, `BR-009`).

---

## 2. Module 2: Health Context & Profile (HCTX)

### AC-HCTX-001 (Mandatory and Optional Health Context Ingestion)
* **Scenario:** Form validation rules for demographic context collection.
* **Given:** A user (Guest or Registered) initiates a new assessment session.
* **When:** The system displays the initial Health Context form layout.
* **Then:** The system MUST enforce mandatory entry of **Age** and **Gender**, blocking forward progress if these fields are missing or invalid (`HCTX-001`, `BR-011`).
* **And:** The interface MUST explicitly format Chronic Conditions, Current Medications, and Allergies as optional secondary entry points, allowing the user to skip them if desired (`HCTX-002`, `BR-024`).

### AC-HCTX-002 (Context Injection and Auto-Fill)
* **Scenario:** Streaming saved data to personalize new assessments.
* **Given:** A logged-in Registered User has a saved persistent Health Profile inside the database.
* **When:** The user starts a new assessment session (`BR-007`).
* **Then:** The system MUST automatically fetch and auto-populate the form layout with their saved age, gender, and chronic context parameters (`HCTX-003`).
* **And:** The system MUST inject this complete contextual payload directly into the active Assessment Engine pipeline to personalize follow-up question loops (`HCTX-004`).

---

## 3. Module 3: 3D Body Interaction & Localization (3DLOC)

### AC-3DLOC-001 (Multi-Level Anatomical Progressive Selection)
* **Scenario:** Navigating the 3D human body model to locate symptoms.
* **Given:** The user initializes the anatomical localization screen layout.
* **When:** The user interacts with the rendered 3D human body mesh component.
* **Then:** The system MUST enforce a 4-level progressive selection sequence: `Whole Body` $\rightarrow$ `Major Body Region` $\rightarrow$ `Sub-Region` $\rightarrow$ `Specific Location` (`3DLOC-002`, `Section 2`).
* **And:** The camera camera view MUST smoothly rotate and zoom into the targeted clicked mesh within 1.5 seconds, maintaining a rendering speed of $\ge$ 30 FPS (`3D-NAV-002`, `PERF-002`).
* **And:** The interface MUST display a responsive Arabic breadcrumb bar (e.g., `الجسم بالكامل` ← `الجذع` ← `الصدر`) allowing navigation back to parent anatomical regions (`3D-NAV-003`, `3DLOC-004`).

### AC-3DLOC-002 (WebGL Failure and Direct Chat Mode Transition)
* **Scenario:** System fallback behavior when WebGL fails to render.
* **Given:** A user lands on the symptom selection entry point screen.
* **When:** The client hardware or browser fails to load the 3D model assets, or WebGL rendering times out past 1.5 seconds (`ERR-003`).
* **Then:** The system MUST intercept the failure loop, seamlessly hide the 3D viewport canvas element, and immediately expand the Direct Conversational Chat Mode interface (Text/Voice Input box) as the primary localized interactive pathway without disrupting the user journey (`3D-PERF-003`, `ERR-003`).

---

## 4. Module 4: Conversational AI & Dynamic Questioning (AICHAT)

### AC-AICHAT-001 (Dialect Comprehension and Streaming Latency)
* **Scenario:** Processing natural language inputs and streaming follow-ups.
* **Given:** A user inputs a free-form text or voice description using colloquial Egyptian Arabic (e.g., *"بقالي يومين حاسس بوجع في بطني ناحية اليمين ويزيد لما أتحرك"*).
* **When:** The text payload is submitted to the system layer.
* **Then:** The AI layer MUST accurately extract the clinical parameters into a standardized structured JSON format, map them to the anatomical taxonomy, and pass them to the Assessment Engine (`AICHAT-002`, `BR-031`).
* **And:** The conversational interface MUST begin streaming or rendering the next dynamically generated, medically relevant question within 2.5 seconds of user submission (`PERF-003`, `BR-019`).

### AC-AICHAT-002 (Answer Revision Integration)
* **Scenario:** Re-evaluating the assessment path when previous answers change.
* **Given:** A user is in an active, incomplete conversational assessment session (`BR-012`).
* **When:** The user navigates back and modifies a previously submitted answer (`AICHAT-004`, `BR-015`).
* **Then:** The Assessment Engine MUST instantly clear any obsolete required downstream fields, update the active context data payload, and dynamically recompute subsequent questions based on the new input parameter (`BR-015`, `BR-028`).

---

## 5. Module 5: Emergency Detection & Safety Guardrails (SAFE)

### AC-SAFE-001 (Continuous Red Flag Processing and Immediate Overrides)
* **Scenario:** Intercepting high-risk emergency symptom patterns.
* **Given:** A user is interacting with an active, incomplete conversational questionnaire stream.
* **When:** The user enters text or voice indicating a high-risk red flag pattern (e.g., crushing chest pain or sudden acute dyspnea) (`SAFE-001`).
* **Then:** The Safety Evaluation layer MUST immediately execute a high-priority system override at the exact millisecond of token recognition, halting the standard evaluation pipeline instantly (`SAFE-002`, `BR-054`).
* **And:** The system MUST truncate all non-safety related diagnostic inquiries, prevent the presentation of typical condition possibilities, and render the urgent Emergency Alert Screen instructing the user to seek immediate emergency care (`SAFE-002`, `SAFE-003`, `BR-053`).

---

## 6. Module 6: Assessment Report & Specialty Guidance (RPT)

### AC-RPT-001 (Report Layout Structural Constraints)
* **Scenario:** Validating final compiled report components.
* **Given:** The Assessment Engine determines that minimum sufficient data has been collected to reach a completed state (`BR-013`).
* **When:** The final structured Assessment Report is generated and compiled for the user interface layout view.
* **Then:** The rendered report MUST strictly enforce the following layout boundaries:
  1. Display a prominent, verified Urgency Level indicator (Urgent or Non-Urgent) (`RPT-001`, `BR-052`).
  2. Present a maximum of three supported possible explanations ranked by clinical context priority (`BR-048`, `BR-066`).
  3. Include the exact mapped clinical Medical Specialty recommendation (`RPT-002`).
  4. Inject prominent medical disclaimers framing outputs clearly as "possible explanations" and never as definitive medical diagnoses (`SAFE-004`, `BR-068`).
* **And:** If the final Urgency status is evaluated as "Urgent", the system MUST reposition the recommended action layout to display at the absolute top of the view, *above* any potential explanatory text blocks (`BR-062`).

### AC-RPT-002 (Cross-User Isolation Enforcement)
* **Scenario:** Access control checks on historical report queries.
* **Given:** Two distinct authenticated users exist in the system (User A with Report ID 100, and User B with Report ID 200).
* **When:** User A attempts to manually trigger or spoof an API request querying Report ID 200.
* **Then:** The backend API security layer MUST apply strict Role-Based Access Control filters matching the active authenticated token ID, block the transaction immediately, return an HTTP 403 Forbidden status code, and hide all data payload records from the unauthorized screen view (`FR-RULE-001`, `SEC-005`, `BR-091`).
