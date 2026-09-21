# 3D Module Specification

**Module:** 3D Body Interaction
**Version:** 1
**Status:** Final — Reconciled with Approved Requirements (Developer-Finalized)
**Supersedes:** v1

---

## 1. Module Overview

The **3D Module** provides an interactive 3D human-body interface that lets the user visually identify the location of their symptom.

The module does **not** perform medical assessment, diagnose symptoms, calculate assessment results, or store patient information.

Its primary responsibility is:

> **Allow the user to select an anatomical location through an interactive 3D model and provide that selection to the Assessment Module in a structured JSON format that matches the approved anatomical payload contract.**

The 3D Module is one of the possible input methods for an assessment. Symptom information can arrive through:

- Text
- Voice
- 3D body selection

Regardless of the input method, the information goes to the **Assessment Module**, which conducts the assessment and produces the final result.

---

## 2. Main Responsibilities

The 3D Module is responsible for:

- Providing an interactive 360° human-body model.
- Supporting separate adult male and adult female models.
- Allowing users to rotate and zoom the model.
- Enforcing the 4-level progressive selection: **Whole Body → Major Region → Sub-Region → Specific Location** (3D-NAV-002).
- Displaying a magnified 3D view of the selected region at each level.
- Providing an Arabic breadcrumb bar for navigating back to parent regions (3D-NAV-003).
- Providing a "Reset View" control that returns to the default full-body view (3D-NAV-004).
- Maintaining the predefined anatomical taxonomy (major region → sub-region → specific location) and each entry's Arabic label and anatomical code.
- Generating a structured anatomical-selection JSON payload matching the approved contract.
- Validating the anatomical selection structure before it leaves the module.
- Sending the anatomical selection(s) to the Assessment Module.

The module is **not** responsible for:

- Diagnosing diseases.
- Determining the severity of symptoms.
- Asking assessment questions.
- Determining whether enough information has been collected.
- Generating the assessment result.
- Storing assessment results.
- Storing health-profile information.
- Storing medical knowledge.
- Interpreting the clinical meaning of the selected location.
- Communicating with the AI Module.
- Communicating with the Medical Knowledge Module.
- Collecting Age or Gender (these come from the Initial Health Context step, not from this module).

---

## 3. Anatomical Hierarchy

This section defines the 4-level structure required by 3D-NAV-002. The tree below is a **representative starting taxonomy**, not an exhaustive clinical dictionary — see Open Item 2.

```text
Whole Body
│
├── Head & Neck (Major Region)
│     ├── Head (Sub-Region)
│     │     ├── Forehead
│     │     ├── Left Temple
│     │     ├── Right Temple
│     │     └── Back of Head
│     ├── Neck (Sub-Region)
│     │     ├── Front of Neck
│     │     └── Back of Neck
│     └── Throat (Sub-Region)
│           └── Throat (Center)
│
├── Torso (Major Region)
│     ├── Chest (Sub-Region — Anterior)
│     │     ├── Upper Left Chest
│     │     ├── Upper Right Chest
│     │     ├── Center Chest
│     │     ├── Lower Left Chest
│     │     └── Lower Right Chest
│     ├── Abdomen (Sub-Region — Anterior)
│     │     ├── Upper Abdomen (Epigastric)
│     │     ├── Right Upper Abdomen
│     │     ├── Left Upper Abdomen
│     │     ├── Right Lower Abdomen
│     │     ├── Left Lower Abdomen
│     │     └── Central / Around Navel
│     ├── Upper Back (Sub-Region — Posterior)
│     │     ├── Upper Back – Left
│     │     └── Upper Back – Right
│     └── Lower Back (Sub-Region — Posterior)
│           ├── Lower Back – Left
│           └── Lower Back – Right
│
├── Upper Limbs (Major Region)
│     ├── Arm (Sub-Region)
│     │     ├── Upper Arm
│     │     ├── Elbow
│     │     └── Forearm
│     └── Hand (Sub-Region)
│           ├── Wrist
│           └── Hand / Fingers
│
└── Lower Limbs (Major Region)
      ├── Leg (Sub-Region)
      │     ├── Thigh
      │     ├── Knee
      │     └── Lower Leg / Shin
      └── Foot (Sub-Region)
            ├── Ankle
            └── Foot / Toes
```

This coverage was chosen to line up with the current MVP candidate symptoms in the MVP Medical Coverage doc (Headache, Chest Pain, Abdominal Pain, Back Pain, and general body regions for the others). It should be extended as medical coverage expands.

### 3.1 `body_side` Derivation

`body_side` is not a separate control — it's derived from the sub-region selected:

