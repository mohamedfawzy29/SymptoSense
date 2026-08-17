# MVP Medical Coverage

**Document Version:** 1.0  
**Status:** Draft — Team Review Required  
**Last Updated:** 2026-08-15

---

## 1. Purpose

This document defines the medical coverage scope that SymptoSense intends to support during the MVP phase.

The goal is not to build a medical knowledge base covering every possible symptom or medical condition. Instead, the MVP shall focus on a limited, well-defined set of symptoms that can be supported with reliable medical information, appropriate assessment logic, and validated safety rules.

This document is intended to provide a structured basis for deciding the initial medical scope of the MVP.

---

## 2. MVP Coverage Principle

The MVP should support a limited number of common and high-value symptoms rather than attempting to cover all possible symptoms from the beginning.

The initial target is:

> **Top 10 Symptoms**

However, the final selection shall be determined by the SymptoSense team after reviewing and discussing the medical coverage, safety requirements, implementation feasibility, and product priorities.

The recommendations in this document are **preliminary and do not represent the final MVP decision**.

---

## 3. Coverage Required for Each Symptom

For every symptom approved for MVP coverage, the team should define the following:

    Symptom
       ↓
    Body Location
       ↓
    Relevant Context
       ↓
    Required Information
       ↓
    Assessment Questions
       ↓
    Red Flags
       ↓
    Possible Explanations
       ↓
    Safety Rules
       ↓
    Assessment Logic
       ↓
    Medical Sources

The level of detail required may vary depending on the symptom and its assessment complexity.

---

## 4. Medical Coverage Boundaries

The MVP shall not attempt to cover:

- Every possible medical symptom.
- Every medical disease or condition.
- Every body region with the same level of detail.
- Extremely rare conditions unless they are necessary for appropriate Safety or Red Flag coverage.
- Medical information that cannot be supported by reliable and traceable sources.

The MVP shall prioritize reliable and testable coverage over breadth.

---

## 5. Medical Source Requirement

Medical information included in the MVP should be supported by reliable and traceable medical sources.

Sources should support the specific information being used rather than simply being listed as general references.

The intended relationship is:

    Medical Rule
        ↓
    Medical Source
        ↓
    Specific Supported Information

Medical information that cannot be adequately supported should not be included in the MVP assessment logic.

---

## 6. MVP Symptom Selection Criteria

Candidate symptoms shall be evaluated using the following criteria.

### 6.1 Frequency

How commonly users are expected to experience or search for the symptom.

### 6.2 User Value

How valuable supporting the symptom would be to the target user.

This considers whether SymptoSense can provide meaningful assessment value rather than simply identifying the symptom.

### 6.3 Assessment Feasibility

How realistically the symptom can be assessed within the MVP using structured information, targeted questions, safety rules, and the Assessment Engine.

### 6.4 Safety Coverage

How effectively relevant safety concerns and Red Flags can be identified using validated medical rules.

### 6.5 Data Availability

How easily the team can obtain sufficient, reliable, and traceable medical information to support the symptom.

This includes information required for:

- Relevant questions
- Possible explanations
- Red Flags
- Safety rules
- Assessment logic

### 6.6 Implementation Complexity

How difficult it is to implement reliable MVP support for the symptom.

This may include:

- Number of required questions
- Number of relevant contexts
- Assessment logic complexity
- Number of body locations
- Breadth of medical knowledge
- Safety-rule complexity

A lower implementation complexity receives a higher score.

### 6.7 Body Coverage

How useful the symptom is for helping the MVP cover different body regions and assessment scenarios.

This criterion helps prevent the selected symptoms from being overly concentrated in one body area.

---

## 7. Candidate Symptoms

The following 12 symptoms have been selected as initial candidates for MVP evaluation:

1. **Headache**
2. **Fever**
3. **Cough**
4. **Abdominal Pain**
5. **Chest Pain**
6. **Back Pain**
7. **Sore Throat**
8. **Nausea / Vomiting**
9. **Diarrhea**
10. **Dizziness**
11. **Shortness of Breath**
12. **Skin Rash**

These candidates are intended to provide a broad starting point for discussion across different body regions and assessment scenarios.

The list is **not final** and does not represent a commitment to include all or any specific symptom in the MVP.

---

## 8. Scoring Method

Each candidate symptom shall be evaluated using the seven criteria defined above.

Each criterion receives a score from **1 to 5**.

### General Score Meaning

| Score | Meaning |
|---|---|
| 1 | Very Low / Unfavorable |
| 2 | Low |
| 3 | Moderate |
| 4 | High |
| 5 | Very High / Favorable |

