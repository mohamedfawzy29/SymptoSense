# AI Requirements

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-15

---

## 1. Purpose

This document defines the role, responsibilities, boundaries, requirements, and technology direction of Artificial Intelligence within SymptoSense.

The goal is to use AI to improve how users communicate their symptoms and understand assessment results while ensuring that the AI model is not responsible for making the final medical assessment.

---

## 2. AI Role in SymptoSense

AI is a supporting capability within the SymptoSense system.

The AI is responsible primarily for:

- Understanding natural-language user input.
- Extracting structured symptom information.
- Understanding contextual information.
- Detecting ambiguity or missing information.
- Supporting clarification.
- Processing voice-derived text.
- Explaining assessment results in user-friendly language.

The AI is **not** the primary medical decision-maker.

The final assessment is performed by the dedicated Assessment Engine.

---

## 3. AI Responsibility Boundary

The system separates AI responsibilities from assessment and safety responsibilities.

The general flow is:

    User
      ↓
    Text / Voice Input
      ↓
    AI Understanding Layer
      ↓
    Structured Symptoms
      ↓
    Assessment Engine
      ↓
    Assessment Result
      ↓
    AI Explanation
      ↓
    User

This separation ensures that the AI model does not independently determine the final assessment.

---

## 4. AI Capabilities

### 4.1 Natural Language Understanding

The system shall allow users to describe their symptoms using natural language.

Example:

> "بقالي يومين حاسس بوجع تحت بطني ناحية اليمين وبيزيد لما أتحرك."

The AI should extract relevant structured information such as:

    {
      "symptom": "pain",
      "location": {
        "region": "abdomen",
        "area": "lower",
        "side": "right"
      },
      "duration": "2 days",
      "trigger": "movement"
    }

The AI should preserve uncertainty when information is not explicitly provided.

It must not invent missing values.

---

### 4.2 Symptom Extraction

The AI shall be capable of extracting relevant symptom information from free-form user descriptions.

Potential information includes:

- Symptom type.
- Location.
- Duration.
- Onset.
- Severity.
- Frequency.
- Triggering factors.
- Relieving factors.
- Associated symptoms.
- User-provided contextual information.

The extracted information should be converted into a structured representation before being passed to the Assessment Engine.

---

### 4.3 Location Understanding

The AI may extract anatomical location information from natural-language input.

Example:

> "وجع في الجزء السفلي من بطني ناحية اليمين"

Possible structured representation:

    Region: Abdomen
    Area: Lower
    Side: Right

The AI should not be responsible for converting 3D coordinates into anatomical locations.

The 3D Body System is responsible for visual location selection.

The flow is:

    3D Body
       ↓
    Selected Coordinates / Region
       ↓
    Anatomical Mapping
       ↓
    Structured Location
       ↓
    Assessment Engine

---

### 4.4 Voice Input

The system shall support voice-based symptom input.

The expected flow is:

    User Voice
       ↓
    Speech-to-Text
       ↓
    AI Understanding
       ↓
    Structured Symptoms
       ↓
    Assessment Engine

Speech-to-text and natural-language understanding are considered separate capabilities.

The AI Requirements therefore cover the understanding and processing of the resulting text, while the specific speech-to-text technology will be evaluated separately.

---

### 4.5 Ambiguity Detection

The AI shall identify ambiguous user input.

Example:

> "عندي وجع في جنبي."

The system may not have enough information to determine the exact location.

Instead of guessing, the AI should support a clarification flow.

The flow is:

    Ambiguous Input
          ↓
    Detect Uncertainty
          ↓
    Request Clarification
          ↓
    Structured Information

---

### 4.6 Missing Information Detection

The AI shall be capable of identifying information that is unclear or unavailable in the user's description.

However, the AI should not independently decide which medical questions are required.

The Assessment Engine determines the information required for the assessment.

The flow is:

    Assessment Engine
           ↓
    Required Information
           ↓
          AI
           ↓
    Natural-language Question
           ↓
          User

---

### 4.7 Clarification

The AI shall support natural-language clarification.

The Assessment Engine determines what information is required.

The AI is responsible for presenting the question naturally and clearly.

Example:

Assessment Engine:

> Need exact side.

AI:

> "هل الألم موجود في الناحية اليمين، الشمال، ولا في منتصف البطن؟"

This separation prevents the AI from independently creating the medical assessment flow.

---

### 4.8 Result Explanation

The Assessment Engine produces the assessment output.

The AI may transform the output into an easier-to-understand explanation.

