# Assessment Engine Requirements

**Document Version:** 1.0  
**Status:** Draft — Team Review Required  
**Last Updated:** 2026-08-15

---

## 1. Purpose

This document defines the functional, logical, and behavioral requirements of the SymptoSense Assessment Engine.

The Assessment Engine is responsible for processing structured assessment information, applying the approved assessment and safety logic, determining whether sufficient information is available, and producing the final assessment outcome.

The Assessment Engine shall operate within the boundaries defined by:

- Business Rules
- MVP Medical Coverage
- Medical Knowledge
- Safety Rules
- Data and Privacy Requirements
- Other approved system requirements

The purpose of this document is to define **what the Assessment Engine must do** without prematurely determining the exact technical implementation approach.

---

## 2. Assessment Engine Authority

The Assessment Engine shall be the authoritative component responsible for determining the assessment outcome.

No other application component shall independently override or replace the final assessment decision.

In particular:

- AI shall not independently determine the final assessment outcome.
- The UI shall not determine medical decisions.
- User-provided information shall not directly determine the final result without assessment logic.
- Safety decisions shall be processed through the approved Safety Rules.
- Assessment decisions shall be traceable to the information and rules used to produce them.

---

## 3. Core Responsibilities

The Assessment Engine shall be responsible for:

- Processing structured assessment information.
- Determining whether sufficient information is available.
- Identifying required missing information.
- Determining relevant next information to collect.
- Evaluating multiple symptoms as a combined assessment context.
- Applying approved assessment rules.
- Applying Safety and Red Flag rules.
- Determining the applicable urgency level.
- Evaluating supported possible explanations.
- Producing an assessment result.
- Providing the information required to explain the assessment result.
- Maintaining traceability between assessment decisions, user information, and applicable rules.

---

## 4. Assessment Input

The Assessment Engine shall receive structured information representing the current state of the assessment.

The input may include:

- Symptoms
- Symptom locations
- Symptom severity
- Symptom duration
- Symptom characteristics
- Associated symptoms
- Relevant user-provided context
- Relevant existing conditions when available
- Previous answers
- User corrections
- Information explicitly marked as unknown
- Other information supported by the approved MVP medical coverage

The Assessment Engine shall only use information that is explicitly available in the assessment context.

---

## 5. Structured Assessment Context

The Assessment Engine shall maintain a structured representation of the current assessment context.

The context shall distinguish between different information states where applicable:

- Known
- Unknown
- Not provided
- Unclear
- Conflicting
- Confirmed

The Assessment Engine shall not treat missing information as a negative answer unless the applicable assessment rule explicitly defines such behavior.

For example:

    fever = unknown

shall not be interpreted as:

    fever = false

---

## 6. Input Validation

Before processing an assessment, the Assessment Engine shall validate the received assessment information.

Validation may include:

- Required field validation
- Data type validation
- Supported value validation
- Context consistency
- Conflicting information detection
- Unsupported symptom detection
- Invalid or incomplete structured data detection

Invalid information shall not silently be treated as valid information.

---

## 7. Information Sufficiency

The Assessment Engine shall determine whether the available assessment information is sufficient to continue or complete the assessment.

The engine shall not require every possible piece of information.

Instead, it shall determine the minimum information required for the applicable assessment context.

The intended flow is:

    Current Assessment Context
            ↓
    Determine Required Information
            ↓
    Information Sufficient?
          /       \
        No         Yes
        ↓           ↓
    Request More   Continue
    Information    Assessment

This requirement supports the principle:

> Minimum Sufficient Information

---

## 8. Required Information Determination

The Assessment Engine shall determine which information is required based on:

- Current symptoms
- Available assessment context
- Applicable assessment rules
- Safety requirements
- MVP medical coverage
- Previously collected information

The engine shall not request information that is already sufficiently available in the assessment context.

---

## 9. Dynamic Assessment

The Assessment Engine shall support dynamic assessment flows.

The required information and next assessment step may change as new information becomes available.

The assessment shall therefore not depend on a single fixed questionnaire that is identical for every user.

The intended flow is:

    Current Information
            ↓
    Evaluate Assessment State
            ↓
    Determine Missing Information
            ↓
    Collect Information
            ↓
    Update Assessment Context
            ↓
    Re-evaluate Assessment State
            ↓
    Continue or Complete

---

## 10. Question Responsibility Boundary

The Assessment Engine shall determine **what information is required**.

The Assessment Engine shall not be responsible for determining the natural-language wording of the question when AI is used for communication.

The responsibility boundary is:

    Assessment Engine
          ↓
    Required Information
          ↓
    AI
          ↓
    Natural-Language Question

Therefore:

> What to ask → Assessment Engine

> How to ask → Communication / AI Layer

The final architecture may modify this separation if the team determines that another implementation provides better safety, reliability, or maintainability.

