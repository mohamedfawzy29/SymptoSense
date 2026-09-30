# SymptoSense — Health Profile Module Specification

**Developer:** Farouk
**Module:** Health Profile
**Version:** 1
**Status:** Draft — Reconciled with Approved Requirements (Team Review Required)
**Architecture:** Modular Monolith + Clean Architecture
**Backend:** ASP.NET Core
**Supersedes:** v1

---

## 1. Module Overview

### 1.1 Purpose

The **Health Profile module** manages and provides the user's health-related information.

The module manages the user's height, weight, BMI, pregnancy status, and categorized medical conditions/allergies. It also provides the health information required by the **Assessment module**, including the user's age and sex.

The module is responsible for creating, displaying, updating, and managing the user's health profile and medical conditions.

---

## 2. Health Profile Responsibilities

The Health Profile module is responsible for:

- Creating a health profile for a user.
- Displaying the user's health profile.
- Updating the user's height and weight.
- Calculating and storing the user's BMI, recalculating it every time height or weight changes.
- Storing and updating the user's pregnancy status, when applicable.
- Adding, updating, and removing medical conditions and allergies, each tagged with its category.
- Providing the supported list of conditions and the supported list of allergies for the user to choose from.
- Providing the required health information to the Assessment module, reflecting the **current** state of the profile at the time of the request.
- Identifying the user's health profile using `UserId`.

---

## 3. Health Profile Data

| Data                | Description                                                             | Source                      |
| ------------------- | ----------------------------------------------------------------------- | --------------------------- |
| `UserId`            | Identifies the user who owns the profile                                | Authentication/Registration |
| `Sex`               | User's sex                                                              | Registration                |
| `DateOfBirth`       | User's date of birth                                                    | Registration                |
| `Height`            | User's height                                                           | Health Profile              |
| `Weight`            | User's weight                                                           | Health Profile              |
| `BMI`               | Body Mass Index calculated from height and weight                       | Health Profile              |
| `MedicalConditions` | Conditions and allergies belonging to the user, each tagged by category | Health Profile              |
| `PregnancyStatus`   | User's current pregnancy status; only collected when `Sex` is Female    | Health Profile              |

### 3.1 Pregnancy Status

`PregnancyStatus` is only shown on the form when the user's `Sex` is Female. For any other value of `Sex`, the field is not requested at all — there is no `NotApplicable` enum member; for non-Female users the value is simply `null`, not a stored enum value.

```csharp
public enum PregnancyStatus
{
    Yes,
    No,
    Unknown
}
```

`Unknown` specifically means the user was asked and doesn't know whether she's pregnant — it does not mean "hasn't been asked yet." Whenever the currently stored value is `Unknown`, the system should actively ask the question to try to resolve it to `Yes` or `No`, rather than leaving it unresolved by default. This applies every time the value is still `Unknown` going into a new assessment, since pregnancy status can change quickly and an unresolved answer carries real safety weight for the Assessment Engine.

### 3.2 Derived Age

Age is derived from the user's `DateOfBirth`. The user does not manually enter or update their age.

```text
DateOfBirth
     ↓
Calculate Age
     ↓
Age
```

Age is provided to the Assessment module as part of the assessment health data.

---

## 4. Health Profile Display

The main Health Profile screen displays:

- Name
- Sex
- Age

Name, sex, and date of birth originate from registration. Age is derived from date of birth.

The profile also provides a separate button that opens the **Medical Conditions** section.

```text
Health Profile
│
├── Name
├── Sex
├── Age
│
└── Medical Conditions
        ↓
      [Open]
```

---

## 5. Medical Conditions

The Medical Conditions section is separate from the main profile display. Each entry the user adds is categorized as either a **Medical Condition** or an **Allergy**, chosen via a selector in the UI at the time it's added.

```csharp
public enum MedicalConditionCategory
{
    Condition,
    Allergy
}
```

