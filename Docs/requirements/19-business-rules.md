# Business Rules

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-15

---

## 1. Purpose

This document defines the business rules that govern the behavior, decision-making, and operational constraints of SymptoSense.

These rules define how the system should behave independently of the specific UI, AI model, or implementation technology.

The purpose of these rules is to ensure that SymptoSense provides a consistent, safe, predictable, and traceable assessment experience.

The Business Rules define the core constraints and decisions related to:

- User and session management
- Guest and authenticated user behavior
- Assessment lifecycle
- Symptom input and evaluation
- Question and information gathering
- Assessment Engine behavior
- Symptom clustering
- Safety and urgency handling
- Assessment possibilities and results
- Medical knowledge usage
- Assessment history
- Personalization
- Chronic conditions
- 3D body interaction
- Account and data management

---

## 2. Business Rule Principle

The system must prioritize user safety and assessment integrity over convenience.

Business Rules shall take precedence over UI behavior, AI-generated suggestions, or implementation-specific decisions.

The Assessment Engine shall be the authoritative component for assessment decisions, while AI shall operate within the boundaries defined by the system's AI Requirements.

---

## 3. Rule Structure

Each Business Rule shall have:

- A unique identifier (`BR-XXX`)
- A descriptive name
- A clear and testable statement
- Additional constraints or conditions when required

Business Rules should be traceable to the following system artifacts:

Business Rule
↓
Functional Requirement
↓
Use Case
↓
Implementation
↓
Test Case

This traceability ensures that every important business decision can be implemented and validated.

---

# 4. User & Session Rules

## BR-001 — First Session as Guest

A user may start the first session as a Guest without providing any personal information.

The initial Guest experience shall allow the user to interact with the core assessment flow without requiring account creation.

---

## BR-002 — Guest Assessment Access

A Guest user shall be allowed to complete a full assessment during the initial Guest Session.

The Guest assessment flow shall not be limited to a preview or a restricted subset of the assessment.

The expected flow is:

Guest
→ Symptoms
→ Questions
→ Assessment
→ Result

---

## BR-003 — No Personal Information for Initial Assessment

The Initial Guest Assessment shall not require the user to provide personal information, including:

- Name
- Email
- Phone Number
- Date of Birth
- Account

This rule does not prevent the system from storing technically necessary session or assessment data. Data storage and privacy requirements shall be defined separately under Data & Privacy Rules.

---

## BR-004 — Authentication After Initial Session

After completing the Initial Guest Session, the user shall authenticate before accessing features that require persistent user data.

The expected flow is:

First Visit
→ Guest
→ Assessment
→ Result
→ Continue Using Persistent Features
→ Login / Sign Up

---

## BR-005 — Guest Session Continuity

If a Guest user completes an assessment and subsequently creates an account, the completed Guest Assessment shall be associated with the newly created user account.

The system shall preserve the assessment result rather than requiring the user to repeat the assessment after registration.

The expected flow is:

Guest Assessment
→ Create Account
→ Associate Guest Session
→ Registered User History

---

## BR-006 — Guest Assessment Limit

A Guest user is allowed to complete one assessment only during the initial Guest Session.

Any additional assessment requires authentication.

The Guest assessment limit applies only to the initial Guest experience and does not represent a general limit on the number of assessments a registered user may create.

After authentication, the user may create additional assessments subject to any future account, subscription, rate-limiting, or system-capacity rules defined separately.

---

## BR-007 — Session and Assessment Separation

A Session and an Assessment shall be treated as separate system concepts.

A single authenticated user session may contain multiple assessments.

For example:

User Session
├── Assessment #1
├── Assessment #2
└── Assessment #3

The Guest initial experience shall remain limited to one assessment according to BR-006.

---

## BR-008 — Assessment Ownership

Every assessment created by an authenticated user shall be associated with that user.

Assessment ownership shall provide the foundation for:

- Assessment History
- Tracking
- Personalization
- Chronic Conditions
- Future Follow-ups

---

## BR-009 — Authentication Enforcement

If a feature requires an authenticated user account, the system shall prevent Guest users from bypassing the authentication requirement through either the UI or API.

Authentication requirements shall be enforced at the backend level and shall not depend solely on frontend restrictions.

---

# 5. Assessment Rules

## BR-010 — Multiple Symptoms per Assessment

A single assessment may contain multiple symptoms provided by the user.

The system shall evaluate the symptoms as a combined clinical context rather than requiring the user to select a single primary symptom.

The user shall be allowed to provide all symptoms they are experiencing.

The AI shall extract and organize the reported symptoms and context.

The Assessment Engine shall receive the complete structured symptom set and determine how the symptoms interact during the assessment.

The user shall not be required to determine which symptom is the primary symptom.

The system shall avoid unnecessary questions even when the user provides multiple symptoms.

The AI shall not determine the diagnosis or final medical assessment.

The expected flow is:

User describes all symptoms
→ AI Understanding
→ Structured Symptoms
→ Assessment Engine
→ Assessment
→ Result

---

## BR-011 — Assessment Started

An assessment shall not be considered started merely because the user opens the Symptom Checker.

The assessment shall officially enter the Started state when the user provides the first meaningful information about their symptoms or health concern.

The expected flow is:

Open Symptom Checker
→ Idle
→ User provides first symptom/input
→ Assessment Started

Opening the feature and leaving without providing meaningful information shall not create a started assessment.

---

## BR-012 — Assessment In Progress

After an assessment has started, it shall remain in the `In Progress` state while the system is still collecting the information required for the assessment.

The expected flow is:

Started
→ In Progress
→ More Questions
→ More Answers

---

## BR-013 — Assessment Completion

An assessment shall be considered `Completed` when the Assessment Engine determines that the minimum required information has been collected to produce an assessment result.

Completion shall not require the user to answer every question that could potentially be asked.

Therefore:

Completion ≠ Answering every possible question

Instead:

