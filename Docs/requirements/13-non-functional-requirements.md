Markdown
# Non-Functional Requirements Document (NFRD)

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-19  

---

## 1. Overview & Purpose

This document specifies the Non-Functional Requirements (NFRs) for **SymptoSense**. While functional requirements define *what* the system does, non-functional requirements define *how well* the system performs, secures, scales, and delivers its capabilities.

Each requirement follows the format:
`[Category Code]-[Sequential Number]: Requirement Description`

---

## 2. Non-Functional Requirement Categories

---

### Category 1: Performance & Responsiveness (PERF)

* **PERF-001 (UI Response Time):** The web user interface MUST load initial assets within 2.0 seconds on standard 4G mobile connections.
* **PERF-002 (3D Model Loading & FPS):** The 3D human body model MUST load and render within 1.5 seconds, maintaining a minimum of 30 Frames Per Second (FPS) on mid-tier mobile and desktop devices.
* **PERF-003 (Conversational AI Latency):** AI assessment follow-up responses and dynamic questions MUST begin streaming or displaying within 2.5 seconds of user input.
* **PERF-004 (Voice Processing Speed):** Speech-to-Text (STT) conversion MUST process audio input and render corresponding text with a latency not exceeding 1.5 seconds for a 10-second voice clip.

---

### Category 2: Security, Privacy & Data Protection (SEC)

* **SEC-001 (Data in Transit Encryption):** All network communications between the client, backend APIs, and external LLM services MUST be encrypted using TLS 1.3 (HTTPS / WSS).
* **SEC-002 (Data at Rest Encryption):** All sensitive user medical data, health context, and assessment histories MUST be encrypted at rest using AES-256.
* **SEC-003 (Password Security):** User passwords MUST be hashed using industry-standard salted adaptive hashing algorithms (e.g., Argon2id or bcrypt) before storage.
* **SEC-004 (Data Minimization):** The system MUST enforce Privacy by Design, collecting only the minimal health information required to perform symptom assessment.
* **SEC-005 (Authorization & Isolation):** Strict Role-Based Access Control (RBAC) MUST prevent users from accessing or modifying other users' personal profiles or assessment histories (`FR-RULE-001`).
* **SEC-006 (Secret Management):** API keys (e.g., LLM vendor keys, database credentials) MUST NEVER be hardcoded in source code or client-side bundles and MUST be managed via secure environment variables.

---

### Category 3: Usability, Accessibility & Localization (USE)

* **USE-001 (Arabic-First Interface):** The application interface MUST be natively designed for Arabic speakers, featuring full Right-to-Left (RTL) layout support.
* **USE-002 (Dialect Comprehension):** The conversational AI MUST accurately process both Modern Standard Arabic (MSA) and Egyptian Arabic symptom descriptions.
* **USE-003 (Mobile-First Responsiveness):** The application MUST be fully responsive and optimized for screen resolutions ranging from modern mobile smartphones (360px width) to desktop displays.
* **USE-004 (Anatomical Clarity):** 3D model interaction MUST provide intuitive touch and mouse controls (pan, zoom, rotate) with clear visual feedback when a body region is selected.

---

### Category 4: Reliability, Availability & Fault Tolerance (REL)

* **REL-001 (System Availability):** The system core services MUST achieve 99.5% operational uptime during non-maintenance windows.
* **REL-002 (AI Service Fallback):** In the event of a primary LLM API failure or timeout, the system MUST gracefully inform the user and retry without crashing the user session or losing entered symptoms.
* **REL-003 (Data Integrity):** Assessment reports MUST be saved atomically; partial or corrupted assessment records MUST NOT be saved to user history.
* **REL-004 (Graceful Emergency Redirection):** Emergency detection logic MUST function independently of complex assessment flows to ensure immediate safety warnings are triggered even under partial system degradation.

---

### Category 5: Scalability & Maintainability (SCAL)

* **SCAL-001 (Modular Architecture):** The application codebase MUST follow a modular architecture, keeping the 3D rendering Engine, AI Assessment Service, and Core Authentication decoupled.
* **SCAL-002 (Concurrent Users):** The backend infrastructure MUST be capable of handling a minimum of 500 concurrent active assessment sessions without degradation in API response times.
* **SCAL-003 (Stateless API Design):** Backend services SHOULD be designed statelessly to support horizontal scaling via containerization (e.g., Docker) when user demand increases.

---

## 3. Summary Matrix

| Category | Primary Metric / Target | Enforcement Point |
| :--- | :--- | :--- |
| **Performance (PERF)** | UI < 2s, AI response < 2.5s, 3D FPS ≥ 30 | Frontend / AI Middleware |
| **Security (SEC)** | TLS 1.3, AES-256, JWT Auth, No Secrets in Code | Backend / Database / Infra |
| **Usability (USE)** | Arabic RTL, Mobile-First, MSA & Egyptian Arabic | UI/UX & Prompt Engineering |
| **Reliability (REL)** | 99.5% Uptime, Graceful LLM Fallback | Architecture / Infrastructure |
| **Scalability (SCAL)** | Modular Structure, Stateless Backend | System Design & Deployment |