The flow is:

    Assessment Engine
           ↓
    Structured Result
           ↓
      AI Explanation
           ↓
    User-friendly Response

The AI must not modify the medical meaning of the assessment result.

It should not:

- Add unsupported conclusions.
- Remove important warnings.
- Change the urgency level.
- Replace the recommended next step.
- Present a definitive diagnosis.

---

## 5. AI Safety Boundaries

The AI must operate within strict boundaries.

### AI Can

- Understand natural language.
- Extract symptoms.
- Normalize user descriptions.
- Identify ambiguity.
- Support clarification.
- Process voice-derived text.
- Explain assessment results.
- Adapt explanations to the user's language and level of understanding.

### AI Cannot Independently

- Provide a definitive diagnosis.
- Override the Assessment Engine.
- Override Safety Rules.
- Ignore red flags.
- Reduce a detected urgency level.
- Invent medical information.
- Invent user information.
- Recommend unsafe actions.
- Claim certainty when the available information is insufficient.

---

## 6. Assessment Engine Authority

The Assessment Engine is the authoritative component responsible for performing the assessment.

The flow is:

    AI
     ↓
    Understanding
     ↓
    Structured Symptoms
     ↓
    Assessment Engine
     ↓
    Assessment Result

The AI must not replace the Assessment Engine.

The Assessment Engine is responsible for:

- Evaluating structured symptoms.
- Applying assessment rules.
- Determining possible explanations.
- Determining urgency.
- Determining appropriate next-step guidance.
- Applying safety-related rules.

---

## 7. Safety Layer

Safety-related decisions must not depend solely on generative AI behavior.

The system should maintain a dedicated safety mechanism capable of detecting and handling high-risk scenarios.

The flow is:

    Structured Symptoms
           ↓
    Safety Evaluation
           ↓
    Assessment Engine
           ↓
          Result

AI-generated content must not override safety decisions.

---

## 8. AI Uncertainty

The AI shall support uncertainty detection.

If the model is uncertain about the meaning of user input, it should not guess.

Instead:

    Low Confidence
          ↓
    Clarification
          ↓
    Additional Information
          ↓
    Higher Confidence

The system should preserve uncertainty instead of converting uncertain information into false certainty.

---

## 9. Hallucination Prevention

The AI must minimize unsupported information generation.

The system should:

- Prefer structured data provided by the user.
- Avoid inventing symptoms.
- Avoid inventing medical history.
- Avoid inventing medications.
- Avoid inventing allergies.
- Avoid inventing assessment results.
- Avoid generating unsupported medical claims.
- Use controlled context where appropriate.
- Validate structured AI output before passing it to the Assessment Engine.

---

## 10. Structured AI Output

Whenever the AI extracts information for system processing, the output should use a structured format.

Example:

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
          }
        }
      ]
    }

The exact schema will be defined during the Data Requirements and Architecture phases.

The system should validate AI-generated structured output before using it in the Assessment Engine.

---

## 11. AI Technology Direction

SymptoSense will use an **Open-Weight AI Model** as the initial AI technology direction.

The project will not train a foundation model from scratch.

The specific production model will not be permanently selected at the requirements stage.

Instead, candidate models will be evaluated against SymptoSense-specific requirements.

### Initial Candidate Models

1. Qwen
2. Llama
3. MedGemma

---

## 12. Candidate Model Comparison

### 12.1 Qwen

Qwen is a strong general-purpose open-weight model family with broad multilingual capabilities.

Qwen3 supports a wide range of languages and dialects, making it a strong candidate for multilingual interaction and Arabic-language experimentation.

#### Strengths

- Strong general-purpose language understanding.
- Broad multilingual support.
- Suitable for natural-language interaction.
- Multiple model sizes provide deployment flexibility.
- Good candidate for structured information extraction.
- Strong candidate for Arabic-language experimentation.
- Suitable for conversational clarification.

#### Weaknesses / Considerations

- Not specifically designed as a medical model.
- Medical performance must be evaluated for our specific use cases.
- Larger variants may require significant compute resources.
- Arabic and Egyptian Arabic performance must be tested.

#### Expected Role

Qwen is the primary candidate for the first prototype and evaluation.

---

### 12.2 Llama

Llama is a strong general-purpose open-weight model family.

Llama 4 introduces native multimodal capabilities, making the family potentially useful for future multimodal product expansion.

#### Strengths

- Strong general-purpose language capabilities.
- Large ecosystem.
- Strong developer community.
- Multilingual capabilities.
- Multimodal capabilities in newer variants.
- Potential future value for multimodal expansion.