Completion = Required information is sufficient for the Assessment Engine

The system shall avoid extending the assessment unnecessarily once sufficient information has been collected.

---

## BR-014 — Required Information

The Assessment Engine shall be responsible for determining what information is required to complete an assessment.

When required information is missing, the Assessment Engine may request additional information through appropriate questions.

The AI may assist by:

- Formulating questions in natural language.
- Understanding user responses.
- Structuring the provided information.

However, the AI shall not independently determine the medical requirements needed to complete the assessment.

The expected flow is:

User Symptoms
→ Assessment Engine
→ Missing Required Information
→ Questions
→ User Answers
→ Assessment Engine

---

## BR-015 — User Can Modify Answers

The user shall be allowed to modify previously provided answers before the assessment is completed.

If modifying an answer affects the information required, subsequent questions, symptom relationships, safety evaluation, or assessment result, the Assessment Engine shall reevaluate the assessment based on the updated information.

The expected flow is:

Question
→ Answer
→ Next Questions
→ Edit Previous Answer
→ Recalculate / Reevaluate Assessment

---

## BR-016 — Assessment Draft Persistence

For authenticated users, an incomplete assessment shall be automatically saved as a draft so the user can resume it later from the point where they left.

The expected flow is:

Registered User
→ Start Assessment
→ Answer Questions
→ Auto-save Draft
→ User Leaves
→ Assessment = Draft
→ Return Later
→ Resume Assessment
→ Continue From Last Saved State
→ Complete Assessment

A Draft assessment:

- Shall not be considered a Completed Assessment.
- Shall not appear as a Completed Assessment in the user's history.
- Shall not have a final assessment result.
- Shall allow the user to modify previously provided answers.
- Shall allow the Assessment Engine to reevaluate the required questions when the user resumes the assessment.
- Shall not be counted as a completed assessment.

For Guest users, incomplete assessment persistence shall be handled separately according to the Guest Session rules and shall not create a persistent completed assessment.

---

## BR-017 — Result Generation

An assessment result shall not be considered final until:

1. The Assessment Engine determines that the required information is sufficient.
2. Applicable Safety Rules have been evaluated.
3. The Assessment Engine performs the assessment.
4. An Assessment Result is generated.

The expected flow is:

Information Sufficient
→ Safety Evaluation
→ Assessment Engine
→ Assessment Result
→ AI Explanation
→ User

The AI may assist in explaining and presenting the result, but it shall not replace the Assessment Engine as the authority responsible for the assessment.

---

# 6. Question & Information Gathering Rules

## BR-018 — Question Authority

The Assessment Engine shall be responsible for determining the information and questions required to complete an assessment.

The AI shall not independently decide that a particular medical question must be asked.

The expected flow is:

Structured Symptoms
→ Assessment Engine
→ Required Information
→ AI
→ Natural-language Question
→ User

The Assessment Engine determines what information is required, while the AI assists with communicating the required question to the user.

---

## BR-019 — Dynamic Questioning

Assessment questions shall not be implemented as one fixed questionnaire that is asked to every user.

The Assessment Engine shall determine questions dynamically based on the information already available within the current assessment context.

For example, if the user already states:

"I have had abdominal pain for 3 days."

The system shall not ask:

"When did the pain start?"

Instead, the Assessment Engine shall identify the next required information.

The expected flow is:

Known Information
→ Determine Missing Information
→ Ask Relevant Question
→ Update Assessment Context
→ Determine Next Question

---

## BR-020 — No Repeated Questions

The system shall not ask the user for information that has already been obtained unless:

- The existing answer is unclear.
- The information is contradictory.
- The information requires confirmation.
- The user has modified related information.

For example, if the user already states:

"The pain started one week ago."

The system shall not ask again:

"When did the pain start?"

unless one of the conditions above applies.

---

## BR-021 — Natural Language Answers

The user shall not be required to answer questions using a predefined format unless the question explicitly requires a structured value.

Users may respond using natural language.

For example, for:

"When did the symptoms start?"

The user may answer:

- "Two days ago."
- "Around the beginning of the week."
- "I don't remember exactly, but around 3 or 4 days ago."

The AI shall interpret the response and convert the relevant information into the structured assessment representation.

---

## BR-022 — Ambiguous Answers

If a user response is ambiguous and the ambiguity may materially affect the assessment, the system shall not assume a value.

The system shall request clarification when required.

For example:

User:
"The pain is severe."

If the assessment requires a defined severity scale, the system shall request clarification rather than converting the response into an assumed numeric value.

The expected flow is:

Unknown / Ambiguous
→ Clarification
→ Structured Information

---

## BR-023 — Conflicting Information

If the user provides conflicting information, the system shall identify the conflict and request clarification before relying on the affected information.

For example:

First statement:
"The pain started two days ago."

Later statement:
"The pain has been there for a week."

The system shall not select one value arbitrarily.

Instead, it should request clarification such as:

"You mentioned that the pain started two days ago, but later said it has been present for a week. Which duration is closer to the correct one?"

---

## BR-024 — User Can Skip Optional Information

The system shall distinguish between Required Information and Optional Information.

If information is optional, the user shall be allowed to skip the corresponding question.

If information is required for the Assessment Engine to safely complete the assessment, the system may require the user to provide or explicitly decline the information according to the applicable assessment rules.

The system shall not classify every potentially useful question as Required.

The goal is to obtain the minimum sufficient information required for the assessment.

---

## BR-025 — Minimum Sufficient Information

The Assessment Engine shall seek the minimum amount of information required to perform an appropriate and safe assessment.

The system shall not assume that collecting more information is always better.

The objective is:

Minimum Sufficient Information

rather than:

Maximum Possible Information

The expected flow is:

User Input
→ Known Symptoms
→ Required Information
→ Question 1
→ Update Context
→ Required Information
→ Question 2
→ Enough Information
→ Assessment