### Implementation Complexity Scoring

Implementation Complexity uses the reverse interpretation:

| Score | Meaning |
|---|---|
| 1 | Very High Complexity |
| 2 | High Complexity |
| 3 | Moderate Complexity |
| 4 | Low Complexity |
| 5 | Very Low Complexity |

Therefore, a higher total score represents a more favorable candidate for MVP consideration.

The maximum possible score is:

**7 criteria × 5 points = 35 points**

---

## 9. Preliminary Scoring

The following scoring is a **proposed preliminary assessment** intended to support team discussion.

| Symptom | Frequency | User Value | Assessment Feasibility | Safety Coverage | Data Availability | Implementation Complexity | Body Coverage | Total / 35 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Headache | 5 | 5 | 4 | 5 | 5 | 4 | 4 | **32** |
| Fever | 5 | 5 | 5 | 5 | 5 | 5 | 3 | **33** |
| Cough | 5 | 5 | 5 | 4 | 5 | 5 | 3 | **32** |
| Abdominal Pain | 5 | 5 | 3 | 5 | 5 | 3 | 5 | **31** |
| Chest Pain | 4 | 5 | 3 | 5 | 5 | 3 | 4 | **29** |
| Back Pain | 5 | 5 | 4 | 5 | 5 | 4 | 4 | **32** |
| Sore Throat | 4 | 4 | 5 | 4 | 5 | 5 | 3 | **30** |
| Nausea / Vomiting | 4 | 4 | 4 | 4 | 5 | 4 | 4 | **29** |
| Diarrhea | 4 | 4 | 5 | 5 | 5 | 5 | 3 | **31** |
| Dizziness | 4 | 5 | 3 | 5 | 5 | 3 | 4 | **29** |
| Shortness of Breath | 4 | 5 | 3 | 5 | 5 | 3 | 4 | **29** |
| Skin Rash | 4 | 4 | 3 | 4 | 5 | 3 | 5 | **28** |

---

## 10. Preliminary Recommended Top 10

Based solely on the preliminary scoring above, the following symptoms are currently recommended for further MVP consideration:

1. **Fever — 33**
2. **Headache — 32**
3. **Cough — 32**
4. **Back Pain — 32**
5. **Abdominal Pain — 31**
6. **Diarrhea — 31**
7. **Sore Throat — 30**
8. **Chest Pain — 29**
9. **Nausea / Vomiting — 29**
10. **Dizziness — 29**

The following candidates are currently outside the preliminary Top 10:

- **Shortness of Breath — 29**
- **Skin Rash — 28**

---

## 11. Team Review and Final Selection

The preliminary ranking in this document shall **not** be considered the final MVP medical scope.

The final symptom selection shall be determined by the SymptoSense team after reviewing and discussing:

- Medical coverage feasibility
- Reliability and availability of medical sources
- Safety and Red Flag requirements
- Assessment Engine complexity
- MVP development capacity
- Product priorities
- Expected user value
- Overall body-region coverage
- Potential overlap between selected symptoms

The team may change the preliminary ranking, replace candidates, or select a symptom outside the preliminary Top 10 if there is sufficient justification.

In particular, symptoms with significant safety implications, such as **Chest Pain** and **Shortness of Breath**, should receive explicit team discussion rather than being included or excluded solely based on their numerical score.

---

## 12. Coverage Expansion

After the MVP, additional symptoms may be added through a controlled coverage expansion process.

The expansion should follow the same general evaluation principles used for the initial MVP:

    New Symptom Candidate
            ↓
    Medical Coverage Review
            ↓
    Safety Review
            ↓
    Assessment Feasibility
            ↓
    Source Availability
            ↓
    Implementation Evaluation
            ↓
    Team Decision
            ↓
    Expanded Medical Coverage

New symptoms should not be added to the Assessment Engine without defining the medical coverage and safety requirements necessary to support them.

---

## 13. Dependency on Assessment Engine

The Assessment Engine Requirements shall be designed based on the medical coverage approved for the MVP.

The Assessment Engine should not be considered fully specified until the supported symptom scope and the associated medical requirements have been defined.

The relationship is:

    MVP Medical Coverage
            ↓
    Supported Symptoms
            ↓
    Medical Information
            ↓
    Safety Rules
            ↓
    Assessment Engine Requirements

The Assessment Engine must operate within the boundaries of the approved medical coverage.

---

## 14. Document Status

**Current Status:** Draft — Team Review Required

The candidate symptoms, scoring, and preliminary Top 10 are recommendations for discussion only.

The final MVP medical coverage shall be recorded after team review and approval.