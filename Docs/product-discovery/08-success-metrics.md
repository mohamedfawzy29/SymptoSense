# Success Metrics

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-08

---

# Purpose

This document defines how the success of SymptoSense will be measured.

The goal is not to measure success only by the number of users or assessments performed.

SymptoSense is intended to help users move from:

> "I don't know what's wrong with me."

to:

> "I understand my symptoms better and I know what my appropriate next step is."

Therefore, the metrics should measure whether the product is successfully reducing uncertainty, collecting useful symptom information, providing relevant assessment, and guiding users toward an appropriate next step.

---

# Measurement Principles

The following principles will guide product measurement:

1. Measure user value, not only product usage.
2. Measure the complete assessment journey.
3. Measure the quality of symptom understanding and input.
4. Measure whether users understand the final guidance.
5. Measure safety separately from general product performance.
6. Avoid treating AI accuracy as a single metric.
7. Do not define arbitrary targets before establishing a baseline.
8. Metrics should evolve as the product moves from prototype to MVP and later stages.

---

# North Star Metric

## Successful Assessment Completion Rate

### Definition

The percentage of users who start a symptom assessment and successfully reach the final assessment stage with clear next-step guidance.

### Formula

```text
Users who complete an assessment and receive actionable guidance
───────────────────────────────────────────────────────────────    × 100
               Users who start an assessment
```

### Why It Matters

The primary purpose of SymptoSense is not simply to make users interact with the 3D model, AI, or voice interface.

The product should successfully guide the user through the complete journey:

```text
Symptom / Concern
       ↓
Symptom Discovery
       ↓
Symptom Understanding
       ↓
Assessment
       ↓
Possible Explanations
       ↓
Urgency Understanding
       ↓
Clear Next Step
```

A successful assessment represents completion of the core product value journey.

---

# Primary Product Metrics

## 1. Assessment Completion Rate

### Definition

Percentage of users who complete the assessment after starting it.

### Formula

```text
Completed Assessments
──────────────────────  × 100
 Started Assessments
```

### Why It Matters

A low completion rate may indicate:

- Complicated UX.
- Excessive questioning.
- Confusing instructions.
- Lack of user trust.
- Long assessment duration.
- Technical problems.

---

## 2. Next-Step Clarity

### Definition

Percentage of users who report that they understand what they should do after receiving the assessment.

### Example Question

> "After completing the assessment, do you understand what your next step should be?"

Possible responses:

- Yes.
- Partially.
- No.

### Why It Matters

This metric directly measures one of the primary goals of SymptoSense.

The product should reduce uncertainty rather than simply provide medical information.

---

## 3. Assessment Helpfulness

### Definition

The percentage of users who consider the assessment useful in understanding their situation.

### Example Question

> "Did this assessment help you understand your symptoms better?"

### Why It Matters

A technically successful assessment is not necessarily a useful assessment.

The user should leave with a clearer understanding of the situation.

---

## 4. Assessment Relevance

### Definition

The percentage of users who consider the questions and information presented relevant to their symptoms.

### Example Question

> "How relevant were the questions asked during your assessment?"

### Why It Matters

The assessment should avoid unnecessary questions and should adapt to the user's specific situation.

---

# User Journey Metrics

The assessment journey should be measurable as individual stages.

```text
Landing
   ↓
Assessment Start
   ↓
Initial Input
   ↓
Symptom Identification
   ↓
Location Identification
   ↓
Follow-up Questions
   ↓
Assessment
   ↓
Results
   ↓
Next-Step Guidance
```

For each stage, we should measure:

- Entry rate.
- Completion rate.
- Drop-off rate.
- Time spent.
- Errors or retries.
- User corrections.

---

# Drop-off Rate

## Definition

The percentage of users who leave the assessment before completing it at each stage.

### Example

```text
Assessment Start
      ↓
Initial Input       → 5% Drop-off
      ↓
Location            → 12% Drop-off
      ↓
Follow-up Questions → 18% Drop-off
      ↓
Results
```

### Why It Matters