Once sufficient information is available, the system shall not continue asking unnecessary questions.

---

## BR-026 — Question Relevance

Every question presented to the user shall have a clear relationship to the current assessment context.

The system shall not use a generic questionnaire that asks the same broad set of questions to every user regardless of their symptoms.

Questions shall be derived from:

- Current Symptoms
- Current Context
- Missing Required Information
- Applicable Assessment Rules

---

## BR-027 — AI Question Generation

The Assessment Engine shall determine what information needs to be collected, while the AI shall assist in determining how to communicate the question naturally to the user.

For example:

Assessment Engine:

Required Information:
Pain severity

AI:

"If we rate the pain from 0 to 10, where 0 means no pain and 10 means the worst pain you can imagine, approximately what number would you give it?"

Therefore:

What to ask → Assessment Engine

How to ask → AI

This separation shall be maintained as an architectural boundary.

---

## BR-028 — User Correction and Information Precedence

If the user corrects or updates previously provided information, the latest confirmed user-provided information shall become the active value for the current assessment context.

The Assessment Engine shall reevaluate the assessment context when the correction may affect:

- Required information
- Subsequent questions
- Symptom relationships
- Safety evaluation
- Assessment result

Previous values shall not remain active as current assessment inputs unless they represent separate historical information.

The expected flow is:

Original Information
→ User Correction
→ Updated Information
→ Assessment Context Updated
→ Re-evaluate Required Information
→ Continue Assessment

---

## BR-029 — Multi-Symptom Questioning

Because a single assessment may contain multiple symptoms, the questioning process shall consider the complete symptom set rather than automatically evaluating each symptom through an isolated questionnaire.

For example:

Symptoms:
- Cough
- Fever
- Vomiting

The system shall not necessarily execute:

Cough Questions
→ Fever Questions
→ Vomiting Questions

The Assessment Engine may determine that a single question can provide useful information for evaluating multiple symptoms or their relationships.

The goal is:

Avoid unnecessary questioning while preserving relevant clinical context.

---

## BR-030 — Assessment Cannot Assume Missing Information

If the user has not provided a piece of information and has not answered a question related to that information, the system shall not automatically interpret the missing information as:

- No
- False
- Normal
- Not Present

The information shall remain `Unknown` unless the applicable question or business rule explicitly defines another interpretation.

For example:

If the system asks:

"Do you have a fever?"

and the user does not answer, the system shall not interpret the result as:

"No fever."

The information shall remain:

Fever = Unknown

This rule shall prevent the Assessment Engine from building an assessment on information that the user never provided.

---

# 7. Symptom Input Rules

## BR-031 — Structured Symptom Input

User-provided symptoms shall be converted into a structured representation before being evaluated by the Assessment Engine.

The structured representation may include, when applicable:

- Symptom
- Location
- Onset
- Duration
- Severity
- Frequency
- Pattern
- Associated symptoms
- Relevant context
- User-provided conditions
- Other clinically relevant attributes

The AI may assist in extracting and structuring this information from natural user input.

The Assessment Engine shall consume the structured representation rather than relying directly on unstructured user text.

---

## BR-032 — Natural Language Symptom Input

The user shall be allowed to describe symptoms using natural language rather than being required to use predefined medical terminology.

The system shall attempt to interpret common variations, colloquial expressions, and non-medical descriptions.

The original user meaning shall be preserved during normalization.

---

## BR-033 — Ambiguous Symptom Handling

If the user's description is ambiguous and the ambiguity may materially affect the assessment, the system shall request clarification before relying on the ambiguous information.

The AI may identify and communicate the ambiguity, but the Assessment Engine shall determine whether the missing distinction is required for the assessment.

---

## BR-034 — Missing Information Handling

If required information is missing, the Assessment Engine shall determine whether:

1. The assessment can continue without the missing information.
2. Additional information must be requested.
3. The assessment cannot safely continue without the information.

The system shall not invent, assume, or silently fill clinically relevant missing information.

---

## BR-035 — Multiple Input Methods

The system may support multiple methods for providing symptom information, including:

- Text
- Voice
- Structured input
- 3D body interaction

Regardless of the input method, the information shall be normalized into a common structured representation before being evaluated by the Assessment Engine.

---

# 8. Assessment Engine Rules

## BR-036 — Assessment Engine Authority

The Assessment Engine shall be the authoritative component responsible for determining the final assessment.

AI-generated output shall not directly determine the final assessment.

The Assessment Engine shall operate according to validated medical knowledge, business rules, safety rules, and structured assessment data.

---

## BR-037 — AI Assessment Boundary

AI shall not independently diagnose the user or determine the final assessment.

AI responsibilities may include:

- Understanding natural language.
- Extracting symptoms.
- Structuring information.
- Generating clarification questions according to Engine requirements.
- Explaining assessment results.
- Converting technical information into user-friendly language.

Final assessment decisions shall remain under the control of the Assessment Engine.

---

## BR-038 — Assessment Engine and Medical Knowledge

The Assessment Engine shall use an approved Medical Knowledge Base and validated assessment rules as the primary sources for assessment decisions.

The AI model shall not be treated as the authoritative source of medical truth.

Medical knowledge used by the Assessment Engine shall be versioned and maintainable so that changes can be tracked and validated.

---

## BR-039 — Additional Information Requirement

The Assessment Engine may request additional information when the currently available data is insufficient to complete the assessment safely or accurately.

Questions shall be limited to information that can materially contribute to the assessment.

The system shall avoid unnecessary or repetitive questioning.

---

## BR-040 — Conflicting Information

If the system detects conflicting information within the current assessment, the conflict shall be identified and resolved before relying on the affected information.

The system may ask the user for clarification when necessary.

The Assessment Engine shall not silently choose between materially conflicting values without an applicable rule or sufficient evidence.

---

## BR-041 — Assessment Evaluation Scope

The Assessment Engine shall evaluate all relevant structured information available within the current assessment context.