---

## 11. Multiple Symptoms

The Assessment Engine shall support multiple symptoms within a single assessment.

The engine shall evaluate the symptoms as a combined assessment context rather than requiring the user to identify one primary symptom.

For example:

    Cough
    Fever
    Shortness of Breath

may be evaluated together as one assessment context.

The presence of multiple symptoms may affect:

- Required information
- Assessment rules
- Possible explanations
- Safety rules
- Urgency
- Final assessment result

---

## 12. Symptom Relationships

The Assessment Engine shall support rules that depend on relationships between multiple symptoms.

A rule may depend on:

- A single symptom
- Multiple symptoms occurring together
- Symptom combinations
- Symptom severity
- Symptom duration
- Relevant user context
- Other supported assessment information

The engine shall not evaluate each symptom independently when the applicable medical rules require combined evaluation.

---

## 13. Missing Information

The Assessment Engine shall explicitly track information that has not been provided.

Missing information shall remain unknown unless sufficient information is later provided.

The engine shall not assume that an unanswered question means:

- No
- False
- Normal
- Not present

unless the applicable rule explicitly defines that interpretation.

---

## 14. Ambiguous Information

The Assessment Engine shall identify information that cannot be reliably interpreted.

When information is ambiguous and is required for the assessment, the engine shall mark the information as requiring clarification.

The engine shall not silently choose an assumed value.

The intended flow is:

    Ambiguous Information
            ↓
    Clarification Required
            ↓
    User Provides Clarification
            ↓
    Assessment Context Updated
            ↓
    Assessment Re-evaluated

---

## 15. Conflicting Information

The Assessment Engine shall detect conflicting information when multiple pieces of information cannot reliably coexist.

For example:

    Duration = 2 days

followed later by:

    Duration = 7 days

The engine shall not arbitrarily select one value.

The conflicting information shall be identified and clarification may be requested when necessary.

---

## 16. User Corrections

The Assessment Engine shall support corrections to previously provided information.

When a user changes an answer:

    Previous Information
            ↓
    User Correction
            ↓
    Updated Assessment Context
            ↓
    Re-evaluate Applicable Rules
            ↓
    Re-evaluate Required Information
            ↓
    Continue Assessment

The updated information shall become the current assessment value when accepted.

---

## 17. Assessment State

The Assessment Engine shall support the assessment lifecycle defined by the Business Rules.

Relevant states include:

- Not Started
- In Progress
- Draft
- Completed
- Abandoned

The exact state transitions shall follow the approved Business Rules.

An assessment shall not be considered Completed merely because the user has answered a predetermined number of questions.

Completion shall depend on whether the required information for the applicable assessment is sufficient.

---

## 18. Assessment Completion

The Assessment Engine shall determine when an assessment has sufficient information for completion.

An assessment may be completed when:

- Required information is available.
- Applicable assessment rules can be evaluated.
- Required Safety Evaluation has been performed.
- No required clarification remains.
- The applicable assessment logic has reached a valid outcome.

The engine shall not continue asking questions after sufficient information has been collected unless additional information is required by an applicable rule.

---

## 19. Safety Evaluation

Every assessment shall undergo Safety Evaluation before a final assessment result is issued.

The Assessment Engine shall apply the approved Safety Rules and Red Flag rules applicable to the current assessment context.

Safety Evaluation shall have priority over non-safety assessment considerations.

The AI shall not independently determine whether an assessment is safe or urgent.

---

## 20. Red Flag Evaluation

The Assessment Engine shall evaluate the current assessment context for applicable Red Flags.

A Red Flag may depend on:

- A specific symptom
- Multiple symptoms
- Symptom severity
- Symptom duration
- Relevant context
- A combination of the above

The presence of a Red Flag shall not itself be considered a diagnosis.

Instead, it may affect:

- Urgency
- Assessment handling
- Result messaging
- Safety recommendations

---

## 21. Urgency Determination

The Assessment Engine shall determine the applicable urgency level according to the approved Safety Rules.

The MVP shall initially support:

- **Urgent**
- **Non-Urgent**

The exact meaning and behavior of each urgency level shall be defined by the approved Safety and Business Rules.

The AI shall not independently assign the urgency level.

---

## 22. Possible Explanations

The Assessment Engine may evaluate supported possible explanations based on the available assessment information.

Possible explanations shall be limited to the medical coverage supported by the MVP.

The engine shall not generate unsupported medical explanations outside the approved medical knowledge.

Possible explanations shall not be treated as confirmed diagnoses.

---

## 23. Explanation Evaluation

Possible explanations shall be evaluated using applicable:

- Symptoms
- Symptom combinations
- Relevant context
- Assessment rules
- Medical knowledge
- Safety considerations

The engine shall use only approved medical information and assessment logic.

---

## 24. Safety Priority Over Likelihood

