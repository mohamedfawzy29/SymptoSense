# Assessment Engine Module — Architectural Documentation

**Project:** SymptoSense  
**Module Code:** AE  
**Version:** 1.1  
**Status:** Architectural Documentation — Pre-Implementation  
**Last Updated:** 2026-09-22

---

## 1. Module Overview

### 1.1 Module Definition

The **Assessment Engine** is the core decision-making module of SymptoSense.

It is responsible for making the system's **assessment decisions**. It receives structured assessment information, applies approved assessment and safety rules, determines whether enough information is available, and produces a traceable assessment outcome.

The Assessment Engine is not responsible for understanding natural language, generating user-facing questions, presenting UI, or modifying medical rules.

### 1.2 Authority Principle

> No other component may override or replace an Assessment Engine decision.

| Component | Responsibility |
|---|---|
| **AI Module** | Understands natural language, extracts and structures information, and helps communicate results. It does not make the final assessment decision. |
| **Assessment Engine** | The authoritative component for assessment decisions. |
| **UI / Frontend** | Collects and presents information. It does not make medical decisions. |
| **Medical Knowledge Module** | Provides approved medical rules, safety rules, supported medical knowledge, and their versions. |
| **Safety Rules** | Define safety conditions and red-flag rules. The Assessment Engine evaluates them. |

> **Source:** `18-assessment-engine-requirements.md`, Business Rules.

---

## 2. Responsibilities and Boundaries

### 2.1 What the Assessment Engine Does

The Assessment Engine is responsible for:

- Receiving and validating the structured assessment context.
- Tracking the state of assessment information.
- Distinguishing between known, unknown, not-provided, unclear, conflicting, and confirmed information.
- Determining the minimum information required to continue an assessment.
- Detecting missing, ambiguous, and conflicting information that affects evaluation.
- Evaluating multiple symptoms as a combined assessment context.
- Identifying relationships between symptoms and organizing them into evaluation clusters when required.
- Applying approved assessment rules.
- Performing safety evaluation and evaluating red flags.
- Determining the assessment urgency level (`Urgent` / `Non-Urgent` for the MVP).
- Determining whether the assessment is complete.
- Producing the assessment result.
- Producing the structured information required by the communication layer to explain the result.
- Maintaining decision traceability.
- Supporting deterministic and reproducible evaluation when the same context and rule versions are used.

### 2.2 What the Assessment Engine Does Not Do

The Assessment Engine does **not**:

- Understand raw natural language.
- Decide what a user's free-form sentence means.
- Generate natural-language questions.
- Translate structured required information into Arabic or English questions.
- Create or modify medical rules.
- Act as the medical knowledge authoring system.
- Render or present the assessment result.
- Control frontend behavior.
- Independently diagnose a medical condition.
- Allow an AI model to override safety rules or assessment decisions.
- Depend on UI-specific concepts such as buttons, screens, camera coordinates, or 3D mesh identifiers.

### 2.3 Boundary with the AI Module

The boundary between AI and the Assessment Engine should remain explicit.

Example:

```text
Assessment Engine
    ↓
Required information:
"location.side is required"
    ↓
AI Module
    ↓
Natural-language question:
"Is the pain on the right or left side?"
    ↓
User
    ↓
"Right side"
    ↓
AI Module
    ↓
Structured data:
{
    "locationSide": "Right",
    "informationState": "Known"
}
    ↓
Assessment Engine
```

The AI layer is responsible for interpretation and communication. The Assessment Engine is responsible for evaluating the structured information.

The Assessment Engine must not require a specific AI provider or model.

---

## 3. Internal Architecture

### 3.1 Architectural Style

The Assessment Engine should follow **Clean Architecture principles inside the module**.

The layers are:

```text
┌─────────────────────────────────────────────────────────────┐
│                 Assessment Engine Module                    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Interface / API Layer                  │   │
│  │     HTTP Endpoints, Request/Response Mapping        │   │
│  └────────────────────────┬────────────────────────────┘   │
│                           ↓                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Application Layer                      │   │
│  │   Commands | Queries | Use Cases | Orchestration     │   │
│  └────────────────────────┬────────────────────────────┘   │
│                           ↓                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                 Domain Layer                        │   │
│  │                                                     │   │
│  │ Assessment | Context | Symptoms | Safety            │   │
│  │ Sufficiency | Rules | Urgency | Result              │   │
│  │                                                     │   │
│  │ Domain Services | Policies | Domain Events           │   │
│  └────────────────────────┬────────────────────────────┘   │
│                           ↑                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │             Infrastructure Layer                    │   │
│  │ EF Core | PostgreSQL | Rule Provider | Event Bus     │   │
│  │ External Integrations | Persistence                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Dependency Direction

The main dependency rule is:

```text
Interface
    ↓
