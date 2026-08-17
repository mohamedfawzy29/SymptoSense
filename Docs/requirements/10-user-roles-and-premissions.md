# User Roles & Permissions

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-08

---

# Purpose

This document defines the user roles within SymptoSense and specifies the permissions and access boundaries associated with each role.

The purpose is to establish clear authorization rules before defining the detailed functional requirements and system architecture.

---

# Roles

SymptoSense will initially support the following roles:

1. Guest User
2. Registered User
3. Admin

The following role is planned for future versions:

4. Medical Reviewer

---

# Role Overview

| Role | Authentication | MVP | Main Purpose |
|---|---|---|---|
| Guest User | Not required | Yes | Perform the initial assessment |
| Registered User | Required | Yes | Continue using the platform and access personal features |
| Admin | Required | Yes | Manage and operate the platform |
| Medical Reviewer | Required | Future | Review medical content and safety rules |

---

# 1. Guest User

## Description

A Guest User is an unauthenticated user who has not created or logged into an account.

The Guest User can access the initial SymptoSense assessment without providing personal account information.

---

## Guest Permissions

The Guest User can:

### Assessment

- Start the initial assessment.
- Provide symptoms using supported input methods.
- Use text input.
- Use voice input when available.
- Use the 3D body interface.
- Select a general body region.
- Refine the selected body region.
- Select a specific location.
- Answer assessment questions.
- Modify answers before completing the assessment.
- Complete the initial assessment.
- View the resulting assessment information.
- View possible explanations.
- View urgency guidance.
- View recommended next steps.

### Account

The Guest User can:

- Choose to register after completing the initial assessment.
- Log in if an account already exists.

---

## Guest Restrictions

The Guest User cannot:

- Access persistent assessment history.
- Access another user's data.
- Manage a personal health profile.
- Maintain persistent chronic-condition information.
- Access authenticated-only features.
- Access administrative functionality.

---

# 2. Registered User

## Description

A Registered User is an authenticated user with an account in the SymptoSense system.

Registration provides access to functionality beyond the initial guest assessment.

---

## Registered User Permissions

### Authentication

The Registered User can:

- Log in.
- Log out.
- Manage their account.
- Update supported profile information.
- Manage account credentials through supported authentication flows.

### Assessment

The Registered User can:

- Start a new assessment.
- Provide symptoms.
- Use text input.
- Use voice input when available.
- Use the 3D body interface.
- Answer dynamic questions.
- Review and modify assessment information.
- Complete assessments.
- View assessment results.
- Receive next-step guidance.

### Assessment History

The Registered User can:

- Save completed assessments.
- View their previous assessments.
- Open assessment details.
- Review previous assessment results.
- Track recurring symptoms where supported.

### Health Context

The Registered User can:

- Add relevant health information.
- Update supported health context.
- Add chronic conditions.
- Update chronic conditions.
- Add other supported contextual information.

The exact health-context fields will be defined during the Data Requirements phase.

---

## Registered User Restrictions

The Registered User cannot:

- Access another user's assessments.
- Modify another user's health information.
- Access administrative functions.
- Modify system-wide assessment rules.
- Modify medical safety rules.
- Manage system users.
- Access internal administrative data.

---

# 3. Admin

## Description

The Admin is an authenticated privileged role responsible for managing and operating the platform.

Admin permissions are separate from normal user permissions.

---

## Admin Permissions

### User Management

The Admin may:

- View users.
- Search users.
- View supported account information.
- Manage account status where permitted.
- Handle account-related administrative actions.

### Content Management

The Admin may eventually:

- Manage system-managed content.
- Manage configurable information.
- Update supported non-clinical content.

Exact content-management permissions will be defined in the Functional Requirements phase.

### System Management

The Admin may:

- Monitor system activity.
- Review operational information.
- Access appropriate administrative dashboards.
- Review system errors and operational issues.

### Assessment Configuration

The Admin may manage configurable assessment elements where explicitly permitted.

However, Admin access does not automatically imply permission to modify medical or safety rules.

---

## Admin Restrictions

The Admin cannot:

- Access passwords or authentication secrets.
- Impersonate users without an explicitly defined and audited support mechanism.
- Modify user data without authorization.
- Bypass security controls.
- Modify protected medical or safety logic unless explicitly authorized by future governance requirements.

---

# 4. Medical Reviewer — Future Role

## Status

**Future Scope**

The Medical Reviewer will not be part of the initial MVP.

---

## Purpose

The Medical Reviewer role is intended to provide medical governance over the product in future versions.

---

## Potential Permissions

A Medical Reviewer may eventually be able to:

- Review medical content.
- Review assessment rules.
- Review red-flag definitions.
- Review safety-related rules.
- Review possible explanations.
- Approve or reject medical content changes.
- Review the history of medical-content changes.

---

## Future Restrictions

A Medical Reviewer should not automatically have access to:

- User passwords.
- Authentication secrets.
- Unnecessary personal information.
- Administrative system-management functions unrelated to medical governance.