The Assessment Engine shall not evaluate possible explanations using likelihood alone.

If a possible explanation has a lower estimated likelihood but represents a significant safety concern, applicable Safety Rules shall still be considered.

Safety-related decisions shall not be suppressed solely because another explanation appears more likely.

---

## 25. Assessment Result

The Assessment Engine shall produce a structured assessment result after the assessment has reached a valid completion state.

The result may contain:

- Assessment status
- Urgency level
- Possible explanations
- Relevant findings
- Safety-related information
- Supporting assessment information
- Information required for user-facing explanation

The exact result structure shall be defined by the system's data model and API requirements.

---

## 26. No Diagnosis Authority

The Assessment Engine shall not represent its output as a definitive medical diagnosis.

The system shall provide an assessment based on the supported medical coverage and available information.

The final wording shown to the user shall preserve the distinction between:

- Assessment
- Possible explanation
- Medical diagnosis

The system shall not claim certainty beyond what the approved medical rules and available information support.

---

## 27. Assessment Traceability

Important Assessment Engine decisions shall be traceable.

The system should be able to determine:

- Which user information was used
- Which assessment rules were applied
- Which Safety Rules were triggered
- Which Red Flags were identified
- Why additional information was required
- Why the assessment was considered complete
- Why the resulting urgency was assigned

The intended relationship is:

    User Information
          ↓
    Applied Rule(s)
          ↓
    Decision
          ↓
    Assessment Result

---

## 28. Determinism and Reproducibility

The Assessment Engine should provide reproducible results when the same assessment context and the same applicable medical rules are evaluated.

The team shall consider determinism and reproducibility as architectural requirements when evaluating possible implementation approaches.

If AI or probabilistic components are included in the final implementation, appropriate controls shall be defined to ensure that safety-critical decisions remain reliable and testable.

---

## 29. Medical Knowledge Dependency

The Assessment Engine shall operate using approved medical knowledge and assessment rules.

The engine shall not independently create or modify medical rules.

Medical knowledge changes shall follow an approved process for:

- Review
- Validation
- Versioning
- Approval
- Deployment

---

## 30. MVP Medical Coverage Boundary

The Assessment Engine shall operate only within the symptoms and medical coverage approved for the MVP.

If a user provides information outside the supported MVP coverage, the system shall not fabricate an assessment based on unsupported medical knowledge.

The behavior for unsupported coverage shall be defined by the Error & Edge Cases and Safety requirements.

---

## 31. Rule and Knowledge Versioning

Assessment rules and relevant medical knowledge shall be versioned where necessary.

An assessment should be associated with the rule and medical knowledge versions used to produce its result.

This supports:

- Traceability
- Auditing
- Reproducibility
- Debugging
- Medical review
- Future rule updates

---

## 32. Assessment Re-evaluation

The Assessment Engine shall re-evaluate the current assessment state when relevant information changes.

Re-evaluation may be triggered by:

- New user information
- User correction
- Clarification
- New symptom
- Updated symptom severity
- Updated duration
- Other information that affects applicable rules

The engine shall not continue using stale assessment decisions when relevant input information has changed.

---

## 33. Separation of Responsibilities

The Assessment Engine shall remain separated from responsibilities that belong to other system components.

The intended responsibility boundaries are:

    AI / Communication Layer
        ↓
    Understand and communicate user information

    Assessment Engine
        ↓
    Evaluate assessment logic and determine outcome

    Medical Knowledge
        ↓
    Provide approved medical information and rules

    Safety Rules
        ↓
    Define applicable safety conditions

    UI
        ↓
    Present information to the user

The final architecture may refine these boundaries while preserving the required system behavior.

---

# 34. Implementation Approach

The implementation approach of the Assessment Engine shall not be finalized during the requirements definition phase.

The SymptoSense team shall evaluate the available approaches after reviewing the complete Assessment Engine Requirements.

Three approaches shall be considered:

1. Rule-Based Decision Engine
2. AI-Based Assessment Engine
3. Hybrid Assessment Engine

The final decision shall be based on the requirements, safety considerations, feasibility, and expected MVP scope.

---

## 34.1 Rule-Based Decision Engine

A Rule-Based Decision Engine makes assessment decisions by applying predefined rules to structured user information.

The general concept is:

    Input
      ↓
    Evaluate Rules
      ↓
    Matching Rule(s)
      ↓
    Decision

For example:

    IF
        chest_pain = true
    AND
        shortness_of_breath = true

    THEN
        urgency = URGENT

The engine does not independently infer the medical rule.

The rule must be explicitly defined, validated, and maintained within the approved medical knowledge and assessment logic.

### Characteristics

A Rule-Based Decision Engine is generally:

- Deterministic
- Predictable
- Testable
- Traceable
- Easier to audit
- Easier to explain