The engine shall consider:

- Reported symptoms
- Symptom relationships
- Relevant context
- User-provided conditions
- Required assessment information
- Safety information
- Applicable medical rules

The engine shall not evaluate symptoms in isolation when their relationships may materially affect the assessment.

---

## BR-042 — Assessment Result Integrity

The final assessment result shall be generated only from information and rules available to the Assessment Engine at the time of evaluation.

AI-generated explanations shall not alter the underlying assessment result.

Any change to clinically relevant input after assessment completion shall trigger reassessment when required by the applicable rules.

---

# 9. Multiple Symptoms & Cluster Rules

## BR-043 — Multiple Symptoms Evaluation

A single assessment may contain multiple symptoms.

The Assessment Engine shall evaluate all reported symptoms within the same assessment context and shall not automatically create separate assessments for individual symptoms.

---

## BR-044 — Symptom Relationships

The Assessment Engine shall evaluate the relationships between identified symptoms and symptom clusters.

A relationship may be classified as:

- Related
- Potentially Related
- Independent
- Unknown

The engine shall not assume a clinical relationship between symptoms without sufficient supporting evidence or applicable medical rules.

---

## BR-045 — Relationship-Based Assessment Evaluation

Symptom clusters shall be considered intermediate analytical units rather than final assessment units.

Before performing the assessment, the Assessment Engine shall evaluate the relationships between identified clusters.

- Related clusters shall be evaluated together as one clinical context.
- Independent clusters shall be evaluated separately.
- Unknown relationships shall remain unresolved until sufficient information becomes available.
- Cluster relationships may be reevaluated when new information is provided.

The final user-facing output shall remain a single assessment response, regardless of the number of clusters involved.

For example, if two clusters are determined to be related, they shall be evaluated together as one clinical context.

If two clusters are determined to be independent, each cluster shall be evaluated separately and their results shall later be combined into the single assessment response.

---

## BR-046 — Maximum Cluster Limit

A single assessment shall support a maximum of 6 symptom clusters.

If the user's reported symptoms result in more than 6 clinically distinct clusters, the Assessment Engine shall prioritize the most relevant clusters for the current assessment and defer the remaining clusters to a subsequent assessment.

Cluster prioritization shall consider, at minimum:

- Safety / Red Flags
- Urgency
- Clinical relevance
- Availability of sufficient information
- Assessment value

Safety and urgency shall take priority over other prioritization factors.

Deferred clusters shall not be discarded.

They shall remain associated with the user's assessment context and may be evaluated in a subsequent assessment.

The system shall inform the user that additional symptoms remain and can be assessed separately after the current assessment.

The cluster limit shall not prevent the system from processing newly reported safety-critical information.

Safety-related information shall always take priority and may trigger reassessment of the current assessment context.

The 6-cluster limit applies to symptom clusters, not individual symptoms.

A single cluster may contain multiple related symptoms.

---

# 10. Assessment Possibility Rules

## BR-047 — Possibility Generation

The Assessment Engine shall generate possible explanations for each evaluated clinical context using only applicable medical knowledge and validated assessment rules.

Generated possibilities shall be evaluated based on:

- Compatibility with the reported symptoms and available information.
- Supporting evidence.
- Contradicting evidence.
- Clinical significance.
- Potential severity if the possibility is the actual condition.
- Urgency and safety relevance.

The Assessment Engine shall not generate unsupported possibilities solely through language-model inference.

The ranking of possible explanations shall consider both their compatibility with the available information and the potential clinical risk associated with missing the possibility.

A highly severe possibility shall not automatically be ranked above a more likely possibility solely because of its severity.

Safety-critical possibilities that require attention but do not meet the criteria for being presented as a normal possible explanation may be handled separately as Safety Alerts or Urgency Warnings.

---

## BR-048 — Possibility Presentation Limit

The Assessment Engine shall present a maximum of 3 possible explanations for each evaluated clinical context.

The engine shall not generate additional weak or unsupported possibilities solely to reach the maximum limit.

If fewer than 3 relevant possibilities are supported by the available information, only the supported possibilities shall be presented.

Examples:

- 1 supported possibility → Present 1.
- 2 supported possibilities → Present 2.
- 3 or more supported possibilities → Present the top 3 according to the applicable prioritization rules.

The maximum limit of 3 applies only to user-facing Possible Explanations.

Safety Alerts and Urgency Warnings shall be presented separately and shall not count toward the 3-possibility limit.

Possible explanations shall not be presented as confirmed diagnoses.

---

# 11. Safety & Urgency Rules

## BR-049 — Safety Evaluation

Every assessment shall undergo a Safety Evaluation before a final assessment result is issued.

The Assessment Engine shall be responsible for performing the Safety Evaluation using validated medical rules and the approved Medical Knowledge Base.

The AI shall not independently determine whether the user's condition is safe or dangerous.

---

## BR-050 — Red Flag Detection

The Assessment Engine shall evaluate the user's symptoms and available assessment context for Red Flags associated with potentially serious or urgent conditions.

A Red Flag may be identified based on:

- A specific symptom
- A combination of symptoms
- Symptom severity
- Symptom duration
- User-provided context
- A combination of the above

The presence of a Red Flag shall not automatically constitute a diagnosis. It may instead affect the Urgency Level of the assessment.

---

## BR-051 — Safety Has Priority

Safety Evaluation shall have higher priority than the ranking of Possible Explanations.

If a possibility has relatively low likelihood but represents a significant safety risk if it were the actual condition, the system shall account for that risk appropriately.

Safety-related decisions shall not be suppressed solely because a possibility has a lower estimated likelihood.

---

## BR-052 — Urgency Level

The Assessment Engine shall classify the assessment into one of two urgency levels based on the applicable Safety Rules:

- **Urgent:** The user may require prompt or immediate medical attention.
- **Non-Urgent:** No applicable safety rule indicates that urgent medical attention is required based on the available information.

