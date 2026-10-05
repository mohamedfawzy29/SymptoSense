# Module Specification: Emergency Detection & Safety Guardrails (SAFE)

---

## 1. Module Scope & Core Objectives

### 1.1 Module Purpose

The **Emergency Detection & Safety Guardrails (SAFE)** module serves as the primary critical protection layer within SymptoSense. It is responsible for continuous real-time monitoring of user symptom inputs against high-risk medical emergency patterns (Red Flags), executing immediate dynamic workflow overrides, suppressing standard diagnostic inquiry routines, forcing the presentation of urgent emergency alerts, and maintaining strict legal and clinical non-diagnosis boundaries.

### 1.2 Core Principles

- **Safety Primacy (SAFE Overrides All):** Safety evaluation routines (SAFE-_) MUST take absolute precedence over standard conversational questioning flows (AICHAT-_), dynamic symptom extraction, and assessment engine logic.
- **Continuous Transactional Evaluation:** Safety evaluations MUST run synchronously on every single input transaction loop (text, speech-to-text token stream, or 3D coordinate payload) from initial interaction through report compilation.
- **Non-Diagnosis & Framing Integrity:** The system is explicitly bounded to provide clinical urgency triage and safety recommendations. It MUST NEVER present a definitive medical diagnosis, prescribe pharmaceutical interventions, or claim clinical certainty beyond safety triage.
- **Independent Failure Resistance:** The safety detection mechanism MUST operate deterministically and independently of complex generative AI inference or heavy backend LLM processing to ensure critical alerts trigger reliably even under partial system degradation.

---

## 2. Functional Requirements & Business Rules

### 2.1 Functional Requirements Mapping

- **SAFE-001 (Red Flag Symptom Monitoring):** The system MUST continuously and synchronously evaluate all natural language text, speech-to-text input, and 3D anatomical selections against high-risk emergency symptom patterns (e.g., severe crushing chest pain, sudden numbness/paralysis, acute dyspnea, loss of consciousness).
- **SAFE-002 (Immediate Flow Interruption):** Upon detecting a potential medical emergency pattern or Red Flag indicator, the system MUST immediately halt the regular conversational assessment sequence, truncate non-safety dynamic questionnaires, and bypass standard condition evaluation.
- **SAFE-003 (Emergency Alert Display):** The system MUST immediately render an urgent, prominent, non-dismissible medical alert screen advising the user to seek immediate emergency medical care (e.g., calling local emergency services or visiting the nearest emergency department).
- **SAFE-004 (Non-Diagnosis Framing):** The system MUST explicitly frame all non-emergency output results as "possible explanations" or "health considerations" and NEVER present definitive medical diagnoses, prescriptions, or treatment plans.

### 2.2 Mapped Business Rules (BR Enforcement)

- **BR-049 (Safety Evaluation Authority):** Every single assessment transaction MUST undergo strict Safety Evaluation using validated medical rules before any outcome or clarification is rendered. AI models cannot independently decide if a condition is safe.
- **BR-050 (Red Flag Detection Criteria):** Red Flags may be triggered by a single critical symptom, specific symptom combinations, severe intensity thresholds, rapid onset, or specific high-risk health contexts.
- **BR-051 (Safety Has Priority Over Likelihood):** Safety evaluation takes precedence over condition probability ranking. Severe safety concerns must be escalated even if their statistical likelihood is low.
- **BR-052 (Urgency Classification):** The system MUST categorize the assessment outcome into one of two authoritative urgency levels: **Urgent** or **Non-Urgent**.
- **BR-053 (Urgent Safety Escalation):** If a safety rule identifies an Urgent state, the system escalates the urgency level immediately and suppresses non-essential diagnostic questions.
- **BR-054 (Continuous Safety Evaluation):** Safety evaluation is active throughout the entire lifecycle of an assessment session and is not delayed until final questionnaire completion.
- **BR-055 (Safety Override Rule):** Any newly ingested symptom token that increases the assessed urgency level MUST instantly override previous lower urgency classifications.
- **BR-056 (No False Reassurance):** The system is strictly prohibited from making definitive statements reassuring the user that their condition is non-serious when available evidence is incomplete.
- **BR-057 (Separation of Safety Alerts):** Emergency warnings and Safety Alerts MUST be displayed separately from Possible Explanations and do not count toward the 3-possibility display limit.
- **BR-058 (Non-Suppressible Safety Data):** Safety-relevant finding tokens cannot be ignored or filtered out due to low model confidence or delayed input.
- **BR-059 (User-Facing Safety Communication):** The Assessment Engine determines the urgency classification, while the AI layer formats the warning into clear, natural Arabic safety advice.
- **BR-060 (No Diagnosis from Red Flag):** A Red Flag trigger is used solely for safety triage and urgency determination, NOT as a confirmed medical diagnosis.
- **BR-061 (Safety Reassessment Trigger):** Any user modification to a previously answered question triggers an instant safety re-evaluation.
- **BR-062 (Urgency Result Priority Layout):** In any result screen classified as Urgent, the emergency recommendation MUST be positioned at the absolute top of the viewport, above any explanatory text blocks.