Application
    ↓
Domain

Infrastructure
    ↓
implements abstractions defined by Application/Domain
```

The Domain must not depend directly on:

- ASP.NET Core
- EF Core
- PostgreSQL
- Semantic Kernel
- Ollama
- HTTP
- MediatR
- Specific AI models

This keeps the assessment logic independently testable and prevents infrastructure choices from becoming domain rules.

### 3.3 Internal Domain Capabilities

The module may contain domain services/policies responsible for:

| Capability | Responsibility |
|---|---|
| **Assessment Evaluation** | Coordinates evaluation of the current assessment context. |
| **Safety Evaluation** | Evaluates approved safety rules and red flags. |
| **Information Sufficiency** | Determines whether enough information is available for evaluation. |
| **Symptom Relationship Evaluation** | Determines relationships between symptoms when required by the rules. |
| **Explanation Prioritization** | Determines the presentation priority of applicable possibilities according to validated rules. |
| **Assessment Lifecycle** | Enforces valid assessment state transitions. |
| **Traceability** | Captures the rules, knowledge versions, and factors behind a decision. |

These are domain responsibilities, not necessarily classes with these exact names.

---

## 4. Domain Model

### 4.1 `Assessment` — Aggregate Root

`Assessment` is the primary aggregate root of the Assessment Engine.

Conceptual model:

```text
Assessment
├── Id: Guid
├── Owner/Session Reference
├── State: AssessmentState
├── Context: AssessmentContext
├── Clusters: Collection<SymptomCluster>
├── Result: AssessmentResult?
├── DecisionMetadata
│   ├── RuleVersion
│   └── KnowledgeVersion
├── CreatedAt
├── UpdatedAt
└── CompletedAt: DateTime?
```

### 4.1.1 Ownership and Session References

The Assessment may be associated with:

- An authenticated user.
- A guest session.

However, the Assessment Engine should not own authentication logic.

`UserId` and `GuestSessionId` should be treated as references to external identity/session concepts rather than evidence that Identity belongs inside the Assessment Engine domain.

### 4.2 `AssessmentContext` — Domain Concept

`AssessmentContext` represents the information currently available to the Assessment Engine for evaluating an assessment.

Conceptually:

```text
AssessmentContext
├── Symptoms
├── Answers / Collected Information
├── Information States
├── Relevant Health Context
├── Corrections / Updated Values
└── Current Evaluation Context
```

The exact persistence representation is an implementation concern.

The Domain should not require the context to be stored as a JSON document merely because PostgreSQL JSONB may be used later.

### 4.3 `InformationState`

`InformationState` represents the semantic state of a piece of assessment information.

```text
InformationState
├── Known
├── Unknown
├── NotProvided
├── Unclear
├── Conflicting
└── Confirmed
```

Critical rule:

> `Unknown` must never be interpreted as `False` unless an explicit validated rule defines that behavior.

Example:

```text
fever = Unknown
```

does not mean:

```text
fever = False
```

This distinction is safety-critical because treating missing information as a negative answer can hide a relevant red flag.

### 4.4 `StructuredSymptom`

A symptom should be represented in a structured form before entering the Assessment Engine.

Conceptually:

```text
StructuredSymptom
├── Type
├── Location
├── Onset
├── Duration
├── Severity
├── Frequency
├── Pattern
├── Triggers
├── Associated Symptoms
├── Relevant Context
└── Information States
```

The source of the symptom may be:

- Natural language
- Voice input
- Structured UI input
- 3D body interaction

The Assessment Engine should consume the normalized structured representation rather than source-specific details.

### 4.5 `Location`

`Location` represents the semantic anatomical location of a symptom.

Example:

```text
Location
├── Region
├── Area
└── Side
```

The Domain should not know how the location was selected.

For example, these are outside the Assessment Engine Domain:

```text
3D mesh ID
mouse coordinates
camera position
selected UI element
```

They should be converted into a semantic anatomical representation before reaching the Engine.

### 4.6 `SymptomCluster`

A `SymptomCluster` represents a group of symptoms that the Assessment Engine evaluates together when their relationships affect the assessment.

Conceptually:

```text
SymptomCluster
├── Id
├── Symptoms
├── Relationship
├── Priority
└── EvaluationResult?
```

Supported relationships include:

```text
Related
PotentiallyRelated
Independent
Unknown
```

The MVP limits the number of evaluation clusters to a maximum of six, with prioritization based on safety and assessment relevance.

The cluster limit is a business rule, not a reason to hard-code `6` throughout the domain.

### 4.7 `AssessmentState`

```text
AssessmentState
├── NotStarted
├── InProgress
├── Draft
├── Completed
└── Abandoned
```

State transitions must be governed by business rules rather than by directly assigning a public property.

Conceptually:

```text
NotStarted
    ↓