| Sub-Region                       | `body_side`    |
| -------------------------------- | -------------- |
| Chest                            | Anterior       |
| Abdomen                          | Anterior       |
| Front of Neck                    | Anterior       |
| Upper Back                       | Posterior      |
| Lower Back                       | Posterior      |
| Back of Neck                     | Posterior      |
| Back of Head                     | Posterior      |
| Forehead / Temples               | Anterior       |
| Arm, Hand, Leg, Foot sub-regions | Not Applicable |

Limb regions default to `"Not Applicable"` for `body_side` since front/back isn't clinically meaningful for most limb symptoms at this granularity — flagged in Open Item 4 in case the team wants this changed.

---

## 4. Guest User Flow

```text
Guest
  |
  v
Initial Health Context (Age, Gender) — collected earlier, per Journey 1
  |
  v
New Assessment → Choose Input Method → 3D
  |
  v
Load 3D Model (Male/Female) using the Gender value
already collected in the Initial Health Context step
  |
  v
Select Major Region
  |
  v
Select Sub-Region
  |
  v
Select Specific Location
  |
  v
Generate Anatomical Selection
  |
  v
Assessment Module
```

The 3D Module no longer asks the guest to choose a sex/model directly. The application layer passes the already-collected Gender value into the 3D Module at initialization — the same value used elsewhere in the assessment. This keeps the 3D Module decoupled from Health Context/Health Profile while avoiding a duplicate question.

Gender/Sex is strictly binary (Male/Female) across the platform, so model selection is always a straightforward lookup — there's no undefined fallback case to design for here.

---

## 5. Logged-In User Flow

```text
Logged-In User
      |
      v
New Assessment → Choose Input Method → 3D
      |
      v
Load 3D Model (Male/Female) using the Sex value
from the Health Profile, provided by the application layer
      |
      v
Select Major Region
      |
      v
Select Sub-Region
      |
      v
Select Specific Location
      |
      v
Generate Anatomical Selection
      |
      v
Assessment Module
```

As in v1.0, Health Profile is **not** a direct dependency of the 3D Module — the application layer resolves the sex value and passes it in at initialization for both guest and logged-in users, keeping the data flow uniform.

---

## 6. 3D Interaction Flow

```text
Full Body
    |
    v
User selects a Major Region (e.g., Torso)
    |
    v
Region is magnified; breadcrumb shows: الجسم بالكامل ← الجذع
    |
    v
User selects a Sub-Region (e.g., Chest)
    |
    v
Breadcrumb shows: الجسم بالكامل ← الجذع ← الصدر
    |
    v
Predefined Specific Locations are displayed (e.g., Upper Left Chest)
    |
    v
User selects a Specific Location
```

At every level, the breadcrumb (3D-NAV-003) lets the user step back to a parent region, and the Reset View control (3D-NAV-004) returns to the full-body default. Camera transitions must complete within 1.5 seconds at ≥30 FPS (3D-NAV-002, PERF-002) — see the 3D Interaction Requirements doc for the full rendering/gesture spec; this document doesn't duplicate it.

If WebGL fails to render, the interface falls back to the direct Conversational Chat Mode per ERR-003 / 3D-PERF-003 — the 3D Module has no special handling here beyond not blocking that fallback.

---

## 7. Selection Constraint

For MVP, the interface UI allows selecting **one active location per assessment**, and the 3D flow is used at most once per assessment. If the initial 3D selection isn't enough information to complete the assessment, that gap is closed through **text-only follow-up questions** (Section 10) — the 3D flow is not invoked a second time within the same assessment.

The underlying contract (Section 8) still wraps the selection in an array rather than a bare object, purely as future-proofing in case multi-location UI is added later — see Open Item 1.

---

## 8. Anatomical Selection JSON

### Contract

```json
{
  "anatomical_selections": [
    {
      "body_side": "Anterior",
      "major_region": "Torso",
      "sub_region": "Chest",
      "specific_location": "Upper Left Chest",
      "arabic_label": "أعلى اليسار من الصدر",
      "anatomical_code": "CHEST_UPPER_LEFT"
    }
  ]
}
```

### Fields

| Field               | Type   | Description                                                                         |
| ------------------- | ------ | ----------------------------------------------------------------------------------- |
| `body_side`         | string | `"Anterior"`, `"Posterior"`, or `"Not Applicable"` — derived, see Section 3.1       |
| `major_region`      | string | Level-2 selection (e.g., `"Torso"`)                                                 |
| `sub_region`        | string | Level-3 selection (e.g., `"Chest"`)                                                 |
| `specific_location` | string | Level-4 selection (e.g., `"Upper Left Chest"`)                                      |
| `arabic_label`      | string | Arabic display label for the specific location (3D-LOC-001)                         |
| `anatomical_code`   | string | Stable machine key for AI/Assessment mapping, e.g. `CHEST_UPPER_LEFT` (3D-DATA-001) |