---

## 3. Architecture & Technical Flows

### 3.1 Safety Execution Flow & System Interruption Mechanics

```text
                  [ User Input Token Stream ]
                   ( Text / Voice / 3D Payload )
                                │
                                ▼
                 [ SAFE Gateway Interceptor ]
                                │
                  ┌─────────────┴─────────────┐
                  │                           │
          ( Red Flag Detected? )     ( No Emergency Detected )
                  │                           │
         ┌────────┴────────┐                  ▼
       YES                 NO          [ Standard AI & Assessment ]
         │                 │          [   Engine Pipeline Loop   ]
         ▼                 │
[ Halts Evaluation ]       │
[  Instantly (0ms) ]       │
         │                 │
         ▼                 │
[ Render Emergency ]       │
[ Alert Screen View]       │
         │                 │
         ▼                 │
[ Log Audit & Halt ] ──────┴──────────────────┘

```

### 3.2 Detailed Flow Sequences

1. **Synchronous Input Interception Flow:**

- **Trigger:** Every user message payload (text string, STT transcript token, or 3D coordinate vector) reaches the backend gateway.
- **Backend Processing:**
- Before passing the payload to the LLM or Assessment Engine, the **SAFE Interceptor** executes a high-speed pattern match against the validated Red Flag Ruleset DB/Rules Engine.
- Checks for critical keywords, medical combinations, and severe intensity parameters (e.g., severe chest pain, sudden unilateral paralysis, acute respiratory distress).

2. **Immediate Workflow Interruption Flow (Override Protocol):**

- **Trigger:** A Red Flag rule match yields `is_emergency = TRUE`.
- **Backend Processing:**
- Immediately signals execution interruption to the AI Chat Agent and Assessment Engine.
- Suppresses and discards any downstream dynamic questioning tasks currently queued.
- Transitions session evaluation state to `EMERGENCY_INTERRUPTED`.
- Bypasses standard condition evaluation and ranking logic.

3. **Emergency Alert Rendering Flow:**

- **Trigger:** Transition to `EMERGENCY_INTERRUPTED` state.
- **Backend Processing & UI Behavior:**
- Constructs an immutable high-priority emergency payload.
- Renders the non-dismissible **Emergency Alert Screen** (SAFE-003, BR-062).
- Positions emergency instructions (e.g., direct prompt to call emergency services or seek immediate hospital care) at the absolute top of the view.
- Restricts further conversational inputs for this specific symptom pipeline.

4. **Continuous Evaluation & Answer Modification Flow:**

- **Trigger:** User edits a previously submitted answer (BR-015, BR-061).
- **Backend Processing:**
- Immediately invalidates previous safety evaluation states.
- Re-executes the SAFE Interceptor against the newly modified state vector.
- If the revised input triggers a Red Flag, the override protocol executes instantly (BR-055).

---

## 4. Security, Privacy & Data Isolation

### 4.1 Clinical Data Isolation & Emergency Session Handling