Rather than free text, the user picks from a supported list specific to the chosen category — a supported list of conditions (e.g., Diabetes, Asthma, Heart Conditions) and a separate supported list of allergies (e.g., Penicillin, Peanuts). The exact contents of these two lists are a medical/content decision for the team, consistent with how the platform's overall MVP Medical Coverage is a curated, bounded set rather than open-ended free text.

The current design still doesn't store severity information for either category — that remains a separate open item (Section 17).

---

## 6. Health Profile Use Cases

### 6.1 Create Health Profile

```text
Height + Weight
       ↓
   Calculate BMI
       ↓
   Save Health Profile
```

Sex and date of birth originate from registration, not from this use case.

### 6.2 Display Health Profile

The system retrieves the user's health profile using their `UserId` and displays Name, Sex, and Age. The user can separately open Medical Conditions.

### 6.3 Update Health Profile

```text
User changes Height/Weight
            ↓
       Recalculate BMI
            ↓
        Save new BMI
```

This recalculation is unconditional: **every** Height or Weight update triggers an immediate BMI recalculation and overwrite of the stored value. There is no state where Height/Weight has changed but the stored BMI is left stale.

The user can also update `PregnancyStatus` here (Female users only).

### 6.4 Add Medical Condition

```text
User
  |
  v
Choose Category: Medical Condition or Allergy
  |
  v
Select from the supported list for that category
  |
  v
Save (linked to UserId)
```

The user can add a medical condition, an allergy, or both, one at a time through this flow.

### 6.5 Update Medical Condition

The user can modify an existing entry, including changing its category or its selected value from the supported list.

### 6.6 Remove Medical Condition

The user can remove an existing medical condition or allergy.

---

## 7. BMI Business Rule

$$
BMI = \frac{Weight\ (kg)}{Height^2\ (m^2)}
$$

BMI is maintained by the Health Profile module, stored in the Health Profile database, and is **not** calculated by the Assessment module.

```text
Height/Weight
     ↓
BMI Calculation
     ↓
Updated BMI
     ↓
Save to Database
```

When Assessment requests the user's health information, the Health Profile module returns the currently stored BMI — always the live value reflecting whatever Height/Weight is on file right now — or an explicit "not available" value if it hasn't been calculated yet (Section 9.1). Preserving what BMI looked like at the moment a specific assessment was completed is the Assessment Module's job, not Health Profile's (Section 9.2) — the assessment record is saved to its own database at completion time, so nothing that happens to the profile afterward can reach back and change it.

---

## 8. Communication With the Assessment Module

The required information is:

- Age
- Sex
- BMI
- Medical Conditions
- Allergies
- Pregnancy Status (only when `Sex` is Female)

Assessment does not directly access the Health Profile database or its internal implementation.

```text
Assessment
     │
     │ Request user's health data
     │
     ▼
Health Profile Contract
     │
     ├── Age
     ├── Sex
     ├── BMI
     ├── Medical Conditions
     ├── Allergies
     └── Pregnancy Status
     │
     ▼
Assessment
```

Current Medications is not part of this contract yet (Section 17).

---

## 9. Assessment Contract

```csharp
public interface IHealthProfileService
{
    Task<AssessmentHealthDataDto> GetAssessmentHealthDataAsync(
        Guid userId);
}
```

```csharp
public class AssessmentHealthDataDto
{
    public int Age { get; set; }

    public Sex Sex { get; set; }

    public decimal? BMI { get; set; }

    public List<string> MedicalConditions { get; set; }

    public List<string> Allergies { get; set; }

    public PregnancyStatus? PregnancyStatus { get; set; }
}
```

`BMI` stays nullable (`decimal?`) to represent "not available" rather than a false default. `PregnancyStatus` is nullable at the contract level too: `null` means the field doesn't apply to this user (`Sex` isn't Female); a non-null value (`Yes` / `No` / `Unknown`) means it applies and this is the recorded answer.

### 9.1 Behavior When No Health Profile Exists Yet

