# AI Module Specification

**Module:** AI Understanding & Communication
**Version:** 2.0
**Status:** Draft — Aligned with Approved AI Requirements, Business Rules, and Error & Edge Cases documents
**Supersedes:** v1.0

---

## 1. Module Overview

The AI Module provides natural-language understanding and natural-language communication for SymptoSense. It lets the user describe their symptoms in free-form Arabic (Modern Standard or Egyptian) text or voice, and it phrases the Assessment Engine's questions and results back to the user in natural language.

The module does not perform medical assessment, determine urgency, evaluate safety or red flags, or decide what medical information is required.

Its primary responsibility is:

> **Convert unstructured user input (text or voice-derived text) into a structured symptom payload that matches the approved AI output contract, and convert structured output from the Assessment Engine into natural-language questions and explanations.**

The AI Module is one stage in the assessment pipeline. Symptom information can arrive through:

- Text
- Voice (after Speech-to-Text)
- 3D body selection (handled entirely by the 3D Module — see Section 9)

Regardless of the input method, all symptom information is evaluated by the Assessment Module, which conducts the assessment and produces the final result. The AI Module never produces that result itself.

---

## 2. Main Responsibilities

The AI Module is responsible for:

- Understanding natural-language symptom descriptions in Arabic and English (AI-001, AI-003).
- Extracting structured symptom information (type, location, duration, severity, onset, associated symptoms) from free text (AI-002).
- Processing Speech-to-Text output as regular text input (AI-008).
- Detecting ambiguous input and flagging it instead of guessing (AI-005).
- Detecting information that is missing or unclear, as defined by the Assessment Engine's requirements (AI-006).
- Generating natural-language clarification questions for a required field supplied by the Assessment Engine (AI-007).
- Generating a user-friendly explanation of an Assessment Engine result without altering its meaning (AI-011).
- Validating any AI-generated structured output before it is used elsewhere in the system (AI-010).
- Minimizing the personal data forwarded to the AI provider (Data & Privacy, Section 52).

The module is not responsible for:

- Diagnosing the user's condition.
- Determining urgency or classifying red flags.
- Deciding which medical questions are required.
- Overriding, modifying, or suppressing an Assessment Engine result (BR-072, BR-085).
- Assuming a value for missing information (BR-030, BR-080).
- Converting 3D coordinates into anatomical locations — that is the 3D Module's responsibility (Section 9).
- Storing assessment history, health profile information, or medical knowledge.
- Communicating directly with the 3D Module, Auth Module, or Admin Module.
- Receiving the user's identity (name, email, userId) — see Section 11.

---

## 3. Structured Extraction Schema

This section defines the structured representation produced by the AI Module, per AI Requirements Section 10.

```text
StructuredSymptom
│
├── symptom_type            (e.g., "pain", "cough", "fever")
├── location
│     ├── region             (e.g., "abdomen")
│     ├── area                (e.g., "lower")
│     └── side                 ("left" | "right" | "center" | "not_applicable")
├── duration
│     ├── value
│     └── unit                 ("hours" | "days" | "weeks")
├── severity                 (0–10, when applicable)
├── onset
├── associated_symptoms[]
└── information_state         ("known" | "unknown" | "unclear" | "conflicting")
```

Every field the AI could not confidently determine is set to `information_state = "unknown"` or `"unclear"` rather than a guessed value (BR-030, AI-014).

### 3.1 Severity Scale Bounds

`severity` is bounded to 0–10. Any AI-extracted value outside this range must be rejected by the Output Validator (Section 18) before it reaches the Assessment Module, per ERR-002 — it must never be silently clamped or passed through.

### 3.2 MVP Symptom Scope (Resolved)

The team has agreed on the final MVP symptom scope: Headache, Fever, Cough, Abdominal Pain, Chest Pain, Back Pain. This closes the item previously listed as an Open Item ("Full extraction schema") — no change to the schema itself was required, since the six cases map onto the existing `symptom_type` + `location.region` fields:

| Agreed MVP case | Representation in this schema |
| :--- | :--- |
| Headache | `symptom_type: "pain"`, `location.region: "head"` |
| Fever | `symptom_type: "fever"` |
| Cough | `symptom_type: "cough"` |
| Abdominal Pain | `symptom_type: "pain"`, `location.region: "abdomen"` |
| Chest Pain | `symptom_type: "pain"`, `location.region: "chest"` |
| Back Pain | `symptom_type: "pain"`, `location.region: "back"` |

This does not exhaust `location.region` as a free-form field — the Assessment Module may still receive other regions if a user reports something outside these six — but the AI Module's supported extraction taxonomy for this MVP is scoped to the four regions above plus fever and cough (see Section 12).

---

## 4. Guest User Flow

```text
Guest
  |
  v
Assessment Started (per BR-011) — user provides first symptom text or voice input
  |
  v
AI Module: Understand Input
  |
  v
Structured Symptom(s)
  |
  v
Assessment Module
```

The AI Module does not distinguish between guest and registered users at the input-understanding level — it receives only the current turn's input and, when relevant, the current assessment context (never the user's identity or account data, per Section 11).

---

## 5. Logged-In User Flow

```text
Logged-In User
      |
      v
Assessment Started / Resumed (Draft, per BR-016)
      |
      v
AI Module: Understand Input
      |
      v
Structured Symptom(s)
      |
      v
Assessment Module
```

As with the Guest flow, the AI Module's behavior is identical for both user types. Any personalization (e.g., chronic conditions from the Health Profile) is injected into the assessment context by the application layer before reaching the AI Module, not retrieved by the AI Module itself.

---

## 6. Understanding & Clarification Flow

```text
User Input (text or voice-derived text)
    |
    v
AI Module: Extract Structured Symptom(s)
    |
    v
Output Validator: Valid?
    |
    +---- NO ----> Reject payload, request re-extraction (ERR-002)
    |
   YES
    |
    v
Send Structured Symptom(s) to Assessment Module
    |
    v
Assessment Module: Enough information?
    |
    +---- YES ---> Assessment continues / completes
    |
    NO
    |
    v
Assessment Module returns Required Field
    |
    v
AI Module: Generate Clarification Question
    |
    v
User answers
    |
    v
(loop back to "Extract Structured Symptom(s)")
```

Per ERR-007, this loop is not limited to the clarification path — every turn's structured output is submitted to the Assessment Module's Safety Evaluation immediately, regardless of whether the current path is a clarification or a new symptom. The AI Module has no authority to skip or delay this submission.

---

## 7. Constraints

- One extraction pass is performed per user turn; the AI Module does not batch multiple unresolved turns before submitting to the Assessment Module (this would delay safety evaluation, violating ERR-007).
- A maximum of one clarification question is generated per Assessment Module request — the AI Module does not bundle multiple required fields into a single compound question, to keep answers unambiguous.
- If the AI provider fails validation twice in a row for the same turn (ERR-002), the AI Module returns a controlled fallback response (Section 13) rather than retrying indefinitely.

---

## 8. Structured Output JSON (Contract)

```json
{
  "structured_symptoms": [
    {
      "symptom_type": "pain",
      "location": {
        "region": "abdomen",
        "area": "lower",
        "side": "right"
      },
      "duration": {
        "value": 2,
        "unit": "days"
      },
      "severity": null,
      "information_state": "known"
    }
  ],
  "requires_clarification": false
}
```

### Fields

| Field | Type | Description |
| :--- | :--- | :--- |
| `structured_symptoms[]` | array | One entry per distinct symptom described in the current turn |
| `symptom_type` | string | Normalized symptom identifier |
| `location` | object | region / area / side, or null when not provided |
| `duration` | object | value + unit, or null when not provided |
| `severity` | int? | 0–10, or null when not provided |
| `information_state` | string | `"known"` \| `"unknown"` \| `"unclear"` \| `"conflicting"` |
| `requires_clarification` | bool | True when any field the Assessment Module needs is unclear or missing |

The AI Module never includes `userId`, `name`, `email`, or any other personal identifier in this payload — only `assessmentId`-scoped data, carried separately by the calling layer (Section 11).

---

## 9. Relationship With Other Modules

```text
+---------------+         +----------------+         +-------------+
|   3D Module   |         |   AI Module    |         | Assessment  |
+---------------+         +----------------+         +-------------+
        \                         |                         /
         \                        |                        /
          \-------- anatomical_selections[] ---------------/
                                  |
                        structured_symptoms[]
                                  |
                                  v
                          Assessment Module
```