The Urgency Level shall be determined by the Assessment Engine and shall not be determined by the AI.

The basic flow shall be:

    Assessment
       ↓
    Safety Evaluation
       ↓
    Urgency

If the result is:

    Urgent
       ↓
    Urgent Recommendation

    Non-Urgent
       ↓
    Normal Assessment Result

The two-level model is intended for the initial MVP and may be expanded in future versions if additional urgency levels are required.

---

## BR-053 — Urgent Safety Escalation

If the Assessment Engine identifies a Safety condition that requires the assessment to be classified as `Urgent`, the system shall escalate the Urgency Level immediately.

The system shall not ask additional questions that are not necessary for the Safety or Urgency decision.

However, if additional information is required to make an accurate Safety or Urgency decision, the Assessment Engine may request the required safety-related information before issuing the final Urgency decision.

The flow shall be:

    User Information
          ↓
    Safety Evaluation
          ↓
    Is Urgency Decision Clear?
          │
       ┌──┴──┐
      Yes    No
       │      │
       ↓      ↓
    Urgent   Required Safety Question
       │      │
       │      ↓
       │   Re-evaluate
       │      │
       └──┬───┘
          ↓
    Urgency Decision
          ↓
    User Recommendation

Additional questions shall only be asked when they provide information that is directly relevant to the Safety or Urgency decision.

---

## BR-054 — Safety Before Completion

Safety Evaluation shall be performed throughout the assessment and shall not be limited to the point at which the assessment reaches the `Completed` state.

If Safety-critical information is provided while the assessment is still in progress, the Assessment Engine shall evaluate that information immediately.

The flow may be:

    Assessment In Progress
            ↓
    New User Information
            ↓
    Safety Evaluation
            ↓
    Potential Safety Issue
            ↓
    Urgency Decision

The system shall not wait for the full assessment to be completed before evaluating a potentially serious safety condition.

---

## BR-055 — Safety Override

If new information increases the assessed level of urgency, the new Safety Evaluation shall override the previous lower urgency classification.

For example:

    Initial Assessment
          ↓
    Urgency = Non-Urgent
          ↓
    User Adds New Information
          ↓
    Safety Re-evaluation
          ↓
    Urgency = Urgent

The Assessment Engine shall not continue using a previous `Non-Urgent` classification after new information establishes that an `Urgent` classification is required.

---

## BR-056 — No False Reassurance

The system shall not provide definitive reassurance that the user's condition is not serious when the available information is insufficient to reasonably exclude a potentially serious condition.

The system shall not provide statements such as:

- "You are definitely fine."
- "There is nothing serious."

Instead, the system shall communicate the appropriate level of uncertainty and provide the recommended next step according to the applicable Safety Rules.

---

## BR-057 — Safety Alerts Separate from Possibilities

Safety Alerts and Urgency Warnings shall be presented separately from Possible Explanations.

For example:

    Assessment Result

    Possible Explanations:
    1. ...
    2. ...
    3. ...

    Safety Alert:
    Urgent medical attention is recommended because ...

A Safety Alert or Urgency Warning shall not count toward the maximum of 3 Possible Explanations defined by BR-048.

---

## BR-058 — Safety Information Cannot Be Suppressed

Any information that may affect Safety or Urgency shall be included in the Safety Evaluation.

Safety-relevant information shall not be ignored because:

- It does not support the top Possible Explanations.
- It reduces assessment confidence.
- It makes the result more complex.
- It was provided late during the assessment.

Safety-relevant information shall always be considered by the Assessment Engine.

---

## BR-059 — User-Facing Safety Communication

When an assessment is classified as `Urgent`, the system shall provide a clear, direct, and understandable safety recommendation to the user.

The responsibilities shall remain separated as follows:

    What is the urgency?
            ↓
    Assessment Engine

    How is it communicated?
            ↓
    AI

The Assessment Engine determines the Urgency Level, while the AI may assist in communicating that decision in natural and understandable language.

---

## BR-060 — No Diagnosis From Red Flag

The presence of a Red Flag or Safety condition shall not be treated as confirmation of a specific diagnosis.

The relationship shall be:

    Red Flag Detected
          ≠
    Confirmed Diagnosis

A Red Flag shall be used to determine Safety and Urgency requirements rather than to establish that the user has a particular disease or condition.

---

## BR-061 — Safety Reassessment

If the user adds new information or modifies existing information that may affect Safety or Urgency, the Assessment Engine shall perform a new Safety Evaluation.

For example:

    Initial Information
          ↓
    Safety Evaluation
          ↓
    Urgency = Non-Urgent
          ↓
    User Adds New Symptom
          ↓
    Safety Re-evaluation
          ↓
    Urgency May Change

This rule shall work together with:

- BR-015 — User Can Modify Answers
- BR-028 — User Correction and Information Precedence
- BR-054 — Safety Before Completion
- BR-055 — Safety Override

---

## BR-062 — Urgency Result Priority

When an assessment is classified as `Urgent`, the Urgency Status and Recommended Action shall be presented before Possible Explanations in the user-facing assessment result.

The expected result structure shall be:

    Assessment Result
           ↓
    ⚠️ Urgent Status
           ↓
    Recommended Action
           ↓
    Possible Explanations
           ↓
    Additional Information

The system shall prioritize communicating the required action before presenting non-urgent explanatory information.

Possible Explanations shall not delay or obscure an Urgent Safety Recommendation.

---

# 12. Assessment Result Rules

## BR-063 — Single Assessment Result

Each assessment shall produce one Assessment Result for the user.

Even when an assessment contains multiple symptoms or multiple clusters, the user shall receive a single consolidated Assessment Result rather than separate results for each component.

If the assessment contains independent clusters, the Assessment Engine may evaluate each cluster separately internally and then combine the outputs into one user-facing Assessment Result.