- **Non-Diagnostic Framing Enforcement (SAFE-004, BR-068):** Safety outputs MUST NOT write or store definitive medical diagnoses, prescriptions, or formal medical treatments into persistent user records. All alert payloads and logs MUST maintain explicit triage language (e.g., "Urgent medical evaluation recommended", "Possible health explanations").
- **Privacy Boundary for Emergency Payloads (SEC-004):** Emergency alert payloads generated by the SAFE module MUST strictly minimize data inclusion. Personal identifiers (e.g., user name, email, phone number) MUST NOT be attached to emergency alert triggers or client-side telemetry logs.

### 4.2 Independent Failure Resistance & Availability

- **High-Availability Standalone Safety Ruleset (REL-004):** Emergency detection logic MUST function independently of generative AI services, external LLM APIs, and complex assessment pipelines. The safety rules engine MUST remain accessible and operational even under partial backend degradation or LLM API downtime (ERR-001).
- **Data in Transit Security (SEC-001):** All safety evaluation requests and emergency payload delivery MUST strictly enforce TLS 1.3 encryption over HTTPS/WSS.

### 4.3 Non-Suppressible Safety Audit Logging

- **Audit Logging for Safety Compliance (BR-058):** Every Red Flag match, emergency workflow override, and user safety alert trigger MUST create an immutable, timestamped audit log entry.
- **Data Anonymization for Auditing (REVI-001):** Safety audit logs made available for clinical review or platform safety compliance MUST be completely stripped of Personally Identifiable Information (PII) before being surfaced in administrative or medical reviewer interfaces.

---

## 5. Edge Cases & Fail-safes

### 5.1 High-Risk Clinical & Timeline Edge Cases

- **ERR-007: Late Ingestion of Red Flag Symptoms:** A user inputs a high-risk emergency indicator (e.g., severe crushing chest pain, sudden numbness, acute dyspnea) late into a conversation that was previously evaluated as a low-urgency track.
- _System Behavior:_ The SAFE interceptor executes an immediate safety override the exact millisecond the Red Flag pattern is recognized. It truncates all downstream questionnaire scripts, halts the dynamic AI conversation, and forces the display of the Emergency Alert Screen instructing the user to seek immediate emergency care (BR-054, BR-055).

- **ERR-008: Symptom Cluster Overload with Safety Priority:** A user provides a multi-symptom description resolving to more than 6 distinct clinical symptom clusters within a single session.
- _System Behavior:_ The system applies the strict 6-cluster boundary. The Assessment Engine and SAFE module enforce priority filters, evaluating Red Flag/Safety rules first and Urgency levels second. Low-priority non-safety clusters are deferred to subsequent sessions while critical safety evaluations proceed without delay (BR-046).

### 5.2 Technical Failure & Infrastructure Edge Cases

- **ERR-001: AI Service Outage / Streaming Timeout During Assessment:** The primary LLM service experiences downtime or exceeds the 2.5-second streaming threshold while evaluating user inputs.
- _System Behavior:_ The standalone SAFE interceptor executes locally on the backend gateway, independent of the LLM state (REL-004). If a Red Flag is detected in the raw input tokens, the Emergency Alert Screen is rendered immediately despite the AI outage, ensuring user safety is never compromised by external API failures.

- **Conflicting User Timeline Contexts with Potential Red Flags (ERR-004):** A user submits contradictory timeline data (e.g., "pain started 2 days ago" vs. "pain has been there for 2 weeks") alongside severe symptom intensity.
- _System Behavior:_ If the severe intensity threshold triggers a Red Flag rule, the SAFE module immediately escalates the session to Urgent status, ignoring non-critical timeline contradictions to prioritize user safety over clarification (BR-051, BR-053).

### 5.3 Ambiguity & False Reassurance Edge Cases

- **ERR-006: Highly Ambiguous Input with High-Risk Keywords:** The user enters an ambiguous colloquial phrase containing high-risk terms (e.g., "حاسس بحاجة تقيلة على صدري مش عارف أتنفس").
- _System Behavior:_ The system does NOT assign a benign assumed value or guess. The SAFE interceptor flags the emergency keyword combination and immediately triggers the Urgent emergency triage pathway rather than risking false reassurance (BR-022, BR-056).