A registered user can exist without ever having completed the height/weight step of their Health Profile. `GetAssessmentHealthDataAsync` defines behavior for that case:

| Field               | When profile is incomplete/missing                         |
| ------------------- | ---------------------------------------------------------- |
| `Age`               | Still returned — sourced from Registration (`DateOfBirth`) |
| `Sex`               | Still returned — sourced from Registration                 |
| `BMI`               | `null` (not available)                                     |
| `MedicalConditions` | Empty list                                                 |
| `Allergies`         | Empty list                                                 |
| `PregnancyStatus`   | `Unknown` if `Sex` is Female, otherwise `null`             |

This keeps the contract consistent with BR-030: a `null`/`Unknown` value should be treated by Assessment as "unknown," never as an assumed default.

### 9.2 BMI Recalculation and Assessment Result Immutability — Firm Rules

Two related but distinct rules:

**Rule 1 — Health Profile's BMI is always live.**

```text
Height or Weight changes
        ↓
BMI recalculated immediately
        ↓
New BMI overwrites the old value in the Health Profile database
```

There is no "stale BMI" state in Health Profile by design — the stored value is always current as of the last Height/Weight update.

**Rule 2 — A completed assessment is immutable, regardless of what happens to Health Profile afterward.**

```text
Assessment Completed
        ↓
Health data (Age, Sex, BMI, Conditions, Allergies, Pregnancy Status) AT THAT MOMENT
        ↓
Stored as part of the Assessment record (Assessment Module's database, not Health Profile's)
        ↓
User later changes Weight → BMI in Health Profile updates
        ↓
The already-stored Assessment record is NOT touched, re-read, or recalculated
```

Once an assessment is completed and persisted, nothing that happens afterward in Health Profile — including a BMI change — can alter, manipulate, or retroactively affect that stored result. This is consistent with BR-074 (Result Version Integrity) and BR-092 (Historical Result Integrity): a saved Assessment Result stays tied to the data that produced it.

Health Profile has no special code to enforce Rule 2 — it simply always answers truthfully with the current state. Immutability of the stored assessment is enforced on the Assessment Module's side: it must copy the health data into its own record at completion time rather than keeping a live reference back to Health Profile.

---

## 10. Responsibility of the Contract

Assessment only knows it can call `GetAssessmentHealthData(UserId)`. It does not need to know where the data is stored, how BMI is calculated, how conditions are stored, how age is derived, or which repository/EF implementation is used.

---

## 11. Read/Write Access Between Modules

The Assessment module has **read-only** access to the health information it requires. Assessment cannot create a profile, update height/weight, change BMI, update pregnancy status, or add/update/remove medical conditions — all of that stays inside Health Profile.

```text
User
 │
 ▼
Health Profile
 │
 ├── Create
 ├── Update
 ├── Add Condition
 ├── Update Condition
 └── Remove Condition

Assessment
 │
 └── Read required health data only
```

---

## 12. Communication With AI

The AI module has no direct dependency on Health Profile. It processes user answers into JSON, which is sent to Assessment alongside the Health Profile data Assessment fetches separately.

```text
                 User
                  │
                  ▼
             User Answers
                  │
                  ▼
                 AI
                  │
                  ▼
             JSON Answers
                  │
                  ▼
             Assessment
                  ▲
                  │
       Health Profile Data
                  │
                  │
          Health Profile
```

---

## 13. Clean Architecture Structure