Overall completion rate tells us that users are leaving.

Stage-level drop-off tells us **where and potentially why** they are leaving.

---

# Symptom Input Metrics

Because helping users express their symptoms is a major part of the product concept, symptom input requires dedicated metrics.

---

## 1. Symptom Input Success Rate

### Definition

The percentage of assessment sessions in which the system successfully extracts a usable symptom representation from the user's initial input.

### Possible Input Sources

- Voice.
- Text.
- 3D body interaction.

### Example

```text
User Input
    ↓
Symptom Detection
    ↓
Location
    ↓
Duration
    ↓
Other Relevant Attributes
```

The exact minimum required representation will be defined during the requirements and AI design phases.

---

## 2. User Correction Rate

### Definition

The percentage of extracted symptom information that users need to correct.

### Example

```text
User:
"My stomach hurts on the right."

System:
"Do you mean the highlighted area?"

User:
"No, slightly lower."
```

This represents a correction.

### Why It Matters

A high correction rate may indicate problems with:

- Natural language understanding.
- Voice recognition.
- Symptom extraction.
- Location interpretation.
- User interface clarity.

---

# 3D Localization Metrics

The 3D body is a major part of the intended UX and therefore requires specific measurement.

---

## 1. Location Identification Success Rate

### Definition

Percentage of users who successfully identify the location of their symptom using the body interface.

### Example Flow

```text
Whole Body
    ↓
Body Region
    ↓
Sub-Region
    ↓
Specific Location
```

The definition of "successful" will be refined during UX testing.

---

## 2. Location Correction Rate

### Definition

Percentage of sessions in which the user has to correct the location selected or interpreted by the system.

### Why It Matters

This helps determine whether the 3D interaction is actually improving symptom localization.

---

## 3. Time to Location Identification

### Definition

Time required for a user to successfully identify the relevant body location.

### Why It Matters

The 3D interface should improve communication without creating unnecessary interaction complexity.

---

# Voice and Text Input Metrics

## Voice Input Success Rate

Measures how often users can successfully communicate a usable symptom description through voice input.

Potential failure reasons include:

- Speech recognition errors.
- Unclear pronunciation.
- Background noise.
- Unsupported terminology.
- Incorrect symptom extraction.

---

## Text Input Success Rate

Measures how often free-text descriptions are successfully converted into usable structured symptom information.

---

# Assessment Quality Metrics

The assessment should not be evaluated using a single "AI accuracy" number.

Instead, assessment quality should be measured across multiple dimensions.

---

## 1. Symptom Extraction Accuracy

Measures whether the system correctly identifies symptoms from user input.

Examples:

- Pain.
- Fever.
- Nausea.
- Dizziness.

---

## 2. Location Extraction Accuracy

Measures whether the system correctly identifies the body location described by the user.

---

## 3. Relevant Question Rate

Measures whether the follow-up questions are relevant to the information already provided.

A good assessment should progressively narrow uncertainty without asking unnecessary questions.

---

## 4. Context Utilization

Measures whether relevant user health context is correctly incorporated into the assessment when available.

Potential context includes:

- Age.
- Chronic conditions.
- Current medications.
- Allergies.
- Relevant medical history.

---

# Safety Metrics

Safety metrics are treated separately from general product metrics.

A highly engaging assessment is not considered successful if it fails to appropriately identify situations requiring urgent medical attention.

---

## 1. Safety Escalation Performance

Measures how well the system handles situations containing potential red flags.

The system should appropriately communicate when the user may need:

- Emergency care.
- Urgent medical evaluation.
- Routine medical consultation.
- Monitoring and follow-up.

The exact evaluation methodology and thresholds will be defined during the Safety and Clinical Requirements phase.

---

## 2. Unsafe Recommendation Rate

Measures cases in which the system provides guidance that could potentially cause harmful delay, inappropriate reassurance, or inappropriate self-treatment.

This metric requires dedicated safety evaluation and expert review.

---

## 3. Safety Communication Clarity

Measures whether users understand when they should seek urgent or professional medical care.

---