For MVP, this array will always contain exactly one entry, since the 3D flow runs at most once per assessment (Section 7). It's kept as an array rather than a bare object purely so a future multi-location UI wouldn't require a breaking contract change.

The 3D Module never generates free-text medical interpretations (e.g., "pain in the left side of the chest") — only the structured object above. The Assessment Module is responsible for interpreting it.

---

## 9. Relationship With Assessment

```text
+-------------+
|  3D Module  |
+-------------+
       |
       | anatomical_selections[]
       v
+----------------+
|   Assessment   |
+----------------+
```

Assessment is the only module that receives the 3D output. The 3D Module still does not communicate with Health Profile, AI, Medical Knowledge, Safety, Authentication, or Admin.

---

## 10. Assessment Interaction

The 3D Module only supplies location data — it does not decide whether that's enough to complete the assessment. If it isn't enough, Assessment does not call back into the 3D flow — it asks a follow-up question via text instead.

```text
3D
 |
 | anatomical_selections: [{ ... }]
 v
Assessment
 |
 v
Check collected information
 |
 +---- Enough information? --- YES ---> Generate Result
 |
 NO
 |
 v
Ask a follow-up question via text
 |
 v
Check information again
 |
 +---- Enough? ---- YES ---> Generate Result
 |
 NO
 |
 v
Continue asking follow-up questions via text
```

The 3D flow is a one-time input step at the start of localization, not a tool Assessment can invoke mid-conversation. Once control has passed to the question-and-answer part of the assessment, everything from that point on — including anything needed to further pin down location — is handled through text.

---

## 11. Stateless Design

The 3D Module has no memory. It doesn't have a database, and it doesn't remember:

```text
User History
Assessment History
Previous Selections
Patient Data
Assessment Results
```

It does one job: take the user's selection, turn it into the `anatomical_selections` object, and send it to Assessment. As soon as that's sent, the 3D Module is done — nothing is kept behind.

Assessment is the module that keeps the full picture over time.

---

## 12. Database