For the same structured input and the same set of active rules, the engine should produce the same decision.

---

## 34.2 AI-Based Assessment Engine

An AI-Based Assessment Engine uses an AI or machine-learning model to infer assessment decisions from the provided information.

The general concept is:

    Input
      ↓
    AI Model
      ↓
    Learned Patterns / Inference
      ↓
    Assessment Output

Unlike a traditional rule-based engine, the decision logic is not necessarily represented as explicit human-defined rules.

The model may identify patterns across the provided information based on its training or learned representation.

### Characteristics

An AI-Based Assessment Engine may provide:

- Greater flexibility
- Ability to identify complex patterns
- Ability to work with less explicitly structured information
- Potential to improve as appropriate data and models become available

However, it may also introduce additional challenges related to:

- Explainability
- Determinism
- Testing
- Validation
- Model behavior
- Data requirements
- Monitoring
- Medical safety

---

## 34.3 Hybrid Assessment Engine

A Hybrid Assessment Engine may combine Rule-Based and AI-based components.

For example:

    User
      ↓
    AI
      ↓
    Structured Information
      ↓
    Assessment Engine
      ├── Rule-Based Logic
      ├── Safety Rules
      └── AI / ML Components
      ↓
    Assessment Result

A hybrid architecture may allow different technologies to be used for different responsibilities.

For example:

### AI may be used for:

- Natural-language understanding
- Symptom extraction
- Information normalization
- Pattern analysis
- Question generation
- Result explanation

### Rule-Based Logic may be used for:

- Safety Rules
- Red Flag evaluation
- Required information determination
- Assessment constraints
- Deterministic decisions

The exact division of responsibilities shall be determined by the final architecture decision.

---

# 35. Approach Comparison

The following comparison shall be used by the team when evaluating the possible implementation approaches.

| Aspect | Rule-Based Decision Engine | AI-Based Assessment Engine | Hybrid |
|---|---|---|---|
| Decision mechanism | Explicit predefined rules | Model inference | Combination |
| Deterministic behavior | High | Not necessarily | Depends on component |
| Predictability | High | Lower | Variable |
| Explainability | High | Potentially lower | Medium to High |
| Testing | Relatively straightforward | More complex | Moderate to High |
| Auditability | High | More challenging | High for rule-based parts |
| Natural-language understanding | Limited by itself | Strong | Strong |
| Pattern recognition | Limited to defined rules | Stronger | Strong |
| Training data requirement | No model training required | Usually requires appropriate data | Depends on AI components |
| Rule changes | Explicitly maintained | May require model/data changes | Depends on component |
| Safety-rule enforcement | Strong fit | Requires additional controls | Strong fit when rules are explicit |
| Implementation complexity | Generally lower for bounded rules | Generally higher | Moderate to High |
| Flexibility | Limited by defined rules | Higher | High |
| Medical decision control | Explicit | More difficult to control directly | Depends on architecture |

---

# 36. Architectural Decision Criteria

Before selecting the final implementation approach, the team shall evaluate at minimum:

- Medical safety
- Explainability
- Determinism
- Testability
- Traceability
- Maintainability
- Data availability
- Implementation complexity
- MVP scope
- Future scalability
- AI integration requirements
- Ability to validate assessment behavior
- Ability to audit assessment decisions

The team may select:

- Rule-Based Decision Engine
- AI-Based Assessment Engine
- Hybrid Assessment Engine
- Another suitable architecture

---

# 37. Architectural Decision Status

The implementation approach is currently:

**Status: Architectural Decision Pending Team Review**

This document intentionally does not mandate a specific implementation approach.

The requirements define **what the Assessment Engine must accomplish**.

The final architecture decision shall determine **how those requirements will be implemented**.

---

# 38. Dependency on Other Requirements

The Assessment Engine depends on several other SymptoSense system documents.

The primary dependencies are:

    Business Rules
          ↓
    Assessment Engine Requirements
          ↓
    MVP Medical Coverage
          ↓
    Medical Knowledge / Safety Rules
          ↓
    Assessment Implementation

The Assessment Engine Requirements shall remain consistent with approved changes to these documents.

---

# 39. Future Extensions

The Assessment Engine architecture should allow future expansion where practical, including:

- Additional supported symptoms
- Additional assessment rules
- Expanded medical coverage
- Additional urgency levels
- Improved AI capabilities
- Additional medical knowledge sources
- More advanced assessment models
- Additional user context
- Future personalization capabilities

Future extensions shall not compromise the safety and traceability requirements of the system.

---

# 40. Document Status

**Current Status:** Draft — Team Review Required

This document defines the required behavior and responsibilities of the Assessment Engine.

The final implementation architecture, including the decision between Rule-Based, AI-Based, Hybrid, or another suitable approach, shall be determined by the SymptoSense team after reviewing this document and the related system requirements.