InProgress
    ↓
Completed

InProgress
    ↓
Draft
    ↓
InProgress
    ↓
Completed

InProgress / Draft
    ↓
Abandoned
```

The exact allowed transitions must be validated against the Business Rules.

### 4.8 `AssessmentResult`

`AssessmentResult` represents the final structured outcome of an assessment.

Conceptually:

```text
AssessmentResult
├── Urgency
├── Possible Explanations
├── Safety Alerts
├── Recommended Next Step
├── Supporting Information
├── Uncertainty / Limitations
└── Traceability
```

The result must not present a possible explanation as a confirmed diagnosis.

The user-facing result may contain a maximum of three possible explanations according to the current Business Rules.

### 4.9 `Urgency`

For the MVP:

```text
Urgency
├── Urgent
└── NonUrgent
```

Urgency is a decision produced by the Assessment Engine after applicable safety and assessment rules have been evaluated.

The AI layer may communicate the urgency naturally but may not change it.

### 4.10 `SafetyEvaluation`

`SafetyEvaluation` represents the outcome of evaluating the current assessment context against approved safety and red-flag rules.

Conceptually:

```text
SafetyEvaluation
├── IsUrgent
├── RedFlagResults
├── TriggeredRules
└── EvaluationMetadata
```

Every assessment must undergo the required safety evaluation before a final result is considered complete.

### 4.11 `RedFlagResult`

A red flag represents a validated safety condition detected from the assessment context.

Conceptually:

```text
RedFlagResult
├── RuleId
├── Severity
├── Triggering Information
├── Recommended Action
└── Traceability
```

A red flag is not itself a medical diagnosis.

### 4.12 `PossibleExplanation`

A possible explanation represents a medically supported possibility that may be consistent with the available information.

Conceptually:

```text
PossibleExplanation
├── Identifier
├── Name
├── Supporting Findings
├── Priority
├── Uncertainty
└── Medical Knowledge Reference
```

The Assessment Engine may evaluate more possibilities internally than are presented to the user. The current Business Rules limit the user-facing list to three.

### 4.13 `TraceabilityRecord`

Every completed assessment should be traceable to the information and decision inputs that produced it.

Conceptually:

```text
TraceabilityRecord
├── RuleVersion
├── KnowledgeVersion
├── Applied Rule References
├── Safety Rule References
├── Required Information Decisions
├── Relevant Findings
└── Evaluation Timestamp
```

The exact audit-log implementation is an infrastructure concern.

---

## 5. Domain Events

Domain events should represent **meaningful domain facts**, not every internal method call.

Candidate events include:

```text
IDomainEvent
├── AssessmentStarted
├── InformationAdded
├── InformationCorrected
├── ConflictDetected
├── SafetyRiskDetected
├── UrgencyChanged
└── AssessmentCompleted
```

### 5.1 Event Responsibility

Example:

```text
InformationCorrected
    ↓
Assessment re-evaluation
    ↓
Safety evaluation
    ↓
Sufficiency evaluation
    ↓