Still no database required. The anatomical taxonomy (major region → sub-region → specific location, plus each entry's `arabic_label`, `anatomical_code`, and `body_side` mapping) is represented as static configuration/module data, e.g.:

```text
Major Region: Torso
  Sub-Region: Chest (body_side: Anterior)
    Upper Left Chest   → CHEST_UPPER_LEFT   → أعلى اليسار من الصدر
    Upper Right Chest  → CHEST_UPPER_RIGHT  → أعلى اليمين من الصدر
    ...
```

Whether this configuration lives in the frontend bundle, a small backend lookup service, or a shared config store is an implementation decision — see Open Item 3.

---

## 13. Backend Responsibility

The backend stays small. Its job is to provide and/or validate the anatomical taxonomy and selection structure — not to do 3D rendering.

```text
Frontend
+--------------------------------+
| 3D Human Model                 |
| Rotation / Zoom                |
| Breadcrumb Navigation          |
| Reset View                     |
| 4-Level Region Selection       |
+---------------+----------------+
                |
                | anatomical_selections[]
                v
+--------------------------------+
|          3D Backend            |
| Validate major_region/         |
| sub_region/specific_location   |
| combination                    |
| Attach arabic_label,           |
| anatomical_code, body_side     |
+---------------+----------------+
                |
                v
          Assessment Module
```

---

## 14. Frontend Responsibilities

### 14.1 Model Rendering

- Adult male model
- Adult female model
  Both model files (`.glb`/`.gltf`, Draco-compressed, ≤3.5 MB total per 3D-PERF-001) are **bundled with the project** rather than fetched from a CDN — they ship as part of the app package itself. This resolves the asset storage/delivery question that was previously open.

### 14.2 3D Interaction

- 360° rotation
- Zoom
- 4-level progressive region selection
- Regional magnification at each level
- Arabic breadcrumb navigation (3D-NAV-003)
- Reset View control (3D-NAV-004)

### 14.3 Model Selection (Revised)

No "choose sex" step inside this module. The application passes the sex/gender value in at initialization for both guest and logged-in users:

```text
Application
  |
  v
Sex/Gender value (Male or Female)
(guest: from Initial Health Context step
 logged-in: from Health Profile)
  |
  v
3D Module loads corresponding model
```

### 14.4 Data Generation

After the user selects a specific location, the frontend (or the small 3D backend) constructs:

```json
{
  "anatomical_selections": [
    {
      "body_side": "Anterior",
      "major_region": "Torso",
      "sub_region": "Chest",
      "specific_location": "Upper Left Chest",
      "arabic_label": "أعلى اليسار من الصدر",
      "anatomical_code": "CHEST_UPPER_LEFT"
    }
  ]
}
```

### 14.5 Fallback

If WebGL fails to render within the required time budget, hide the 3D viewport and expand Conversational Chat Mode (ERR-003, 3D-PERF-003) without interrupting the session.

---

## 15. Backend API

```http
GET /api/3d/body-parts
```

Returns the full taxonomy:

```json
[
  {
    "majorRegion": "Torso",
    "subRegions": [
      {
        "subRegion": "Chest",
        "bodySide": "Anterior",
        "specificLocations": [
          {
            "location": "Upper Left Chest",
            "code": "CHEST_UPPER_LEFT",
            "arabicLabel": "أعلى اليسار من الصدر"
          },
          {
            "location": "Upper Right Chest",
            "code": "CHEST_UPPER_RIGHT",
            "arabicLabel": "أعلى اليمين من الصدر"
          }
        ]
      }
    ]
  }
]
```

```http
POST /api/3d/selection
```

Request:

```json
{
  "majorRegion": "Torso",
  "subRegion": "Chest",
  "specificLocation": "Upper Left Chest"
}
```

The backend validates that the combination exists in the taxonomy, then returns/forwards the full enriched object (with `body_side`, `arabic_label`, `anatomical_code`) to Assessment as shown in Section 8.

Final API shape is still an implementation detail, as in v1.0.

---

## 16. Clean Architecture

```text
3D Module
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
│   └── External/technical implementations
│
└── Presentation
    └── API Controllers
```

---

## 17. Domain Layer

```text
AnatomicalTaxonomyEntry
├── MajorRegion
├── SubRegion
├── SpecificLocation
├── BodySide
├── ArabicLabel
└── AnatomicalCode
```

Still no dependency on ASP.NET, Entity Framework, HTTP, controllers, databases, or frontend technology.

---

## 18. Application Layer

```csharp
public class AnatomicalSelectionDto
{
    public string MajorRegion { get; set; }
    public string SubRegion { get; set; }
    public string SpecificLocation { get; set; }
    public string BodySide { get; set; }
    public string ArabicLabel { get; set; }
    public string AnatomicalCode { get; set; }
}

public class AnatomicalSelectionRequestDto
{
    public List<AnatomicalSelectionDto> AnatomicalSelections { get; set; }
}

public interface IAnatomicalSelectionService
{
    Task<bool> ValidateSelectionAsync(
        string majorRegion, string subRegion, string specificLocation);

    Task<AnatomicalSelectionDto> EnrichSelectionAsync(
        string majorRegion, string subRegion, string specificLocation);
}
```

`EnrichSelectionAsync` is what attaches `body_side`, `arabic_label`, and `anatomical_code` once the raw selection is validated.

---

## 19. Infrastructure Layer

Unchanged in spirit from v1.0 — if the taxonomy is eventually externalized, this layer implements retrieval; for now a configuration-based implementation is sufficient since there's no database.

---

## 20. Presentation Layer

```text
3DController
```

```http
POST /api/3d/selection
```

Controller delegates to the Application layer; no validation/business logic lives in the controller itself.

---

## 21. Open Items Requiring Team Confirmation

1. **Multi-location contract shape.** The payload is wrapped in an array purely as future-proofing (Section 7–8), even though only one entry is ever populated for MVP. Confirm this is acceptable, or say the word and it can be flattened back to a single object.
2. **Full anatomical taxonomy.** Section 3's tree is a representative starting point aligned to current MVP candidate symptoms, not an exhaustive, clinically validated dictionary — that's a content task, not something to invent wholesale here.
3. **3D asset storage/delivery.** Still undecided (bundled vs. CDN vs. other), carried over from v1.0.

---

## 22. Summary

```text
Display 3D Body
      ↓
Select Major Region
      ↓
Select Sub-Region
      ↓
Select Specific Location
      ↓
Enrich with body_side / arabic_label / anatomical_code
      ↓
Send anatomical_selections[] to Assessment
```

The 3D flow runs once per assessment. If more information is still needed after that, it's gathered through text-only follow-up questions — the 3D flow is never invoked a second time. The 3D Module still does no assessment or medical reasoning — Assessment remains responsible for everything downstream of receiving the location data.