The AI Module never communicates with the 3D Module directly. Both modules send their respective structured output only to the Assessment Module, which is the single point of integration (consistent with the 3D Module Specification, Section 9).

---

## 10. Assessment Interaction

```text
AI Module
 |
 | structured_symptoms[]
 v
Assessment Module
 |
 v
Safety Evaluation (continuous, per ERR-007)
 |
 +---- Red Flag? --- YES ---> Halt questioning, return Urgent status
 |
 NO
 |
 v
Check collected information
 |
 +---- Enough information? --- YES ---> Generate Result
 |
 NO
 |
 v
Return Required Field to AI Module
 |
 v
AI Module generates clarification question
 |
 v
(loop)
```

Unlike the 3D Module's one-time input step, the AI Module is invoked on every conversational turn for energy and remainder of the assessment. It has no ability to decide when the conversation ends — that determination belongs entirely to the Assessment Module (BR-013, BR-018).

---

## 11. Data Minimization & Statelessness

The AI Module holds no persistent memory of its own. It does not have a database, and it does not retain:
- User Identity
- Assessment History
- Previous Turn Context (beyond what the caller explicitly supplies)
- Health Profile Data

Each call receives only the minimum context required for that specific operation (Data & Privacy, Sections 20–21, 52):

| Operation | Required Context | Excluded |
| :--- | :--- | :--- |
| Understand Input | Raw text/voice + `assessmentId` | `userId`, name, email |
| Generate Clarification | Required field + relevant symptom context | Full assessment history |
| Explain Result | The Assessment Result object (read-only) | Any data not in the result |

The calling layer (Conversation Orchestrator) is responsible for maintaining the ongoing conversation state and for resolving `assessmentId` to a user account when persistence is required. The AI Module never performs that resolution itself.

---

## 12. Database

No database is required. The severity scale bounds, supported symptom taxonomy for extraction, and prompt templates per provider are represented as static configuration, e.g.:

```text
Supported Symptom Types (MVP): pain, fever, cough
Supported Pain Regions (MVP): head, abdomen, chest, back

Severity Scale: 0–10
Supported Languages: ar-EG, ar-SA (MSA), en
```

This list reflects the six agreed MVP cases (Section 3.2): Headache, Fever, Cough, Abdominal Pain, Chest Pain, Back Pain. It replaces the earlier placeholder set, which included vomiting and shortness_of_breath — those are not in MVP scope and have been removed from this configuration; they can be reintroduced later as a config change with no structural impact.

---

## 13. AI Provider Integration

The AI Module is a consumer of an external Open-Weight model (Qwen, Llama, or MedGemma — final selection pending evaluation per AI Requirements Section 21). It does not train, fine-tune, or manage the model itself.

```text
Application
 |
 v
IAiProviderAdapter
 |
 +---- QwenAdapter (AWS Bedrock)
 +---- LlamaAdapter (AWS Bedrock)
 +---- MedGemmaAdapter (AWS SageMaker, pending confirmation)
```

If the provider call fails or times out beyond the 2.5s threshold (ERR-001), the AI Module performs one immediate retry, then returns a controlled Arabic fallback message without fabricating extracted data:

> *"نواجه صعوبة مؤقتة في معالجة البيانات، لم نفقد أيًا من أعراضك المسجلة. يمكنك المحاولة مرة أخرى خلال ثوانٍ."*

---

## 14. Backend API

```http
POST /api/ai/understand
```
Request:
```json
{
  "assessmentId": "a-12345",
  "rawInput": "بقالي يومين حاسس بوجع تحت بطني ناحية اليمين",
  "inputType": "text"
}
```
Response: Structured Output JSON shown in Section 8.

```http
POST /api/ai/clarify
```
Request:
```json
{
  "assessmentId": "a-12345",
  "requiredField": "symptom_severity"
}
```
Response:
```json
{
  "question": "لو هنقيّم الألم من 0 لـ 10، تقريبًا هتحطه عند رقم كام؟"
}
```