Access should follow the principle of least privilege.

---

# Permission Matrix

The following matrix defines the high-level permission boundaries.

| Capability | Guest | Registered User | Admin | Medical Reviewer |
|---|---:|---:|---:|---:|
| Start Initial Assessment | Yes | Yes | Yes* | Future |
| Provide Symptoms | Yes | Yes | Yes* | Future |
| Use Text Input | Yes | Yes | Yes* | Future |
| Use Voice Input | Yes | Yes | Yes* | Future |
| Use 3D Body | Yes | Yes | Yes* | Future |
| Answer Questions | Yes | Yes | Yes* | Future |
| View Assessment Result | Yes | Yes | Yes* | Future |
| View Next-Step Guidance | Yes | Yes | Yes* | Future |
| Save Assessment History | No | Yes | No | Future |
| View Own History | No | Yes | No | Future |
| Manage Own Health Context | No | Yes | No | Future |
| Manage Own Chronic Conditions | No | Yes | No | Future |
| View Other Users' Data | No | No | Controlled | Restricted |
| Manage Users | No | No | Yes | No |
| Manage System Content | No | No | Controlled | Future |
| Configure Assessment | No | No | Controlled | Future |
| Manage Medical Rules | No | No | No | Future |
| Manage Safety Rules | No | No | No | Future |
| Access Admin Dashboard | No | No | Yes | No |
| Access Medical Review Tools | No | No | No | Future |

\* Administrative access to the end-user assessment flow does not mean the Admin receives special medical privileges. Administrative permissions and user assessment permissions should remain conceptually separate.

---

# Authorization Principles

## 1. Least Privilege

Every role should receive only the permissions required to perform its intended responsibilities.

---

## 2. User Data Isolation

A Registered User must only be able to access their own personal data and assessment history unless a future explicitly authorized workflow allows otherwise.

```text
User A
  ↓
Own Data Only

User B
  ↓
Own Data Only
```

---

## 3. Administrative Separation

Administrative permissions should be separated from normal user permissions.

An Admin should not automatically receive unrestricted access to sensitive user information.

---

## 4. Future Medical Governance Separation

Medical Reviewer permissions should be separated from general administrative permissions.

The future Medical Reviewer should focus on:

```text
Medical Content
Assessment Rules
Safety Rules
        ↓
Medical Governance
```

rather than general platform administration.

---

# Guest-to-Registered Transition

The initial user journey follows this model:

```text
                    User
                      │
                      ↓
              Guest Experience
                      │
                      ↓
              Initial Assessment
                      │
                      ↓
                 Results
                      │
                      ↓
        ┌─────────────┴─────────────┐
        │                           │
        ↓                           ↓
   Leave Product              Register / Login
                                    │
                                    ↓
                           Registered User
                                    │
                                    ↓
                          Extended Features
```

The first assessment should provide meaningful value before requiring registration.

After the initial guest assessment, authentication is required for persistent and extended functionality.

---

# Session and Data Boundary

Guest and Registered User sessions should be treated differently.

## Guest Session

The system may maintain temporary session data required to complete the initial assessment.

Guest session data should not automatically become a permanent medical history record.

## Registered Session

Authenticated assessment data may be associated with the user's account and stored according to the product's data-retention and privacy requirements.

The exact retention behavior will be defined in the Data Requirements and Security phases.

---

# Authorization Rules

The following rules are established:

```text
ROLE-001
A Guest User can perform the initial assessment without authentication.

ROLE-002
A Guest User cannot access persistent assessment history.

ROLE-003
A Registered User can perform new assessments.

ROLE-004
A Registered User can access only their own persistent assessment data.

ROLE-005
An Admin can access administrative functionality according to explicitly defined permissions.

ROLE-006
Administrative access does not automatically grant unrestricted access to user medical data.

ROLE-007
Medical Reviewer is a future role and is not implemented in the initial MVP.

ROLE-008
Medical Reviewer permissions will focus on medical governance rather than general administration.

ROLE-009
Authorization must follow the principle of least privilege.

ROLE-010
Sensitive actions should be auditable where appropriate.
```

---

# Open Questions for Later Requirements

The following decisions will be finalized during later requirements analysis:

- Exact Admin permissions.
- Whether Admin can view specific user assessment information.
- Guest session expiration.
- Guest data retention.
- Exact registration trigger.
- Whether a guest can continue the same assessment after registration.
- Account verification requirements.
- Password recovery behavior.
- Exact Medical Reviewer permissions in future versions.
- Audit requirements for privileged actions.

These questions should not be resolved through implementation assumptions.

---

# Relationship to Next Phase

The role and permission definitions provide the foundation for:

```text
User Roles
     ↓
User Journeys
     ↓
Functional Requirements
     ↓
Authorization Requirements
     ↓
Acceptance Criteria
     ↓
Security Design
```

The next document will define the major user journeys and system flows for each primary role.