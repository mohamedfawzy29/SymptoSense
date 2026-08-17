# Stakeholder Analysis

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-08

---

# Purpose

This document identifies the stakeholders involved in the SymptoSense product, their relationship with the system, their goals, needs, and level of involvement.

The purpose of this analysis is to ensure that the product requirements are based on the needs of all relevant parties and that no important stakeholder perspective is overlooked during the requirements engineering phase.

---

# Stakeholder Definition

A stakeholder is any individual, group, or organization that:

- Uses the system.
- Is affected by the system.
- Has an interest in the success of the product.
- Influences how the product operates.
- Is responsible for maintaining or governing part of the product.

Not every stakeholder is necessarily a direct system user.

---

# Stakeholder Categories

Stakeholders are divided into three main categories:

## Primary Stakeholders

Users who directly interact with the core product experience.

- Guest User
- Registered User

## Secondary Stakeholders

Stakeholders involved in operating, maintaining, or supporting the product.

- Admin
- Product / Engineering Team
- Support / Operations

## Future Stakeholders

Stakeholders that are expected to be introduced in future versions of the product.

- Medical Reviewer

---

# 1. Guest User

## Description

A Guest User is a first-time or unauthenticated user who wants to understand unexplained symptoms without creating an account initially.

The Guest User represents one of the primary target users of SymptoSense.

## Main Goals

The Guest User wants to:

- Understand what their symptoms may indicate.
- Describe symptoms without needing medical terminology.
- Identify the location of the problem.
- Understand possible explanations.
- Understand the potential urgency of the situation.
- Know what the appropriate next step may be.
- Get initial value from the product without creating an account.

## Main Pain Points

The Guest User may:

- Not know the medical name of their symptom.
- Not know the exact anatomical location of the problem.
- Have difficulty explaining what they are experiencing.
- Be unsure which medical specialty they should consult.
- Find conflicting information when searching online.
- Be uncertain about whether the symptom is serious.
- Want to understand the situation before deciding to seek medical care.
- Prefer not to create an account before experiencing the product's value.

## Product Needs

The Guest User should have access to:

- Initial guest assessment.
- Natural-language symptom input.
- Voice input.
- Text input.
- 3D body interaction.
- Guided symptom localization.
- Dynamic follow-up questions.
- Assessment results.
- Possible explanations.
- Urgency guidance.
- Clear next-step recommendations.

## Access Constraint

The Guest User can perform the initial assessment without creating an account.

After the initial guest assessment, continued access to the product's extended functionality requires registration/login.

---

# 2. Registered User

## Description

A Registered User is an authenticated user who has completed the initial guest experience and decided to continue using SymptoSense.

## Main Goals

In addition to the goals of the Guest User, the Registered User wants to:

- Access previous assessments.
- Track recurring symptoms.
- Maintain relevant health context.
- Store chronic conditions.
- Reuse relevant information during future assessments.
- Monitor changes across multiple assessments.

## Product Needs

The Registered User should have access to:

- Authentication.
- Personal profile.
- New assessments.
- Assessment history.
- Previous assessment details.
- Relevant health context.
- Chronic condition information.
- Continued use of symptom assessment features.

## Value Proposition

Registration should provide additional value rather than simply acting as a barrier to the initial assessment.

The user should have a clear reason to continue using the authenticated experience.

---

# 3. Admin

## Description

The Admin is responsible for managing and operating administrative aspects of the SymptoSense platform.

The Admin is not a primary end-user of the symptom assessment experience.

## Main Goals

The Admin should be able to:

- Manage users.
- Manage system content where applicable.
- Monitor system activity.
- Manage configurable assessment elements where permitted.
- Review system issues.
- Support operational management.

## Potential Responsibilities

```text
Admin
 ├── User Management
 ├── Content Management
 ├── Assessment Configuration
 ├── System Monitoring
 └── Operational Management
```

The exact administrative capabilities will be defined during the Functional Requirements phase.

---

# 4. Product / Engineering Team

## Description

The Product / Engineering Team is responsible for building, maintaining, and improving SymptoSense.

This stakeholder group includes roles such as:

- Product Manager / Product Owner.
- Backend Engineers.
- Frontend Engineers.
- AI / ML Engineers.
- QA Engineers.
- DevOps / Infrastructure Engineers.

## Main Goals

The team needs to:

- Implement the product vision.
- Translate requirements into a working system.
- Maintain system quality.
- Monitor product performance.
- Maintain security and reliability.
- Analyze product metrics.
- Fix defects.
- Introduce future capabilities safely.