Lifecycle/result update
```

A domain event should not itself be treated as a command.

For example:

```text
InformationCorrectedEvent
```

means:

> Information was corrected.

It does not mean:

> Execute SafetyEvaluator now.

The Application/Domain orchestration decides what reactions are required.

### 5.2 Event Bus Clarification

For the Modular Monolith, in-process domain/application events are sufficient for the MVP.

**MediatR may be used as an implementation mechanism**, but it is not itself an architectural "event bus" or a domain concept.

An external message broker should only be introduced if there is a demonstrated need for cross-process asynchronous communication.

---

## 6. Data Model — Initial Persistence Direction

The following is an initial persistence model, not the final database schema.

```text
┌──────────────────────────────────┐
│           assessments            │
├──────────────────────────────────┤
│ id                  UUID PK      │
│ owner_reference     UUID?        │
│ guest_session_ref   VARCHAR?     │
│ state               VARCHAR      │
│ created_at          TIMESTAMP    │
│ updated_at          TIMESTAMP    │
│ completed_at        TIMESTAMP?   │
└───────────────┬──────────────────┘
                │ 1
                │
                │ many
                ▼
┌──────────────────────────────────┐
│        symptom_clusters          │
├──────────────────────────────────┤
│ id                  UUID PK      │
│ assessment_id       UUID FK      │
│ relationship        VARCHAR      │
│ priority            INT         │
│ evaluation_result   JSONB?      │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│       assessment_contexts        │
├──────────────────────────────────┤
│ id                  UUID PK      │
│ assessment_id       UUID FK      │
│ context_data        JSONB        │
│ snapshot_at         TIMESTAMP    │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│       assessment_results         │
├──────────────────────────────────┤
│ id                  UUID PK      │
│ assessment_id       UUID FK UQ   │
│ result_data         JSONB        │
│ generated_at        TIMESTAMP    │
└──────────────────────────────────┘
```

### 6.1 Persistence Principles

The Domain Model must not be designed around PostgreSQL JSONB.

JSONB is an implementation option that may be appropriate for evolving medical/assessment data, but the final persistence model must be validated against:

- Query requirements
- Auditability
- Data retention
- Versioning
- Performance
- Integrity constraints
- Privacy requirements
- Reporting requirements

### 6.2 Event Persistence

A persistent event log may be introduced for auditability and traceability:

```text
domain_events_log
├── id
├── assessment_id
├── event_type
├── payload
└── occurred_at
```

Whether every domain event should be persisted must be decided during the auditability and persistence design phase.

---

## 7. Use Cases and Interface Direction

The following use cases describe the module's external behavior. Exact HTTP contracts should be finalized during API Design.

### UC-AE-001: Start a New Assessment

**Description:** Creates a new assessment for a guest session or authenticated user after the required initial context is available.

**Primary outcome:**

```text
Assessment created
State = InProgress
Next required information identified
```

Possible interface:

```http
POST /api/v1/assessments
```

Example request:

```json
{
  "healthContext": {
    "age": 35,
    "gender": "male",
    "chronicConditions": ["diabetes"],
    "currentMedications": [],
    "allergies": []
  }
}
```

Example response:

```json
{
  "assessmentId": "uuid",
  "state": "InProgress",
  "nextStep": {
    "type": "SymptomInput"
  }
}
```

The `message` should not be generated by the Assessment Engine.

---

### UC-AE-002: Add Symptoms

**Description:** Adds normalized structured symptoms to the current assessment and triggers re-evaluation.

Possible interface:

```http
POST /api/v1/assessments/{assessmentId}/symptoms
```

Example:

```json
{
  "symptoms": [
    {
      "type": "pain",
      "location": {
        "region": "abdomen",
        "area": "lower",
        "side": "right"
      },
      "duration": {
        "value": 2,
        "unit": "days"
      },
      "severity": 7,
      "triggers": ["movement"],
      "informationStates": {
        "onset": "Unknown",
        "frequency": "Unknown"
      }
    }
  ]
}
```

The source may be tracked as metadata:

```text
NaturalLanguage
Voice
StructuredInput
3DBody
```

but the Domain should operate on the normalized structured information.

---

### UC-AE-003: Provide an Answer

**Description:** Adds structured information to the assessment and triggers re-evaluation.

Possible interface:

```http
POST /api/v1/assessments/{assessmentId}/answers
```

Example:

```json
{
  "questionId": "q_fever_presence",
  "answer": {
    "value": true,
    "informationState": "Known"
  }
}
```

The Assessment Engine may return:

```json
{
  "state": "InProgress",
  "nextStep": {
    "type": "Question",
    "requiredInformation": {
      "field": "fever_degree",
      "reason": "Required for safety evaluation"
    }
  }
}
```

The AI/Communication layer may convert the structured requirement into natural language.

---

### UC-AE-004: Correct Previously Provided Information

**Description:** Replaces previously provided information and re-evaluates the assessment.

Possible interface:

```http
PATCH /api/v1/assessments/{assessmentId}/answers/{questionId}
```

Example:

```json
{
  "newAnswer": {
    "value": "7 days",
    "informationState": "Known"
  },
  "correctionReason": "UserCorrection"
}
```

The correction must trigger the required re-evaluation of:

```text
Context
    ↓
