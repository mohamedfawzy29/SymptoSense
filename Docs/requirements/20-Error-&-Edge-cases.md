# Error & Edge Cases Requirements

**Document Version:** 1.1  
**Status:** Approved  
**Last Updated:** Aug 25, 2026  

---

## 1. Purpose
This document defines the systemic behavior, fail-safes, and boundary constraints for handling technical, operational, and clinical edge cases within SymptoSense. Every error-handling routine specified herein is strictly derived from the approved Business Rules (BR-*), Functional Requirements (FR-*), and Non-Functional Requirements (SEC-*, REL-*). The absolute mandate of this system is that no technical or data-level failure shall compromise user safety or result in an independent AI diagnosis.

---

## 2. Technical & Infrastructure Edge Cases

### ERR-001: AI Service Outage and Timeout Handling
* **Condition:** The selected Open-Weight AI model infrastructure (to be finalized post-evaluation from candidates: Qwen, Llama, or MedGemma) becomes unavailable, times out beyond the maximum streaming latency threshold of 2.5 seconds, or encounters a platform API downtime event.
* **System Behavior:**
  1. The backend system MUST prevent user session degradation and maintain all active reported symptoms within temporary memory without crash or data loss.
  2. The infrastructure MUST initiate one immediate background retry to stabilize the connection.
  3. If the connection cannot be recovered, the platform MUST gracefully invoke its defined failure fallback mechanism, preventing any system fabrication or guesswork of results.
  4. The application interface MUST render a localized, user-friendly error response to the user:
     *"نواجه صعوبة مؤقتة في معالجة البيانات، لم نفقد أيًا من أعراضك المسجلة. يمكنك المحاولة مرة أخرى خلال ثوانٍ."*

### ERR-002: Invalid Structured AI Output Data Compliance
* **Condition:** The AI layer generates corrupted structured schema outputs, invalid fields, or values that violate formatting validation bounds (e.g., extracting an invalid pain severity rating of 15 instead of the defined 0–10 scale).
* **System Behavior:**
  1. The platform's API validation layer MUST block the corrupted structured data from reaching the authoritative Assessment Engine.
  2. The system MUST reject the schema payload, flag it as an incomplete structural state, and prevent the ingestion of inconsistent AI data outputs.
  3. The system MUST trigger an active clarification or retry execution mechanism to re-acquire valid data from the conversational layer without selecting an assumed value.

### ERR-003: WebGL Rendering Failure & Interface Mode Transition
* **Condition:** The client browser or smartphone hardware fails to load or render the binary `.gltf` / `.glb` interactive 3D human body assets within the mandatory 1.5-second runtime limit.
* **System Behavior:**
  1. The UI controller MUST intercept the rendering failure without disrupting the user session or throwing a breaking application error.
  2. Because the platform natively supports a Hybrid Input Model, the system MUST gracefully hide the 3D viewport canvas.
  3. The interface MUST automatically expand and pivot the view to the direct Conversational Chat Mode (Text/Voice Input Interface) as the primary alternative input pathway, allowing the user to describe their symptoms naturally in Arabic immediately without interruption.

---

## 3. Conversational & Input Edge Cases

### ERR-004: Ingestion of Conflicting User Temporal/Severity Contexts
* **Condition:** A user enters explicitly contradictory symptom timeline data within a single continuous assessment session (e.g., stating *"بقالي يومين حاسس بوجع"* and later asserting *"المرض مستمر منذ أسبوع"*).
* **System Behavior:**
  1. The authoritative Assessment Engine MUST actively flag the conflict and prevent any arbitrary selection of a value.
  2. The engine MUST lock the current context data state and instruct the conversational AI to deliver a localized Arabic clarification prompt:
     *"لقد ذكرت في بداية التقييم أن الألم بدأ منذ يومين، ثم أشرت لاحقاً أنه مستمر منذ أسبوع. هل يمكنك تحديد المدة بدقة لمساعدتنا في تقييم حالتك؟"*

### ERR-005: Out-of-Scope Symptom Entry Exceeding MVP Medical Coverage
* **Condition:** A user describes an unfamiliar health complaint or symptom that falls completely outside the bounded Top 10 medical coverage criteria defined for the SymptoSense MVP release.
* **System Behavior:**
  1. The AI model layer MUST NOT create or infer medical rules, evaluate unsupported medical claims, or fabricate a clinical analysis outside the authorized validation context.
  2. The platform MUST invoke its defined fallback mechanism for unsupported symptom parameters and communicate the boundaries of the platform using a clear, direct safety notification in Arabic:
     *"عذرًا، العرض المذكور خارج نطاق التغطية الطبية الأولية للمنصة حاليًا. ننصحك باستشارة طبيب متخصص للحصول على تقييم دقيق وآمن."*

### ERR-006: Highly Ambiguous Conversational Symptom Input
* **Condition:** The user inputs an excessively vague colloquial phrase (e.g., *"عندي وجع في جنبي"* or *"جسمي فارد"*) that does not resolve to an exact body sub-region or anatomical side.
* **System Behavior:**
  1. The system MUST NOT guess, extrapolate, or assign a silent assumed clinical value to the active state representation.
  2. The Assessment Engine MUST classify the input property state explicitly as `Unknown` or `Unclear`.
  3. The conversational AI layer MUST trigger an active clarification question flow to isolate the exact anatomical region.

---

## 4. Clinical & Safety Edge Cases

### ERR-007: Late Ingestion of Red Flag Symptoms
* **Condition:** A user enters a critical red flag emergency indicator (e.g., sudden severe numbness, crushing chest pain, acute dyspnea) late into a conversation that was previously evaluated as a normal, low-urgency track.
* **System Behavior:**
  1. The platform MUST run safety evaluations continuously upon every single transaction loop.
  2. The system MUST execute an immediate safety override the exact millisecond the red flag pattern is recognized, halting the dynamic questionnaire pipeline instantly.
  3. The system MUST truncate all non-safety operational questionnaire scripts, invoke the immediate flow interruption rule, and force render the urgent Emergency Alert Screen instructing the user to seek emergency medical attention.

### ERR-008: Symptom Cluster Overload Constraints
* **Condition:** The natural language symptom extraction engine extracts a massive multi-symptom description that resolves to more than 6 distinct clinical symptom clusters inside one session.
* **System Behavior:**
  1. The Assessment Engine MUST apply the 6-symptom-cluster maximum cap boundary constraint.
  2. The engine MUST enforce strict priority execution filters: sorting all identified clusters based on Red Flags/Safety rules first, and Urgency levels second.
  3. The system MUST defer all remaining low-priority clusters to a subsequent assessment session, generate the report for the top active contexts, and display a prominent notice informing the user that additional symptoms remain unassessed and can be evaluated separately.

---

## 5. Session & State Boundary Edge Cases

### ERR-009: Expiration and Purging of Inactive Guest Data Sessions
* **Condition:** An unauthenticated guest user leaves an incomplete or idle evaluation session active within a device browser for a duration exceeding 24 hours.
* **System Behavior:**
  1. The platform's automated background operational data cycle MUST automatically purge or fully anonymize the unmigrated guest data.
  2. This process MUST run systematically to ensure that no stale, temporary health context parameters remain stored in an insecure or unauthenticated state, upholding the Privacy by Design core values. 
