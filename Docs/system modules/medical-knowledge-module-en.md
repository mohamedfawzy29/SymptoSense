# Medical Knowledge Module — Architectural Documentation

**Project:** SymptoSense  
**Module Code:** MK  
**Version:** 1.0  
**Status:** Architectural Documentation — Pre-Implementation  
**Last Updated:** 2026-09-28  

---

## 1. Module Overview & Philosophy

### 1.1 Module Definition

The **Medical Knowledge Module** serves as the authoritative, central **Medical Single Source of Truth (SSOT)** for SymptoSense.

It is responsible for storing, organizing, and serving standardized clinical taxonomies (symptoms, body regions, and health conditions), clinical assessment rules, and versioned medical knowledge bundles. 

It acts strictly as an informational and rule repository:
* It **supplies** approved rules and clinical models to the **Assessment Engine**.
* It **does not execute** evaluations, compute clinical inferences, manage active assessment sessions, or communicate with the end-user.

### 1.2 Immutability and Auditability Principle

```text
Any medical knowledge used in an assessment MUST be versioned, immutable, 
and strictly auditable.
```

1. **Deterministic Reproducibility:** Historical assessments retain the exact `KnowledgeVersion` used at the moment of evaluation. Changes to clinical guidelines never silently mutate or invalidate past assessment records.
2. **Medical Approval Gate:** No rule or symptom mapping can become `Active` in production without formal approval through a versioned release lifecycle.

### 1.3 Module Positioning in the Architecture

| Component | Responsibility Relative to Medical Knowledge |
|---|---|
| **Medical Knowledge Module** | **Owns and serves** clinical taxonomies, assessment rules, symptom attributes, condition profiles, and knowledge versions. |
| **Assessment Engine Module** | **Consumes** medical rules and criteria from Medical Knowledge via strict read-only contracts to evaluate user sessions. |
| **Safety Rules & Emergency Module** | **Owns and evaluates** red flags, critical emergency rules, and acute escalation criteria independently. |
| **AI Module** | Maps extracted natural language entities to standard symptom codes defined in the Medical Knowledge taxonomy. |
| **Admin & Medical Portal** | Authoring interface for clinical specialists to draft, review, test, and release knowledge versions. |

---

## 2. Responsibilities & Explicit Boundaries

### 2.1 In-Scope Responsibilities ✅

* **Standardized Symptom Catalog:** Maintain symptoms taxonomy, localized descriptions (Arabic / English), severity scales, duration units, qualifying attributes, and anatomical anchors.
* **Condition Catalog:** Maintain supported health conditions, non-diagnostic guidance summaries, and clinical specialty classifications (e.g., Cardiology, Gastroenterology).
* **Assessment Rules Repository:** Store declarative clinical evaluation rules (e.g., qualifying question criteria, symptom-condition associations, required findings).
* **Knowledge Versioning:** Manage semantic versioning (`Major.Minor.Patch`), release stages (`Draft`, `InReview`, `Approved`, `Active`, `Deprecated`), and change auditing.
* **High-Performance Querying:** Provide low-latency, cached in-memory query contracts tailored for high-frequency consumption by the Assessment Engine.

### 2.2 Out-of-Scope Responsibilities ❌

* **No Safety & Red-Flag Rules:** Red flags, emergency interruption triggers, and immediate escalation guardrails are explicitly owned by the **Safety Rules & Emergency Module**.
* **No Session State Management:** The module is completely stateless with respect to patient visits or running consultations.
* **No Diagnostic Execution:** The module stores rules; the **Assessment Engine** runs and reasons over them.
* **No NLP / Language Translation:** Free-form text parsing, colloquial Egyptian Arabic handling, and speech processing belong exclusively to the **AI Module**.

---

## 3. Internal Architecture (.NET 10 & Clean Architecture)

The module follows Clean Architecture principles within the Modular Monolith structure (`src/Modules/MedicalKnowledge/`):