Safety
    ↓
Required Information
    ↓
Assessment Rules
    ↓
State / Result
```

---

### UC-AE-005: Get Current Assessment State

**Description:** Retrieves the current state of an assessment, especially when resuming a saved Draft.

Possible interface:

```http
GET /api/v1/assessments/{assessmentId}
```

The response may include:

```json
{
  "assessmentId": "uuid",
  "state": "Draft",
  "urgency": "NonUrgent",
  "nextStep": {
    "type": "Question",
    "requiredInformation": {
      "field": "pain_radiation"
    }
  }
}
```

---

### UC-AE-006: Get Assessment Result

**Description:** Returns the completed assessment result.

Possible interface:

```http
GET /api/v1/assessments/{assessmentId}/result
```

Example:

```json
{
  "assessmentId": "uuid",
  "state": "Completed",
  "urgency": "NonUrgent",
  "safetyAlerts": [],
  "possibleExplanations": [
    {
      "name": "Possible Explanation A"
    },
    {
      "name": "Possible Explanation B"
    }
  ],
  "recommendedNextStep": {
    "type": "NonUrgentMedicalEvaluation"
  },
  "uncertainty": {
    "present": true
  }
}
```

The final wording, disclaimer presentation, and UI representation belong to the communication/presentation layer.

---

### UC-AE-007: Emergency Escalation

If safety evaluation identifies a red flag that requires urgent action, the Assessment Engine must produce an urgent outcome.

Conceptually:

```text
New information
    ↓
Safety Evaluation
    ↓
Red Flag Detected
    ↓
Urgency = Urgent
    ↓
Assessment becomes safety-complete
    ↓
No unnecessary additional questions
    ↓
Urgent result / recommended action
```

Example structured result:

```json
{
  "state": "Completed",
  "urgency": "Urgent",
  "safetyAlerts": [
    {
      "severity": "Critical",
      "redFlagReference": "ChestPainWithShortnessOfBreath",
      "recommendedAction": "Seek urgent medical attention"
    }
  ],
  "possibleExplanations": []
}
```

The frontend decides how the emergency state is visually presented.

---

### UC-AE-008: Get Assessment History

Assessment history is a user-facing capability that may depend on the Assessment/History boundary.

Possible interface:

```http
GET /api/v1/assessments?page=1&limit=10
```

This use case should not force the Assessment Engine Domain to own authentication or user-management logic.

---

## 8. Key Flows

### Flow 1: Assessment Happy Path

```text
User / Frontend
      │
      │ Start Assessment
      ▼
Assessment Module
      │
      ├── Create Assessment
      │
      ├── Evaluate Context
      │
      ├── Safety Evaluation
      │
      ├── Sufficiency Check
      │
      └── Required Information
      │
      ▼
AI / Communication Layer
      │
      │ asks user for required information
      ▼
User
      │
      ▼
AI / Communication Layer
      │
      │ structured answer
      ▼
Assessment Module
      │
      ├── Update Context
      ├── Safety Re-evaluation
      ├── Sufficiency Check
      └── Assessment Evaluation
      │
      ├── Not Sufficient → Required Information
      │
      └── Sufficient
              ↓
        Final Safety Evaluation
              ↓
        Assessment Result
```

### Flow 2: Emergency Escalation

```text
New Structured Information
        ↓
Assessment Context Updated
        ↓
Safety Evaluation
        ↓
Red Flag Detected
        ↓
Urgency = Urgent
        ↓
Stop unnecessary information gathering
        ↓
Produce urgent safety outcome
        ↓
Frontend / Communication Layer
        ↓
Present urgent guidance
```

### Flow 3: User Correction and Re-evaluation

```text
User Correction
      ↓
Update Assessment Context
      ↓
Re-evaluate affected context
      ↓
Safety Evaluation
      ↓
Sufficiency Evaluation
      ↓
Assessment Rules
      ↓