The general structure shall be:

    Assessment
     ├── Cluster 1
     ├── Cluster 2
     └── Cluster 3
            ↓
      One Assessment Result

---

## BR-064 — Result Reflects Current Assessment

The Assessment Result shall reflect the latest valid version of the assessment information.

If the user modifies an answer or adds a symptom before the assessment is completed, the Assessment Context shall be updated and the assessment shall be re-evaluated as required.

The flow shall be:

    Previous Information
          ↓
    User Correction / New Information
          ↓
    Updated Assessment Context
          ↓
    Re-evaluation
          ↓
    Final Assessment Result

A previous result shall not be considered representative of the user's current assessment state after relevant information has been changed.

---

## BR-065 — Assessment Result Components

The Assessment Result shall contain the following components when applicable:

- Urgency Status
- Recommended Next Step
- Possible Explanations
- Supporting Information
- Uncertainty / Limitations
- Safety Information, when applicable

Not every component is required to be displayed in every assessment.

The displayed components shall depend on the outcome of the Assessment Engine and the applicable Business Rules.

---

## BR-066 — Possible Explanations Limit

The Assessment Result shall display a maximum of three Possible Explanations to the user.

The Assessment Engine may internally evaluate additional possibilities when required for the assessment, but only the highest-priority applicable possibilities shall be presented to the user.

The maximum user-facing structure shall be:

    Possible Explanations

    1. Possibility A
    2. Possibility B
    3. Possibility C

Safety Alerts and Urgency Warnings shall not count toward this limit, as defined by BR-057.

---

## BR-067 — Possibility Ranking

Possible Explanations shall be ranked by the Assessment Engine based on the applicable validated medical rules and available assessment information.

Ranking shall not be based on likelihood alone.

The ranking may consider:

- Estimated likelihood
- Severity or potential harm if the possibility is correct
- Relevance to the user's reported symptoms
- Available supporting information

A possibility with a lower estimated likelihood may receive a higher presentation priority when failing to consider it would represent a significant safety concern.

The final ranking shall remain subject to the applicable Safety Rules.

---

## BR-068 — No Guaranteed Diagnosis

The Assessment Result shall not present any Possible Explanation as a confirmed diagnosis.

Possible Explanations shall be communicated as assessment possibilities rather than definitive medical conclusions.

Appropriate wording may include:

- Possible explanation
- May be consistent with
- Could be related to
- Requires medical evaluation

The system shall not present statements such as:

    "You have X."

unless a future product version explicitly introduces a medically validated workflow that supports confirmed diagnosis, which is outside the scope of the current MVP.

---

## BR-069 — Result Must Reflect Uncertainty

If the available information is insufficient or there is meaningful uncertainty affecting the assessment, the Assessment Result shall communicate the relevant uncertainty.

The system shall not present a level of confidence or certainty that exceeds what is supported by the available information and assessment process.

The purpose of this rule is to prevent the user from interpreting an assessment result as more medically certain than it actually is.

---

## BR-070 — Recommended Next Step

Each Assessment Result shall provide an appropriate Recommended Next Step based on the Assessment Engine outcome and applicable Safety Rules.

Examples include:

    Urgent
       ↓
    Seek urgent medical attention

    Non-Urgent
       ↓
    Appropriate non-urgent next step

The Assessment Engine and applicable Business Rules shall determine the Recommended Next Step.

The AI may assist in communicating the recommendation in natural and understandable language but shall not independently change the recommended action.

---

## BR-071 — Result Explanation

The Assessment Result shall provide an understandable explanation of why the presented Possible Explanations were considered, when supporting information is available.

The explanation shall not present a Possible Explanation as a confirmed diagnosis.

For example, the system may communicate:

    "This possibility was considered because your reported symptoms are consistent with..."

The facts and assessment factors used in the explanation shall originate from the Assessment Engine and the approved Medical Knowledge Base.

The AI may assist in generating the natural-language explanation but shall not introduce unsupported medical facts or assessment conclusions.

---

## BR-072 — Result Consistency

The user-facing Assessment Result shall remain consistent with the decisions and data produced by the Assessment Engine.

The AI shall not:

- Add a diagnosis that was not provided by the Assessment Engine.
- Remove an `Urgent` status.
- Change `Urgent` to `Non-Urgent`.
- Add a Possible Explanation that was not approved by the Assessment Engine.
- Change the meaning of the Recommended Next Step.
- Introduce unsupported medical conclusions.

The responsibility boundary shall remain:

    Assessment Engine
          ↓
    Decision / Result Data

    AI
          ↓
    Explanation / Communication

The AI shall communicate the Assessment Engine output without changing its meaning.

---

## BR-073 — Result Generation Authority

The Assessment Engine shall be the authoritative source for the data used to generate the Assessment Result.

The AI shall not independently create the Assessment Result from raw user input.

The expected flow shall be:

    User Input
        ↓
    AI Understanding
        ↓
    Structured Information
        ↓
    Assessment Engine
        ↓
    Assessment Decision
        ↓
    Result Data
        ↓
    AI Explanation
        ↓
    User-Facing Assessment Result

The Assessment Engine shall remain responsible for the assessment decision, while the AI may assist with natural-language communication.

---

## BR-074 — Result Version Integrity

Each Assessment Result shall be associated with the specific version of the Assessment that was evaluated.

The relationship shall be:

    Assessment Version
          ↓
    Assessment Engine
          ↓
    Assessment Result

If the assessment information changes after a Result has been generated, the system shall not silently modify the existing Result.

Instead, the updated Assessment shall undergo the required re-assessment process and generate a new Result when applicable.

Previous Results shall remain traceable to the Assessment version from which they were generated.

This rule supports:

- Assessment History
- Result Traceability
- Auditability
- Re-assessment
- Historical Result Integrity

---

# 13. Medical Knowledge & Assessment Engine Rules

## BR-075 — Medical Knowledge Source Authority