```text
┌────────────────────────────────────────────────────────────────────────┐
│                   Medical Knowledge Module                             │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                     API / Controllers Layer                      │  │
│  │         (ASP.NET Core Minimal APIs / Internal RPC Endpoints)      │  │
│  └──────────────────────────────────┬───────────────────────────────┘  │
│                                     │                                  │
│  ┌──────────────────────────────────▼───────────────────────────────┐  │
│  │                    Application Layer                             │  │
│  │   Queries: GetActiveRulesQuery, GetSymptomTaxonomyQuery          │  │
│  │   Commands: CreateKnowledgeVersionCommand, PublishVersionCommand │  │
│  │   Caching: MemoryCache / Distributed Cache Invalidation Handler  │  │
│  └──────────────────────────────────┬───────────────────────────────┘  │
│                                     │                                  │
│  ┌──────────────────────────────────▼───────────────────────────────┐  │
│  │                       Domain Layer                               │  │
│  │   Entities: Symptom, Condition, AssessmentRule, KnowledgeVersion │  │
│  │   Value Objects: SymptomCode, ClinicalSpecialty, RuleCondition   │  │
│  │   Domain Events: KnowledgeVersionPublishedDomainEvent            │  │
│  └──────────────────────────────────┬───────────────────────────────┘  │
│                                     │                                  │
│  ┌──────────────────────────────────▼───────────────────────────────┐  │
│  │                    Infrastructure Layer                          │  │
│  │   PostgreSQL + EF Core 10 (JSONB Rules Engine Persistence)       │  │
│  │   Read-optimized Repositories & Compiled EF Core Queries         │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────┬──────────────────────────────────┘
                                      │ Exposes Contract Library
                                      ▼
             ┌─────────────────────────────────────────────────┐
             │ SymptoSense.Modules.MedicalKnowledge.Contracts │
             │  - IMedicalKnowledgeReader                      │
             │  - Read Models & DTOs (SymptomDto, RuleDto)     │
             └─────────────────────────────────────────────────┘
```

---

## 4. Domain & Data Models (ERD)

### 4.1 Entities & Aggregates

#### `KnowledgeVersion` (Aggregate Root)
Encapsulates an immutable bundle of approved clinical knowledge.
* `Id`: Guid
* `VersionTag`: String (e.g., `"1.0.0"`)
* `Status`: VersionStatus Enum (`Draft`, `InReview`, `Approved`, `Active`, `Deprecated`)
* `EffectiveFrom`: DateTime?
* `ApprovedBy`: String? (Medical Specialist Identifier)
* `ChangeNotes`: String
* `CreatedAt`: DateTime

#### `Symptom` (Entity)
Represents a standardized clinical complaint.
* `Id`: Guid
* `KnowledgeVersionId`: Guid (FK)
* `Code`: String (e.g., `"SYM_HEADACHE"`, `"SYM_CHEST_PAIN"`)
* `DisplayNameEn`: String
* `DisplayNameAr`: String
* `DescriptionAr`: String
* `PrimaryRegionCode`: String (links to 3D Body anatomical coordinates)
* `AllowedLocations`: List<AnatomicalLocation>
* `SupportedAttributes`: JSONB (duration units, severity scales, character options)
* `IsMvpScope`: Boolean (flags the 6 MVP symptoms)

#### `Condition` (Entity)
Represents a clinical possibility used for patient guidance.
* `Id`: Guid
* `KnowledgeVersionId`: Guid (FK)
* `Code`: String (e.g., `"CND_MIGRAINE"`, `"CND_TENSION_HEADACHE"`)
* `NameEn`: String
* `NameAr`: String
* `SummaryAr`: String (patient-safe explanation)
* `RecommendedSpecialty`: Specialty Enum (e.g., `Neurology`, `Cardiology`, `GeneralPractice`)
* `GuidanceType`: GuidanceType Enum (`RoutineConsultation`, `SpecialistEvaluation`)