```http
POST /api/ai/explain
```
Request: The Assessment Result object, unmodified.  
Response:
```json
{
  "explanation": "بناءً على الأعراض اللي وصفتها، في احتمال إنها تكون مرتبطة بـ..."
}
```

---

## 15. Clean Architecture

```text
AI Module
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
│   └── AI Provider Adapters
│
└── Presentation
    └── API Controllers
```

---

## 16. Domain Layer

```text
StructuredSymptom
├── SymptomType
├── Location (Region, Area, Side)
├── Duration (Value, Unit)
├── Severity
└── InformationState

ClarificationRequest
├── RequiredField
└── SymptomContext
```

Still no dependency on ASP.NET, Entity Framework, HTTP, controllers, databases, or any specific AI provider SDK.

---

## 17. Application Layer

```csharp
public class StructuredSymptomDto
{
    public string SymptomType { get; set; }
    public LocationDto Location { get; set; }
    public DurationDto Duration { get; set; }
    public int? Severity { get; set; }
    public string InformationState { get; set; }
}

public class LocationDto
{
    public string Region { get; set; }
    public string Area { get; set; }
    public string Side { get; set; }
}

public class DurationDto
{
    public int Value { get; set; }
    public string Unit { get; set; }
}

public class UnderstandInputRequestDto
{
    public string AssessmentId { get; set; }
    public string RawInput { get; set; }
    public string InputType { get; set; } // "text" | "voice"
}

public class UnderstandInputResponseDto
{
    public List<StructuredSymptomDto> StructuredSymptoms { get; set; }
    public bool RequiresClarification { get; set; }
}

public interface IAiUnderstandingService
{
    Task<UnderstandInputResponseDto> UnderstandAsync(UnderstandInputRequestDto request);
}

public interface IClarificationService
{
    Task<string> GenerateClarificationAsync(string requiredField, StructuredSymptomDto context);
}

public interface IExplanationService
{
    Task<string> ExplainResultAsync(AssessmentResultDto result);
}

public interface IStructuredOutputValidator
{
    ValidationResult Validate(StructuredSymptomDto symptom);
}
```

`IStructuredOutputValidator.Validate` rejects out-of-range values (e.g., Severity = 15) before `UnderstandInputResponseDto` is returned to the caller (ERR-002).

---

## 18. Infrastructure Layer

```csharp
public interface IAiProviderAdapter
{
    Task<string> CompleteAsync(string prompt);
}
```

`QwenBedrockAdapter`, `LlamaBedrockAdapter`, and `MedGemmaAdapter` each implement `IAiProviderAdapter` against their respective AWS endpoint. Swapping the active provider requires only a configuration change, not a change to Application or Domain layers.

---

## 19. Presentation Layer

```text
AIController
```
```http
POST /api/ai/understand
POST /api/ai/clarify
POST /api/ai/explain
```
Delegates directly to the Application layer; no validation or business logic lives in the controller itself.

---

## 20. Open Items Requiring Team Confirmation

1. **Conversation Orchestrator ownership.** Something must own the multi-turn loop and short-lived conversation state (assumed Redis outside the AI Module).
2. **Full extraction schema.** RESOLVED — MVP covers Headache, Fever, Cough, Abdominal Pain, Chest Pain, Back Pain (Section 3.2).
3. **AI provider configuration store.** Where prompt templates and provider endpoint configuration live.
4. **MedGemma hosting path.** AWS managed offering vs. self-hosted GPU infrastructure.
5. **Output Validator boundary.** Whether `IStructuredOutputValidator` lives only inside the AI Module or is shared with the Assessment Module as a defense-in-depth check.

---

## 21. Summary

```text
Receive User Input (text / voice)
      ↓
Extract Structured Symptom(s)
      ↓
Validate Output (reject if out of range)
      ↓
Send structured_symptoms[] to Assessment Module
      ↓
Assessment Module: Safety Evaluation + Sufficiency Check
      ↓
   Enough? ----NO----> Return Required Field ----> Generate Clarification ----> (loop)
      ↓
     YES
      ↓
Assessment Result
      ↓
AI Module: Generate Explanation (read-only)
      ↓
User
```

The AI Module runs on every conversational turn until the Assessment Module signals completion. It never determines completion itself, never evaluates safety, and never modifies a result it did not produce.