---

## 6. Acceptance Criteria & Test Scenarios

### 6.1 Formal Acceptance Criteria (Given-When-Then Format)

- **AC-SAFE-001: Continuous Red Flag Processing & Immediate Flow Interruption**
- **Given:** A user (Guest or Registered) is actively engaged in an incomplete conversational assessment stream.
- **When:** The user inputs text, voice transcript, or 3D coordinate data matching a recognized high-risk emergency pattern (e.g., severe crushing chest pain, sudden unilateral numbness, acute dyspnea) (SAFE-001).
- **Then:** The Safety Evaluation layer MUST execute an immediate high-priority system override at the exact millisecond of pattern recognition (SAFE-002, BR-054).
- **And:** The system MUST instantly truncate all downstream diagnostic inquiry scripts, halt non-safety questioning loops, and suppress standard condition probability evaluations (SAFE-002, BR-053).

- **AC-SAFE-002: Emergency Alert Screen Rendering & View Priority**
- **Given:** The Safety Evaluation layer has identified a Red Flag pattern and triggered a system override.
- **When:** The system transitions the active session evaluation state to `EMERGENCY_INTERRUPTED`.
- **Then:** The client interface MUST immediately render a prominent, non-dismissible Emergency Alert Screen advising the user to seek immediate emergency medical care (SAFE-003).
- **And:** The emergency instructions and local emergency contact buttons MUST be positioned at the absolute top of the viewport layout, above any secondary explanatory content (BR-062).

- **AC-SAFE-003: Non-Diagnosis Framing & Disclaimer Enforcement**
- **Given:** The SAFE module processes any non-emergency or urgent assessment outcome payload.
- **When:** The assessment report data is compiled and rendered for display.
- **Then:** All presented potential explanations MUST be explicitly framed using non-definitive clinical triage terms (e.g., "Possible Explanations", "May be consistent with") (SAFE-004, BR-068).
- **And:** The system MUST strictly prohibit displaying definitive diagnoses, pharmaceutical prescriptions, or guarantees of medical certainty (SAFE-004, BR-056).

- **AC-SAFE-004: Safety Priority Over Condition Probability**
- **Given:** An active assessment session contains multiple symptom inputs where a high-severity potential condition has low statistical likelihood compared to a benign condition.
- **When:** The SAFE module evaluates the combined context vector.
- **Then:** The system MUST prioritize the safety risk escalation and classify the urgency level as Urgent (BR-051, BR-052).
- **And:** The safety alert warning MUST be rendered independently of and prioritized above statistical likelihood rankings (BR-051, BR-057).

- **AC-SAFE-005: Instant Safety Re-evaluation on Answer Revision**
- **Given:** A user is reviewing their input history within an active assessment session.
- **When:** The user modifies or updates a previously submitted symptom answer to include a critical severity parameter (BR-015, BR-061).
- **Then:** The SAFE module MUST instantly invalidate the previous assessment state and re-execute the safety interceptor pipeline (BR-061).
- **And:** If the updated value matches a Red Flag pattern, the system MUST execute an immediate urgency escalation and override lower previous classifications (BR-055).

---

## 7. API Endpoints & Interface Contracts

#### Endpoint 1: Evaluate Real-Time Safety Interception (Synchronous Pre-Check)

- **Route:** `POST /api/v1/safety/evaluate`
- **Access Level:** Internal Gateway / Public Interceptor (Session Token or JWT)
- **Description:** Evaluates incoming user text, speech tokens, or 3D coordinate selections against active Red Flag rules _before_ passing execution to generative AI models (SAFE-001, SAFE-002).

**Request Headers:**

```http
Content-Type: application/json
Authorization: Bearer <JWT_ACCESS_TOKEN>  # Or X-Guest-Session-ID

```

**Request Body:**