#### `AssessmentRule` (Entity)
Declarative rule used by the Assessment Engine to drive question generation and condition weighting.
* `Id`: Guid
* `KnowledgeVersionId`: Guid (FK)
* `RuleCode`: String
* `TargetSymptomCode`: String (FK to Symptom)
* `RuleType`: RuleType Enum (`RequiredInformationRule`, `ConditionWeightRule`, `ClusterRule`)
* `Expression`: JSONB (Microsoft RulesEngine compatible predicate)
* `Payload`: JSONB (Defines question requirement or condition score delta)

### 4.2 Entity Relationship Diagram (PostgreSQL)

```text
┌────────────────────────────────────────┐
│           knowledge_versions           │
├────────────────────────────────────────┤
│ id                 UUID PRIMARY KEY    │
│ version_tag        VARCHAR(20) UNIQUE  │
│ status             VARCHAR(20)         │
│ effective_from     TIMESTAMPTZ NULL    │
│ approved_by        VARCHAR(100) NULL   │
│ change_notes       TEXT                │
│ created_at         TIMESTAMPTZ         │
└───────────────────┬────────────────────┘
                    │ 1
                    │
         ┌──────────┴──────────┬────────────────────────┐
         │ 1..*                │ 1..*                   │ 1..*
         ▼                     ▼                        ▼
┌──────────────────┐  ┌──────────────────┐  ┌───────────────────────┐
│     symptoms     │  │    conditions    │  │   assessment_rules    │
├──────────────────┤  ├──────────────────┤  ├───────────────────────┤
│ id          UUID │  │ id          UUID │  │ id               UUID │
│ version_id  UUID │  │ version_id  UUID │  │ version_id       UUID │
│ code     VARCHAR │  │ code     VARCHAR │  │ rule_code     VARCHAR │
│ name_en  VARCHAR │  │ name_en  VARCHAR │  │ target_symptomVARCHAR │
│ name_ar  VARCHAR │  │ name_ar  VARCHAR │  │ rule_type     VARCHAR │
│ primary_region   │  │ specialtyVARCHAR │  │ expression      JSONB │
│ allowed_loc JSONB│  │ summary_ar  TEXT │  │ payload         JSONB │
│ attributes JSONB │  │ guidance_type    │  │ created_atTIMESTAMPTZ │
│ is_mvp   BOOLEAN │  └──────────────────┘  └───────────────────────┘
└──────────────────┘
```

---

## 5. Contract & Integration with Assessment Engine

### 5.1 Contract Interface (`IMedicalKnowledgeReader`)

The `AssessmentEngine` references only `SymptoSense.Modules.MedicalKnowledge.Contracts` and consumes the following contract via .NET Dependency Injection:

```csharp
namespace SymptoSense.Modules.MedicalKnowledge.Contracts;

public interface IMedicalKnowledgeReader
{
    /// <summary>
    /// Retrieves the current active knowledge version bundle metadata.
    /// </summary>
    Task<KnowledgeVersionDto> GetActiveVersionAsync(CancellationToken ct = default);

    /// <summary>
    /// Retrieves full clinical details and allowed attributes for given symptom codes.
    /// </summary>
    Task<IReadOnlyList<SymptomDefinitionDto>> GetSymptomsAsync(
        IEnumerable<string> symptomCodes, 
        string? versionTag = null, 
        CancellationToken ct = default);

    /// <summary>
    /// Fetches all assessment rules triggered by the specified symptom codes.
    /// </summary>
    Task<IReadOnlyList<AssessmentRuleDto>> GetAssessmentRulesAsync(
        IEnumerable<string> symptomCodes, 
        string? versionTag = null, 
        CancellationToken ct = default);

    /// <summary>
    /// Retrieves condition profiles and mapped specialties.
    /// </summary>
    Task<IReadOnlyList<ConditionProfileDto>> GetConditionsAsync(
        IEnumerable<string> conditionCodes, 
        string? versionTag = null, 
        CancellationToken ct = default);
}
```

### 5.2 High-Performance Caching Strategy

Because medical knowledge changes very infrequently but is read during every single step of an assessment session:
1. **L1 In-Memory Cache:** All active symptoms and rules are held in memory (`IMemoryCache`) using compiled dictionary lookups `O(1)`.
2. **Cache Invalidation:** When an administrator publishes a new `KnowledgeVersion`, a MediatR notification `KnowledgeVersionPublishedNotification` immediately flushes and reloads the active cache.