#### Weaknesses / Considerations

- Not specifically designed for medical assessment.
- Medical performance must be evaluated.
- Larger models may require significant infrastructure.
- The specific model and license must be evaluated before production use.
- Arabic and Egyptian Arabic performance must be tested.

#### Expected Role

Llama will be included as a general-purpose baseline for comparison.

---

### 12.3 MedGemma

MedGemma is a medical-focused open model family designed for healthcare-related AI applications, including medical text and image understanding.

#### Strengths

- Specifically optimized for healthcare-related applications.
- Strong medical text comprehension.
- Medical image comprehension capabilities in applicable variants.
- Potentially valuable for medical terminology and context.
- Strong candidate for medical-specific experimentation.

#### Weaknesses / Considerations

- Medical specialization does not automatically make it suitable as an autonomous medical decision-maker.
- Specific use cases require validation.
- Larger variants may have higher infrastructure requirements.
- Medical models can still produce incorrect or unsafe outputs.
- Arabic and Egyptian Arabic performance must be tested.

#### Expected Role

MedGemma will be evaluated as the medical-specialized candidate.

---

## 13. High-Level Model Comparison

| Criterion | Qwen | Llama | MedGemma |
|---|---|---|---|
| General Language Understanding | High | High | High |
| Multilingual Support | Strong | Strong | Requires Evaluation |
| Arabic / Egyptian Arabic | Must be Tested | Must be Tested | Must be Tested |
| Medical Specialization | General | General | Strong |
| Structured Extraction | Strong Candidate | Strong Candidate | Strong Candidate |
| Clarification | Strong Candidate | Strong Candidate | Strong Candidate |
| Medical Text Understanding | Good Candidate | Good Candidate | Strong Candidate |
| Multimodal Potential | Depends on Variant | Strong | Strong |
| Healthcare Focus | General | General | High |
| Deployment Flexibility | High | High | Depends on Variant |
| MVP Suitability | High | High | High |
| Main Risk | Medical accuracy | Medical specialization | General conversational performance / validation |
| Current Evaluation Priority | **Highest** | High | High |

This is a preliminary product-level comparison and is not considered a final benchmark.

---

## 14. Initial Recommendation

Qwen is currently the preferred candidate for the first prototype because SymptoSense initially requires strong:

- Natural-language understanding.
- Multilingual interaction.
- Structured extraction.
- Clarification.
- General conversational capability.

However, Qwen is **not yet approved as the final production model**.

MedGemma remains a strong candidate because of its medical specialization, while Llama remains an important general-purpose baseline.

The final decision must be based on internal evaluation.

---

## 15. Model Evaluation Strategy

The final model will be selected through controlled evaluation rather than by general benchmark reputation.

The evaluation dataset should represent real SymptoSense use cases.

### 15.1 Language Understanding

- Arabic.
- Egyptian Arabic.
- English.
- Mixed Arabic/English medical terminology.

### 15.2 Symptom Extraction

- Symptom type.
- Location.
- Duration.
- Severity.
- Onset.
- Associated symptoms.

### 15.3 Location Understanding

- Anatomical terminology.
- Everyday descriptions.
- Egyptian colloquial descriptions.
- Ambiguous locations.

### 15.4 Clarification

- Detect missing information.
- Detect ambiguous information.
- Generate appropriate clarification.

### 15.5 Structured Output

- Correct schema.
- Valid JSON.
- Consistent field mapping.
- No unsupported fields.

### 15.6 Safety

- No hallucinated symptoms.
- No invented medical history.
- No unsupported diagnosis.
- Correct handling of uncertainty.
- Respect for safety boundaries.

### 15.7 Performance

- Response latency.
- Throughput.
- Memory usage.
- GPU requirements.
- Resource consumption.

---

## 16. Model Evaluation Metrics

Candidate models should be evaluated using measurable metrics.

| Metric | Purpose |
|---|---|
| Symptom Extraction Accuracy | Measures correct extraction |
| Location Accuracy | Measures anatomical location extraction |
| Structured Output Validity | Measures schema compliance |
| Clarification Accuracy | Measures whether clarification is appropriate |
| Hallucination Rate | Measures unsupported information |
| Safety Violation Rate | Measures unsafe behavior |
| Consistency | Measures response stability |
| Arabic Understanding Score | Measures Arabic performance |
| Egyptian Arabic Score | Measures local-language performance |
| Latency | Measures response speed |
| Resource Usage | Measures infrastructure requirements |

---

## 17. AI Fallback Strategy

The system should not assume that the AI service will always succeed.