Update State / Result / Required Information
```

### Flow 4: Draft Resume

```text
Registered User
      ↓
Resume Draft
      ↓
Load Assessment Context
      ↓
Re-evaluate current state
      ↓
Return current required information / result
```

---

## 9. Dependencies and Module Boundaries

### 9.1 Logical Dependencies

```text
                    ┌─────────────────────────┐
                    │   Medical Knowledge     │
                    │                         │
                    │ Assessment Rules        │
                    │ Safety Rules             │
                    │ Supported Coverage      │
                    │ Knowledge Versions       │
                    └────────────┬────────────┘
                                 │
                                 ▼
┌───────────────┐       ┌───────────────────────┐
│ AI /          │       │ Assessment Engine     │
│ Communication │──────►│                       │
│               │       │ Decision Authority    │
└───────┬───────┘       └───────────┬───────────┘
        ▲                            │
        │                            ▼
        │                   ┌───────────────────┐
        └───────────────────│ Result / Required │
                            │ Information       │
                            └───────────────────┘
```

### 9.2 Dependency Rules

The Assessment Engine should depend on **abstractions**, not implementation details.

For example:

```text
Assessment Engine
    ↓
IMedicalKnowledgeProvider
    ↓
Medical Knowledge implementation
```

and:

```text
Assessment Engine
    ↓
IAssessmentRuleProvider
    ↓