## Product Needs

The team requires:

- Clear requirements.
- Defined business rules.
- Acceptance criteria.
- Measurable success metrics.
- Clear system boundaries.
- Traceable requirements.
- Monitoring and logging capabilities.

---

# 5. Support / Operations

## Description

Support / Operations is responsible for handling operational and user-related issues after the product is deployed.

This stakeholder may not be required as a dedicated role in the initial MVP but should be considered for future operational needs.

## Main Goals

- Help users resolve account-related problems.
- Handle reported technical issues.
- Investigate failed assessment sessions.
- Escalate critical issues.
- Monitor recurring operational problems.

## Product Needs

Support / Operations may eventually require:

- User issue information.
- Assessment session status.
- Error information.
- Operational logs.
- Appropriate support tools.

---

# 6. Medical Reviewer — Future Role

## Description

The Medical Reviewer is a future stakeholder responsible for reviewing and validating medical content, assessment rules, and safety-related logic.

This role is **not part of the initial MVP**.

However, the system should avoid making future medical review impossible or excessively difficult to introduce.

## Main Goals

A Medical Reviewer may eventually:

- Review medical content.
- Review assessment rules.
- Review safety rules.
- Review red-flag definitions.
- Review medical explanations.
- Identify potentially inaccurate or unsafe content.
- Participate in medical quality governance.

## Future Product Needs

A future Medical Reviewer may require:

- Medical content management.
- Assessment rule review.
- Safety rule review.
- Versioning.
- Review workflows.
- Approval mechanisms.
- Audit history.

These capabilities are considered **Future Scope** and will not be implemented as part of the initial MVP unless the scope is explicitly changed.

---

# Stakeholder Prioritization

| Stakeholder | Category | Priority | MVP |
|---|---|---:|---:|
| Guest User | Primary | Critical | Yes |
| Registered User | Primary | Critical | Yes |
| Admin | Secondary | High | Yes |
| Product / Engineering Team | Secondary | High | Yes |
| Support / Operations | Secondary | Medium | Limited / Future |
| Medical Reviewer | Future | High | No |

---

# Stakeholder Needs Summary

| Stakeholder | Main Need |
|---|---|
| Guest User | Understand symptoms and receive clear next-step guidance |
| Registered User | Continue using the product and access personal assessment history/context |
| Admin | Manage and operate the platform |
| Product / Engineering Team | Build, maintain, measure, and improve the product |
| Support / Operations | Resolve user and operational issues |
| Medical Reviewer | Review and govern medical content and safety logic in future versions |

---

# User vs. System Actors

Stakeholders and system actors should not be treated as the same concept.

## Human / Organizational Stakeholders

```text
Guest User
Registered User
Admin
Product / Engineering Team
Support / Operations
Medical Reviewer (Future)
```

## System Components / Actors

The following are system capabilities or components rather than human stakeholders:

```text
AI Processing
Assessment Engine
Safety Engine
Authentication System
Notification / External Services
```

These components will be analyzed in more detail during the Functional Requirements and Architecture phases.

---

# Stakeholder Map

```text
                         SymptoSense
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ↓                   ↓                   ↓
        Users             Operations          Governance
          │                   │                   │
     ┌────┴────┐         ┌────┴────┐              │
     ↓         ↓         ↓         ↓              ↓
  Guest    Registered   Admin    Support      Medical
  User       User                / Ops        Reviewer
                                              (Future)
```

The Product / Engineering Team operates across the product and is responsible for building and maintaining the system.

---

# Key Decisions

1. Guest Users can perform the initial assessment without registration.
2. Registration is required for continued access to extended product functionality after the initial guest assessment.
3. Guest User and Registered User are the primary stakeholders.
4. Admin is an MVP role responsible for administrative operations.
5. Product / Engineering Team is considered a core internal stakeholder.
6. Support / Operations is considered a secondary operational stakeholder.
7. Medical Reviewer is a **Future Role** and is not part of the initial MVP.
8. Future Medical Reviewer capabilities should remain possible to introduce without fundamentally redesigning the product.
9. AI and assessment components are system components, not human stakeholders.

---

# Relationship to Next Phase

The stakeholder analysis provides the foundation for defining:

```text
Stakeholders
      ↓
User Roles
      ↓
Permissions
      ↓
User Journeys
      ↓
Functional Requirements
```

The next document will define the system roles and their permissions in detail.