# Guest User Metrics

The first assessment should be available to the user as a guest.

Therefore, the guest experience should have its own measurements.

---

## 1. Guest Assessment Completion Rate

Measures how successfully users complete the first assessment without requiring account creation.

### Why It Matters

The initial guest experience is designed to reduce friction and allow the user to experience the core product value before registration.

---

## 2. Guest-to-Registration Conversion

### Definition

Percentage of guest users who decide to create an account after experiencing the product.

### Why It Matters

Registration should be driven by perceived value rather than forced before the user understands the product.

---

# User Retention Metrics

Retention becomes more important after the first version of the product is released.

Potential metrics include:

- Returning users.
- Repeat assessments.
- Assessment history usage.
- Chronic-condition monitoring usage.
- Follow-up assessment usage.

These metrics will become more relevant once the product supports authenticated users and longitudinal health context.

---

# Product Performance Metrics

Product performance should also be monitored because technical problems can directly affect the assessment experience.

Potential metrics include:

- Assessment response time.
- AI response latency.
- Voice processing latency.
- 3D interaction performance.
- Error rate.
- API failure rate.
- Assessment session failure rate.

These metrics are technical health metrics and should be monitored separately from user-value metrics.

---

# MVP Metrics

The initial MVP should focus on a small set of high-value metrics.

## Primary

1. Successful Assessment Completion Rate.

## Secondary

2. Assessment Completion Rate.
3. Next-Step Clarity.
4. Assessment Helpfulness.
5. Symptom Input Success Rate.
6. Location Identification Success Rate.
7. Drop-off Rate.

## Safety

8. Safety Escalation Performance.
9. Unsafe Recommendation Rate.

---

# Future Metrics

As the product matures, additional metrics can be introduced.

Potential future metrics include:

- Guest-to-registration conversion.
- User retention.
- Repeat assessment rate.
- Chronic condition assessment usage.
- Assessment history usage.
- Personalized health context usage.
- Voice adoption.
- 3D interaction adoption.
- Longitudinal symptom tracking.
- User trust and satisfaction.

---

# Metrics vs. Targets

Metrics and targets should be treated separately.

A metric defines:

> What we measure.

A target defines:

> What level of performance we want to achieve.

At this stage, arbitrary numerical targets should not be assigned.

Targets should be established after:

1. UX prototype testing.
2. Initial usability testing.
3. MVP implementation.
4. Baseline data collection.
5. Safety evaluation.

This allows targets to be based on evidence rather than assumptions.

---

# Measurement Levels

Success should be evaluated at multiple levels.

```text
Product Success
      │
      ├── User Value
      │     ├── Next-Step Clarity
      │     └── Assessment Helpfulness
      │
      ├── UX
      │     ├── Completion
      │     ├── Drop-off
      │     └── Time
      │
      ├── Input
      │     ├── Symptom Extraction
      │     ├── Location Identification
      │     └── User Corrections
      │
      ├── Assessment
      │     ├── Relevance
      │     ├── Context Utilization
      │     └── Question Quality
      │
      ├── Safety
      │     ├── Escalation Performance
      │     └── Unsafe Recommendation Rate
      │
      └── Technical
            ├── Latency
            ├── Errors
            └── Availability
```

---

# Success Definition

SymptoSense should be considered successful when it consistently helps users move from uncertainty about their symptoms toward a clearer understanding and an appropriate next step.

The product should therefore optimize for:

```text
Less Uncertainty
        +
Better Symptom Understanding
        +
Relevant Assessment
        +
Clear Next-Step Guidance
        +
Strong Safety
```

rather than optimizing solely for:

```text
More Users
More Sessions
More AI Interactions
```

---

# Future Evaluation

The success metrics defined in this document will be reviewed and refined as the product progresses through:

```text
Discovery
    ↓
UX Prototype
    ↓
Usability Testing
    ↓
MVP
    ↓
Initial Baseline
    ↓
Target Definition
    ↓
Continuous Measurement
```

Metrics may be added, removed, or adjusted based on evidence collected during these stages.