The Assessment Engine shall rely only on an approved Medical Knowledge Base as the source of medical rules used during assessment.

The AI shall not be considered an independent medical source for making Assessment decisions.

---

## BR-076 — Validated Medical Rules

Every medical rule used by the Assessment Engine to make an assessment decision shall be:

- Documented.
- Traceable to an approved medical source.
- Reviewed before being used in production.
- Associated with the version of the Medical Knowledge Base in which it is defined.

A rule shall not be added to the Assessment Engine solely because an AI model indicates that the rule is medically correct.

---

## BR-077 — Assessment Engine Authority

The Assessment Engine shall be responsible for applying the approved Medical Rules and making the final Assessment Decision.

The AI may:

- Understand user input.
- Extract symptoms.
- Convert natural language into structured data.
- Generate questions based on Engine requirements.
- Explain assessment results.

The AI shall not replace the Assessment Engine in making assessment decisions.

---

## BR-078 — Structured Input Requirement

The Assessment Engine shall not rely directly on raw natural-language input from the user.

User input shall first be converted into Structured Assessment Data before being evaluated by the Assessment Engine.

For example:

    User:
    "I've had severe back pain for about a week."

            ↓

    AI Understanding

            ↓

    Structured Data:
    Symptom = Back Pain
    Severity = Severe
    Duration = 7 days

            ↓

    Assessment Engine

---

## BR-079 — AI Output Validation

Any Structured Assessment Data extracted by the AI shall be validated before being provided to the Assessment Engine.

For example, if the supported severity scale is `0–10` and the AI extracts:

    Severity = 15

the system shall not pass the value directly to the Assessment Engine.

The expected flow shall be:

    AI Output
       ↓
    Validation
       ↓
    Valid?
      / \
    Yes  No
     ↓    ↓
    Engine  Clarification / Correction

Invalid, unsupported, or inconsistent AI output shall not be used for assessment decisions until it has been resolved.

---

## BR-080 — No Unsupported Inference

The Assessment Engine shall not treat information that was not provided by the user as a confirmed fact.

For example, if the user states:

    "I have a cough."

the system shall not automatically infer:

    Fever = No
    Smoking = No
    Chest Pain = No

Such information shall remain:

    Unknown

unless it is explicitly provided or reliably obtained through the assessment process.

---

## BR-081 — Knowledge Base Versioning

The Medical Knowledge Base shall be versioned.

Each assessment shall be traceable to the specific Medical Knowledge Base version used during its evaluation.

For example:

    Assessment
       ↓
    Assessment Engine Version: 1.0
    Knowledge Base Version: 1.3
    Rules Version: 1.2
       ↓
    Assessment Result

---

## BR-082 — Knowledge Update Isolation

Updating the Medical Knowledge Base shall not silently modify previously generated Assessment Results.

For example:

    Old Assessment
         ↓
    Knowledge Base v1.0
         ↓
    Result A

Later:

    Knowledge Base v1.1

The previous Result shall remain associated with Knowledge Base v1.0.

If the assessment needs to be evaluated using the newer knowledge version, an explicit Re-assessment shall be performed.

---

## BR-083 — Rule Conflict Handling

If the Assessment Engine encounters conflicting Medical Rules, it shall not select a rule arbitrarily.

A defined precedence mechanism shall determine which rule takes priority.

For example:

    Safety Rule
         ↓
    Higher Priority

    General Assessment Rule
         ↓
    Lower Priority

If a conflict cannot be resolved using the defined precedence rules, the Assessment Engine shall not produce an unsupported or unreliable decision.

The system shall instead use the approved fallback mechanism for unresolved rule conflicts.

---

## BR-084 — Assessment Determinism

Given the same:

- Structured Assessment Data
- Medical Knowledge Base Version
- Rules Version
- Assessment Engine Version

the Assessment Engine shall produce the same Assessment Decision.

For example:

    Same Input
       +
    Same Rules
       +
    Same Knowledge Version
       ↓
    Same Assessment Decision

The AI may generate different wording when communicating the result, but differences in AI-generated wording shall not change the Assessment Decision.

---

## BR-085 — AI Cannot Override Engine Decision

The AI shall not modify, override, or contradict an Assessment Decision produced by the Assessment Engine.

For example, if the Assessment Engine determines:

    Urgency = Urgent

the AI shall not change it to:

    Urgency = Non-Urgent

Similarly, if the Assessment Engine provides:

    Possible Explanations:
    A
    B
    C

the AI shall not independently add:

    D

to the user-facing assessment.

---

## BR-086 — Assessment Traceability

Every Assessment Decision shall be traceable to the information and rules that produced it.

The system shall maintain traceability to:

- Structured Assessment Data
- Medical Knowledge Base Version
- Applied Rule Set
- Assessment Engine Version
- Assessment Result Version

The purpose of this rule is to allow the system to determine:

    Why was this Assessment Decision produced?

rather than only:

    What was the Assessment Decision?

---

## BR-087 — Medical Rule Scope

Every Medical Rule shall have a defined scope.

The scope shall specify, when applicable:

- Conditions under which the rule applies.
- Information required to apply the rule.
- Conditions under which the rule does not apply.
- Rule priority.
- Expected assessment impact.

Medical Rules shall not be implemented as overly broad assumptions such as:

    "If symptom X then condition Y."

Rules shall be defined within the appropriate medical context.

---

## BR-088 — Insufficient Evidence Handling

If the available information is insufficient to apply a Medical Rule reliably, the Assessment Engine shall not assume the missing information.

The system shall follow a flow such as:

    Missing Information
          ↓
    Unknown / Insufficient Evidence
          ↓
    Required Question
          OR
    Appropriate Fallback

This rule works together with:

- BR-024 — Required vs Optional
- BR-025 — Minimum Sufficient Information
- BR-030 — Missing ≠ No

---

## BR-089 — Knowledge Base Does Not Equal Diagnosis