```json
{
  "assessment_id": "7c8b9a0f-1e2d-3c4b-5a6f-7e8d9c0b1a2f",
  "input_type": "TEXT",
  "user_input_text": "بقالي ساعة حاسس بوجع شديد زي السكينة في نص صدري ومشه قادر أتنفس",
  "anatomical_selection": {
    "anatomical_code": "CHEST_CENTER",
    "body_side": "Anterior",
    "major_region": "Torso"
  },
  "current_severity": 9
}
```

**Response 200 OK (Emergency Red Flag Triggered — Immediate Interruption):**

```json
{
  "status": "success",
  "data": {
    "evaluation_id": "e1f2a3b4-c5d6-7e8f-9a0b-1c2d3e4f5a6b",
    "is_emergency_override": true,
    "urgency_level": "URGENT",
    "action_required": "HALT_ASSESSMENT_SHOW_EMERGENCY_ALERT",
    "safety_alert": {
      "title_ar": "تنبيه طبي عاجل — حماية سلامتك هي الأولوية",
      "message_ar": "بناءً على الأعراض المذكورة (ألم حاد بالصدر مع صعوبة في التنفس)، يتطلب وضعك الصحي تقييماً طبياً فورياً في الطوارئ.",
      "recommended_action_ar": "يرجى التوجه فوراً إلى أقرب مستشفى أو الاتصال بإسعاف الطوارئ (123).",
      "emergency_contacts": [{ "name_ar": "الإسعاف المصري", "number": "123" }],
      "display_position": "TOP_OF_PAGE"
    },
    "triggered_rule": {
      "rule_code": "RF_CHEST_PAIN_ACUTE",
      "category": "CARDIOVASCULAR"
    }
  }
}
```

**Response 200 OK (Non-Emergency — Safe to Proceed):**

```json
{
  "status": "success",
  "data": {
    "evaluation_id": "f2e1d0c9-b8a7-6f5e-4d3c-2b1a0f9e8d7c",
    "is_emergency_override": false,
    "urgency_level": "NON_URGENT",
    "action_required": "PROCEED_TO_ASSESSMENT_ENGINE"
  }
}
```

#### Endpoint 2: Get Active Session Emergency Status & Urgency Triage Layout

- **Route:** `GET /api/v1/safety/status/:assessment_id`
- **Access Level:** Authenticated / Active Guest Session
- **Description:** Retrieves authoritative urgency classification and non-diagnosis framing layout specifications (SAFE-003, SAFE-004, BR-062).

**Request Headers:**

```http
Authorization: Bearer <JWT_ACCESS_TOKEN>

```

**Response 200 OK:**

```json
{
  "status": "success",
  "data": {
    "assessment_id": "7c8b9a0f-1e2d-3c4b-5a6f-7e8d9c0b1a2f",
    "urgency_level": "URGENT",
    "is_emergency_override": true,
    "framing_disclaimer_ar": "جميع المخرجات هي احتمالات ومعلومات استرشادية للتقييم الأولي وليست تشخيصاً طبياً نهائياً.",
    "layout_rules": {
      "emergency_alert_position": "TOP_PRIORITY",
      "suppress_dynamic_questions": true,
      "allow_answer_revision": true
    }
  }
}
```

#### Endpoint 3: Safety Audit Event Log (Internal Clinical Triage Compliance)

- **Route:** `POST /api/v1/safety/audit-log`
- **Access Level:** Internal Service / System Admin
- **Description:** Persists immutable safety audit events with strict PII anonymization for medical quality governance (BR-058, REVI-001).

**Request Body:**

```json
{
  "evaluation_id": "e1f2a3b4-c5d6-7e8f-9a0b-1c2d3e4f5a6b",
  "assessment_id": "7c8b9a0f-1e2d-3c4b-5a6f-7e8d9c0b1a2f",
  "rule_code": "RF_CHEST_PAIN_ACUTE",
  "anonymized_symptom_summary": "Acute chest pain with dyspnea",
  "urgency_assigned": "URGENT",
  "override_executed": true
}
```

**Response 201 Created:**

```json
{
  "status": "success",
  "message": "Safety audit record created securely with PII anonymized."
}
```