---

## 6. Knowledge Lifecycle & Authoring Flow

```text
┌─────────────────┐
│      Draft      │ ◄── Clinical team creates/updates symptoms or rules
└────────┬────────┘
         │ Submit for Review
         ▼
┌─────────────────┐
│    InReview     │ ◄── Automated schema checks & peer clinical review
└────────┬────────┘
         │ Medical Sign-off
         ▼
┌─────────────────┐
│    Approved     │ ◄── Staged for release; immutable snapshot taken
└────────┬────────┘
         │ Publish Trigger
         ▼
┌─────────────────┐
│     Active      │ ◄── Assessment Engine immediately serves this version
└────────┬────────┘
         │ Superseded by next release
         ▼
┌─────────────────┐
│   Deprecated    │ ◄── Retained read-only for historical assessment audit
└─────────────────┘
```

---

## 7. MVP Medical Scope Alignment (The 6 Symptoms)

The Medical Knowledge module's schema directly prepares for and supports the 6 prioritized MVP symptoms defined in `17-MVP-medical-coverage.md`:

| # | Symptom Code | English Name | Arabic Name | Anatomical Region |
|---|---|---|---|---|
| 1 | `SYM_HEADACHE` | Headache | صداع | Head / Cranium |
| 2 | `SYM_FEVER` | Fever | حمى / ارتفاع درجة الحرارة | Systemic / General |
| 3 | `SYM_COUGH` | Cough | كحة / سعال | Chest / Respiratory |
| 4 | `SYM_ABDOMINAL_PAIN` | Abdominal Pain | ألم بالبطن | Abdomen (Quadrant mapped) |
| 5 | `SYM_CHEST_PAIN` | Chest Pain | ألم بالصدر | Thorax / Anterior Chest |
| 6 | `SYM_BACK_PAIN` | Back Pain | ألم بالظهر | Spine / Upper & Lower Back |

*Note: The specific clinical rule predicates and differential condition weights for these 6 symptoms will be seeded via standardized JSON data migrations once finalized by the clinical review team.*

---

## 8. Endpoints & Use Cases

### 8.1 Engine Internal Endpoints (High Performance / Read-Only)

* `GET /api/v1/medical-knowledge/version/active`  
  *Returns active version metadata and checksum.*
* `POST /api/v1/medical-knowledge/symptoms/batch`  
  *Fetches definitions, anatomical regions, and valid questions for selected symptoms.*
* `POST /api/v1/medical-knowledge/rules/query`  
  *Returns evaluation rules matching active symptom clusters.*

### 8.2 Medical Administration Endpoints (Restricted / Review Flow)

* `POST /api/v1/admin/medical-knowledge/versions`  
  *Initializes a new draft knowledge version.*
* `POST /api/v1/admin/medical-knowledge/versions/{id}/publish`  
  *Promotes an approved version to active status and invalidates cache.*
* `POST /api/v1/admin/medical-knowledge/symptoms`  
  *Adds or updates a symptom definition in a draft version.*
* `POST /api/v1/admin/medical-knowledge/rules`  
  *Adds a Microsoft RulesEngine JSON logic rule.*

---

## 9. Business Rules Traceability Matrix

| Requirement / Rule | System Enforcement |
|---|---|
| **BR-038 (Knowledge Versioning)** | Every assessment records `knowledge_version` linked to an immutable version record in `knowledge_versions`. |
| **BR-029 (Standardized Medical Data)** | All symptoms, locations, and conditions use standardized, strictly typed codes (`SYM_*`, `CND_*`). |
| **BR-036 (Assessment Independence)** | Medical Knowledge strictly answers *what the rules are*; it never computes session scores. |
| **Section 29-31 (`18-assessment-engine-requirements.md`)** | Clear decoupling between the rule storage layer (JSONB in PostgreSQL) and rule evaluation layer in the Engine. |