Potential failures include:

- Model unavailable.
- Timeout.
- Invalid structured output.
- Low confidence.
- Unsupported input.
- Infrastructure failure.

The system should handle these situations gracefully.

Example:

    AI Failure
        ↓
    Retry / Fallback
        ↓
    If unsuccessful
        ↓
    Controlled Error Response

The system must never generate a fabricated assessment simply because the AI service failed.

---

## 18. AI Provider Abstraction

The application should not be tightly coupled to a single AI model.

The AI integration should conceptually follow:

    Application
         ↓
    AI Service Interface
         ↓
    AI Provider
      ├── Qwen
      ├── Llama
      └── MedGemma

This allows the project to evaluate or replace models without redesigning the entire product.

The exact implementation pattern will be defined during the Architecture phase.

---

## 19. Privacy Considerations

Because SymptoSense processes potentially sensitive health-related information, AI integration must consider:

- Where data is processed.
- Whether data is stored.
- Data retention.
- Access control.
- Encryption.
- Logging restrictions.
- Model infrastructure security.
- Third-party dependencies.

Self-hosting an open-weight model can provide greater operational control over data, but it does not automatically guarantee security or privacy.

The system remains responsible for securing its infrastructure and data.

---

## 20. AI Functional Requirements

### AI-001 — Natural Language Understanding

The system shall understand natural-language symptom descriptions.

### AI-002 — Symptom Extraction

The system shall extract structured symptom information from user input.

### AI-003 — Multilingual Input

The system shall support Arabic and English input.

### AI-004 — Egyptian Arabic Evaluation

The AI model shall be specifically evaluated using Egyptian Arabic test cases.

### AI-005 — Ambiguity Detection

The system shall detect ambiguous symptom descriptions.

### AI-006 — Missing Information Support

The system shall support identification of missing or unclear information required by the Assessment Engine.

### AI-007 — Clarification

The system shall generate natural-language clarification questions based on requirements provided by the Assessment Engine.

### AI-008 — Voice Input Processing

The system shall support processing of speech-to-text output.

### AI-009 — Structured Output

The system shall produce structured output when AI-extracted information is consumed by system components.

### AI-010 — Output Validation

The system shall validate AI-generated structured output before passing it to the Assessment Engine.

### AI-011 — Result Explanation

The system shall provide user-friendly explanations of Assessment Engine results.

### AI-012 — Assessment Authority

The AI shall not independently perform or replace the final medical assessment.

### AI-013 — Safety Authority

The AI shall not override Safety Rules or reduce an established urgency level.

### AI-014 — Uncertainty Handling

The AI shall preserve uncertainty and request clarification when required rather than guessing.

### AI-015 — Hallucination Prevention

The system shall minimize unsupported information generation and prevent fabricated assessment data from entering the assessment pipeline.

### AI-016 — Failure Handling

The system shall handle AI service failures without fabricating assessment results.

### AI-017 — Replaceable AI Model

The AI integration shall allow replacement of the underlying model without requiring a complete redesign of the application.

### AI-018 — Model Evaluation

Candidate models shall be evaluated using SymptoSense-specific test cases.

### AI-019 — Safety Evaluation

Candidate models shall be evaluated for hallucination and safety violations.

### AI-020 — Performance Evaluation

Candidate models shall be evaluated for latency, throughput, and infrastructure requirements.

### AI-021 — Open-Weight Strategy

The system shall initially use an Open-Weight AI Model.

### AI-022 — Final Model Selection

The final production model shall remain undecided until the model evaluation phase is completed.

---

## 21. Final AI Technology Decision

The current product decision is:

> **SymptoSense will use an Open-Weight AI Model as its AI technology direction.**

Initial candidates:

- Qwen
- Llama
- MedGemma

Current evaluation priority:

1. **Qwen** — Primary Prototype Candidate
2. **MedGemma** — Medical-specialized Candidate
3. **Llama** — General-purpose Baseline

This ranking is **not a final production decision**.

The final model will be selected based on controlled evaluation using SymptoSense-specific test cases.

The most important evaluation areas are:

- Egyptian Arabic understanding.
- Symptom extraction.
- Anatomical location extraction.
- Clarification quality.
- Structured output.
- Hallucination rate.
- Safety behavior.
- Latency.
- Infrastructure requirements.

---

## 22. Core Design Principle

The fundamental AI principle of SymptoSense is:

> **AI understands and communicates. The Assessment Engine evaluates. The Safety Layer protects.**

The AI must remain an assistive intelligence layer and must not become the autonomous medical decision-maker.