Rule Engine / Medical Knowledge implementation
```

### 9.3 Module Boundary Clarification

The Assessment Engine should not directly depend on:

- Auth implementation
- Frontend implementation
- 3D rendering implementation
- Specific AI model
- Specific AI provider

Other modules should communicate with the Assessment Engine through explicit application contracts, module interfaces, or domain/application events according to the final Modular Monolith architecture.

> **Important:** In a Modular Monolith, "communication through the API layer only" is not a required rule. Internal modules may communicate through explicit contracts without making HTTP calls to each other.

---

## 10. Business Rules Mapping

| Business Rule | Assessment Engine Impact |
|---|---|
| BR-010 | Supports multiple symptoms in one assessment. |
| BR-011 | Assessment starts as `NotStarted` until the first relevant input. |
| BR-012 | Assessment remains `InProgress` while required information is missing. |
| BR-013 | Completion depends on sufficient information, not question count. |
| BR-014 | Engine determines required information. |
| BR-015 | Supports correction and re-evaluation. |
| BR-016 | Registered users may resume saved Draft assessments. |
| BR-017 | Final result requires Safety Evaluation. |
| BR-019 | Questions are dynamically determined rather than fixed. |
| BR-020 | Already sufficient information should not be unnecessarily requested again. |
| BR-023 | Conflicting information must be detected and handled explicitly. |
| BR-025 | Uses minimum sufficient information rather than maximum information gathering. |
| BR-028 | Corrections update context and trigger re-evaluation. |
| BR-030 | Missing information is not treated as a negative value. |
| BR-036 | Assessment Engine is the authoritative assessment decision-maker. |
| BR-043 → BR-046 | Supports symptom relationships and clustering, with an MVP maximum of six clusters. |
| BR-047, BR-048 | Produces and prioritizes possible explanations, with a maximum of three user-facing explanations. |
| BR-049 → BR-062 | Performs safety evaluation and prioritizes safety over convenience or likelihood alone. |
| BR-065 → BR-071 | Produces structured result components, uncertainty, explanation support, and recommended next steps. |

---

## 11. Technical Decisions and Open Decisions

This section intentionally separates **confirmed architectural principles** from **implementation choices that still require team validation**.

### 11.1 Confirmed Direction

| Decision | Direction |
|---|---|
| Architectural style | Clean Architecture principles inside the module |
| Deployment style | Part of the Modular Monolith |
| Assessment authority | Assessment Engine |
| AI authority | AI does not determine or override assessment decisions |
| Safety authority | Safety rules are evaluated by the Assessment Engine |
| Assessment result | Structured and traceable |
| Medical knowledge | Versioned and approved |
| AI dependency | Replaceable; no specific model required by the Domain |

### 11.2 Candidate Implementation Choices — Team Decision Required

| Topic | Candidate | Status |
|---|---|---|
| Rule evaluation | Microsoft RulesEngine or custom rule abstraction | **Open** |
| In-process messaging | MediatR or another internal dispatcher | **Open** |
| Persistence | PostgreSQL + EF Core | **Likely**, validate with overall system architecture |
| Context storage | Relational model, JSONB, or hybrid | **Open** |
| AI integration | Separate AI Module through an application contract | **Recommended boundary, implementation open** |
| External message broker | None initially unless a demonstrated requirement appears | **Open / likely unnecessary for MVP** |
| Event persistence | Persist selected audit events vs full event log | **Open** |

### 11.3 Important Technical Corrections

#### Rule Engine

A rule-engine library is an implementation choice, not part of the Domain model.

The Domain should depend on an abstraction such as:

```text
IAssessmentRuleProvider
```

rather than directly depending on Microsoft RulesEngine.

#### MediatR

MediatR is an implementation mechanism for in-process request/notification handling.

It should not be described as an external Event Bus.

The architectural concept is:

```text
Domain / Application Events
```

MediatR may be one implementation option.

#### Semantic Kernel / Ollama

Semantic Kernel and Ollama belong to the AI/infrastructure boundary, not the Assessment Engine Domain.

The Assessment Engine should not contain:

```text
SemanticKernel
Ollama
Qwen
Llama
MedGemma
```

as domain dependencies.

If AI is needed for a specific capability, expose an abstraction or module contract and keep the implementation outside the Domain.

---

## 12. Non-Functional Domain Constraints

The Assessment Engine must preserve the following properties.

### 12.1 Safety

Safety evaluation must happen before a final result.

### 12.2 Determinism

Given the same:

```text
Assessment Context
+
Assessment Rule Version
+
Medical Knowledge Version
+
Safety Rule Version
```

the decision should be reproducible unless an explicitly controlled probabilistic component is introduced.

### 12.3 Traceability

A completed assessment should be able to explain:

- Which information was used.
- Which information was missing.
- Why more information was required.
- Which assessment rules were applied.
- Which safety rules were evaluated.
- Why the assessment was considered complete.
- Why the urgency was selected.
- Which medical knowledge version was used.
- Why a possible explanation was presented.

### 12.4 No Unsupported Medical Output

The Engine must not produce possible explanations outside the approved MVP medical coverage.

### 12.5 No Diagnosis Authority

The Assessment Engine provides assessment guidance and possible explanations.

It does not claim to establish a confirmed diagnosis.

---

## 13. Architectural Decision Summary

The current intended architecture is:

```text
                 SymptoSense Modular Monolith
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      AI Module      Assessment Engine   Other Modules
                           │
                 ┌─────────┴─────────┐
                 │                   │
            Application           Domain
                 │                   │
                 │        ┌──────────┼──────────┐
                 │        │          │          │
                 │   Assessment   Safety   Sufficiency
                 │        │          │          │
                 │        └──────────┼──────────┘
                 │                   │
                 │              Result
                 │
                 ▼
             Infrastructure
                 │
        ┌────────┼─────────┐
        │        │         │
       EF     Knowledge   Rule
      Core     Provider   Provider
        │
    PostgreSQL
```

The key architectural principle is:

> **The Assessment Engine owns assessment decisions, while AI, UI, medical knowledge, persistence, and infrastructure remain separate concerns.**

The Domain should remain independent of the technical choices used to implement those concerns.

---

## 14. Next Design Step

Before implementing classes or database tables, the next step should be a **Domain Model Review**.

The team should explicitly review:

1. Which concepts are Entities.
2. Which concepts are Value Objects.
3. Which concepts are Enums.
4. Which behaviors belong inside the `Assessment` Aggregate.
5. Which behaviors belong to Domain Services/Policies.
6. What the exact Aggregate boundaries are.
7. Which rules belong to the Domain versus the Medical Knowledge module.
8. How Safety Evaluation interacts with the Assessment aggregate.
9. What the minimum public application contract of the module should be.
10. Which technical decisions require ADRs.

Only after this review should the team move to:

```text
Domain Model
    ↓
Aggregate Boundaries
    ↓
Application Use Cases
    ↓
Ports / Interfaces
    ↓
Persistence Model
    ↓
API Contracts
    ↓
Implementation
```

---

## 15. Document Status

**Current status:** Architectural Draft — Team Review Required

This document defines the current architectural direction and domain boundaries. It does not freeze unresolved implementation choices.

Any decision that materially affects the module's architecture should be recorded as an ADR before implementation.