The presence of medical information in the Medical Knowledge Base shall not mean that the Assessment Engine can confirm a diagnosis.

The Medical Knowledge Base may contain:

- Medical facts
- Relationships
- Assessment rules
- Risk factors
- Red Flags
- Possible Explanations
- Assessment criteria

The Assessment Result shall remain an assessment rather than a confirmed diagnosis, in accordance with BR-068.

---

# 14. Assessment History Rules

## BR-090 — Completed Assessment History

Every completed assessment performed by an authenticated user shall be available in the user's Assessment History.

Draft assessments shall not be considered completed assessments and shall not appear as completed Assessment History entries.

---

## BR-091 — Assessment History Ownership

An authenticated user shall only be able to access assessments associated with their own account.

A user shall not be able to access another user's Assessment History.

---

## BR-092 — Historical Result Integrity

A saved Assessment Result shall remain associated with the assessment and the versions used to produce that result.

Updates to the Medical Knowledge Base or Assessment Engine shall not automatically modify previously generated Assessment Results.

This rule is consistent with:

- BR-074 — Assessment Result Versioning
- BR-082 — Knowledge Update Isolation

---

## BR-093 — Historical Assessment Is Read-Only

A completed assessment stored in Assessment History shall not be directly modified.

If the user wants to provide different information from a previous assessment, the system shall treat it as a new assessment rather than modifying the original completed assessment.

This preserves the integrity of the user's assessment history.

---

## BR-094 — Historical Assessment Context

Information from previous assessments may be used as contextual information when performing a new assessment if that information is relevant to the current assessment.

Historical information shall not automatically be considered current information.

For example:

    Previous Assessment
          ↓
    Back Pain = 7 days
          ↓
    New Assessment
          ↓
    Back Pain = 3 weeks

The new information represents the current context.

---

## BR-095 — Historical Data Does Not Replace Current Input

Information stored in previous assessments shall not replace current information required by the Assessment Engine.

If the Assessment Engine requires current information, the system shall obtain that information from the user even if similar information exists in a previous assessment.

---

## BR-096 — Historical Assessment Does Not Determine New Assessment

A Possible Explanation, Assessment Result, or other assessment decision from a previous assessment shall not determine the result of a new assessment.

Each new assessment shall be evaluated according to the current information and the applicable Assessment Rules.

For example:

    Previous Assessment
          ↓
    Possible Explanation A
          ↓
    New Symptoms
          ↓
    New Assessment
          ↓
    Possible Explanation A
          OR
    Different Possible Explanations

The previous result may provide context but shall not determine the new result.

---

## BR-097 — Assessment History Ordering

Assessment History shall be presented in a clear chronological order so that the user can easily identify their most recent assessments.

The default ordering shall be from newest to oldest.

---

## BR-098 — Drafts and Assessment History

A Draft Assessment shall not be considered part of the user's Completed Assessment History.

An authenticated user shall be able to access and resume their Draft Assessment according to BR-016.

Once the assessment is completed, it shall transition from Draft to Completed and become part of the user's Assessment History.

---

# 15. Chronic Conditions Rules

## BR-099 — Chronic Condition as Context

A chronic condition provided by the user may be used as contextual information during an assessment when it is relevant to the current symptoms.

The presence of a chronic condition shall not automatically determine the assessment result.

---

## BR-100 — Chronic Condition Does Not Determine Cause

The system shall not assume that a current symptom is caused by an existing chronic condition solely because the user has that condition.

The current symptoms and applicable assessment rules shall be evaluated independently.

For example:

    Chronic Condition
          ↓
    Relevant Context
          ↓
    Current Symptoms
          ↓
    Assessment Engine
          ↓
    Assessment

---

## BR-101 — Current Symptoms Take Priority

When current symptom information conflicts with previously stored information about a chronic condition, the current user-provided information shall be treated as the current context.

The system shall not rely on outdated information when evaluating the current assessment.

---

## BR-102 — Chronic Condition Information May Affect Safety

A chronic condition may affect the Safety Evaluation when an applicable validated medical rule indicates that it changes the risk associated with the current symptoms.

The Assessment Engine shall determine whether the condition affects urgency.

The AI shall not independently determine this effect.

---

## BR-103 — Unknown Chronic Condition Information

If the system does not have sufficient information about a user's chronic condition, it shall treat the relevant information as `Unknown` rather than assuming that the condition is absent, controlled, or irrelevant.

---

## BR-104 — User Correction of Chronic Conditions

The user shall be able to correct or update previously provided chronic condition information.

When updated information affects the current assessment context, the Assessment Engine shall use the updated information.

---

## BR-105 — Chronic Conditions Are Not Diagnoses

A chronic condition stored or provided by the user shall be treated as user-provided medical history and shall not be considered a diagnosis made by SymptoSense.

The system shall not infer a chronic condition solely from symptoms unless the applicable assessment rules explicitly support such an assessment.

---

## BR-106 — Chronic Condition Relevance

A chronic condition shall only be considered in an assessment when it is relevant to the current assessment context.

The system should not introduce unrelated chronic conditions into the assessment or generate unnecessary questions about them.

---

# 16. Account & Data Management Rules

## BR-107 — Assessment Deletion

If the system provides assessment deletion functionality, an authenticated user may delete an assessment associated with their account.

Deleting an assessment shall not affect other assessments belonging to the same user.

---

## BR-108 — Account Deletion

When a user deletes their account, the system shall handle the data associated with that account according to the approved Data & Privacy Requirements.

Account deletion shall not leave personal data associated with the deleted account in a manner that violates the system's approved data and privacy policies.

---

## BR-109 — User Data Management

The system shall provide authenticated users with appropriate means to manage the data stored in their accounts, according to the approved Data & Privacy Requirements.

This may include:

- Viewing stored data
- Managing assessments
- Deleting assessments
- Managing user-provided medical information
- Deleting stored data where applicable