```text
HealthProfile/
│
├── Domain/
│   └── Entities/
│       ├── HealthProfile.cs
│       ├── MedicalCondition.cs
│       ├── MedicalConditionCategory.cs (enum)
│       └── PregnancyStatus.cs (enum)
│
├── Application/
│   ├── Interfaces/
│   │   └── IHealthProfileService.cs
│   │
│   ├── DTOs/
│   │   └── AssessmentHealthDataDto.cs
│   │
│   ├── Commands/
│   │   ├── CreateHealthProfile/
│   │   ├── UpdateHealthProfile/
│   │   ├── AddMedicalCondition/
│   │   ├── UpdateMedicalCondition/
│   │   └── RemoveMedicalCondition/
│   │
│   ├── Queries/
│   │   ├── GetHealthProfile/
│   │   └── GetSupportedConditionsAndAllergies/
│   │
│   └── Services/
│       └── HealthProfileService.cs
│
├── Infrastructure/
│   ├── Persistence/
│   │   └── HealthProfileDbContext.cs
│   │
│   └── Repositories/
│       └── HealthProfileRepository.cs
│
└── Presentation/
    └── Controllers/
        └── HealthProfileController.cs
```

---

## 14. Clean Architecture Data Flow

```text
Client
  ↓
Controller
  ↓
Application
  ↓
Domain
  ↓
Repository Interface
  ↓
Repository Implementation
  ↓
Database
```

```csharp
public interface IHealthProfileRepository
{
    Task<HealthProfile?> GetByUserIdAsync(Guid userId);

    Task AddAsync(HealthProfile profile);

    Task UpdateAsync(HealthProfile profile);
}
```

`GetByUserIdAsync` returning `null` is the expected signal for "no profile yet" — the Application layer maps that into the defaults described in Section 9.1 rather than throwing.

---

## 15. Proposed API Endpoints

| HTTP Method | Endpoint                                   | Purpose                                                           |
| ----------- | ------------------------------------------ | ----------------------------------------------------------------- |
| `GET`       | `/api/health-profile`                      | Get the current user's profile                                    |
| `POST`      | `/api/health-profile`                      | Create a health profile                                           |
| `PUT`       | `/api/health-profile`                      | Update height, weight, and pregnancy status                       |
| `GET`       | `/api/health-profile/conditions`           | Get the user's medical conditions and allergies                   |
| `GET`       | `/api/health-profile/conditions/supported` | Get the supported list of conditions and allergies to choose from |
| `POST`      | `/api/health-profile/conditions`           | Add a medical condition or allergy                                |
| `PUT`       | `/api/health-profile/conditions/{id}`      | Update a medical condition or allergy                             |
| `DELETE`    | `/api/health-profile/conditions/{id}`      | Remove a medical condition or allergy                             |

Subject to final team/API agreement.

---

## 16. Module Boundary

```text
┌──────────────────────────┐
│    Health Profile        │
│                          │
│  Health Profile Data     │
│  Medical Conditions      │
│  Allergies               │
│  BMI                     │
│  Pregnancy Status        │
│                          │
│  Database                │
└─────────────┬────────────┘
              │
              │ Contract
              ▼
       ┌──────────────┐
       │  Assessment  │
       └──────────────┘
```

---

## 17. Open Items Requiring Team Confirmation

1. **Current Medications** — not part of this module's data model or the Assessment contract yet.
2. **Severity for medical conditions and allergies** — carried over, still undecided.

---

## 18. Summary

The Health Profile module manages: User ID, Height, Weight, BMI, Pregnancy Status (Female users only), and a categorized list of Medical Conditions and Allergies. Sex and date of birth originate from registration; Age is derived from date of birth.

Two rules are firm, not open questions:

- BMI is recalculated and overwritten every single time Height or Weight changes — Health Profile never holds a stale BMI.
- Once an assessment is completed and stored, it is immutable. Nothing that happens to Health Profile afterward — including a later BMI change — can alter, manipulate, or retroactively affect a completed assessment record. Assessment is responsible for keeping its own copy of the health data at the moment it was used.

Pregnancy Status is only ever asked of Female users, uses a three-value answer (`Yes` / `No` / `Unknown`), and is actively re-asked whenever it's still `Unknown`, since an unresolved answer carries real safety weight.

Medical Conditions and Allergies are explicitly categorized at the point of entry, each drawn from its own supported list rather than free text, and are returned to Assessment as two separate lists.

Two items remain open for team discussion: Current Medications and severity for conditions/allergies .
