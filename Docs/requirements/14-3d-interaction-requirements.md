# 3D Interaction Requirements Document

**Document Version:** 1.0  
**Status:** Approved  
**Last Updated:** 2026-08-19  

---

## 1. Overview & Purpose

This document defines the functional and technical requirements for the interactive **3D Human Body Model** within **SymptoSense**. The 3D model serves as a primary, intuitive visual entry point for symptom localization, bridging the gap between a user's physical sensation and conversational AI assessment.

---

## 2. Progressive Anatomical Navigation Engine

The system MUST enforce a 4-level hierarchical selection pattern to ensure precise symptom localization:

[Level 1: Whole Body View]
│
▼
[Level 2: Major Body Region]  (e.g., Head & Neck, Torso, Limbs)
│
▼
[Level 3: Sub-Region]          (e.g., Chest, Upper Abdomen, Shoulder)
│
▼
[Level 4: Specific Location]    (e.g., Upper Left Chest, Right Epigastric)




### Navigation Rules:
* **3D-NAV-001 (Default State):** On assessment initialization, the system MUST render the full 3D human body in a neutral standing position (A-pose or T-pose).
* **3D-NAV-002 (Hierarchical Selection):** Clicking/tapping on any mesh component MUST smoothly zoom and rotate the camera focus to that specific region (Level 1 → Level 2 → Level 3 → Level 4).
* **3D-NAV-003 (Breadcrumb Navigation):** The interface MUST display a clear Arabic breadcrumb bar allowing users to navigate back to parent body regions (e.g., `الجسم بالكامل` ← `الجذع` ← `الصدر`).
* **3D-NAV-004 (Reset Camera):** A prominent "Reset View" (`إعادة ضبط العرض`) button MUST be available to return the 3D canvas to the default full-body orientation.

---

## 3. Visual Feedback & Gesture Controls

* **3D-CTRL-001 (Touch & Pointer Controls):** The 3D viewport MUST support the following gesture interactions across touch devices and desktop environments:
  * **Rotate (Orbit):** Single-finger drag / Left-click drag.
  * **Pan:** Two-finger drag / Right-click drag.
  * **Zoom:** Pinch gesture / Mouse scroll wheel.
* **3D-CTRL-002 (Hover & Selection States):**
  * **Hover State:** Sub-regions MUST highlight subtly on mouse hover or touch target focus.
  * **Selected State:** The active region MUST be highlighted with a distinct primary visual outline/glow.
* **3D-CTRL-003 (Bounding Box Constraints):** Camera movement MUST be constrained within predefined bounding box limits to prevent camera clipping inside the 3D model or losing sight of the model.

---

## 4. Anatomical Region Mapping Payload (AI Integration)

Once a user confirms a location on the 3D model, the 3D Engine MUST generate a structured JSON payload to feed directly into the Conversational AI Engine.

### Standard Payload Specification:
```json
{
  "anatomical_selection": {
    "body_side": "Anterior",
    "major_region": "Torso",
    "sub_region": "Chest",
    "specific_location": "Upper Left Chest",
    "arabic_label": "أعلى اليسار من الصدر",
    "anatomical_code": "CHEST_UPPER_LEFT"
  }
}
```
* **3D-DATA-001 (Standardized Mapping): All 3D mesh identifiers MUST map to a predefined anatomical taxonomy dictionary to ensure consistency when passed to the LLM.

* **3D-DATA-002 (Gender Model Matching): The 3D model geometry MUST dynamically adjust based on the mandatory demographic context provided (Male / Female body model).



5. Performance & Mobile Constraints
* **3D-PERF-001 (Asset Optimization): The 3D asset format MUST use optimized binary .gltf / .glb files with mesh compression (e.g., Draco compression). File size MUST NOT exceed 3.5 MB total.

* **3D-PERF-002 (Frame Rate): The 3D rendering loop MUST maintain a minimum of 30 FPS on mid-tier mobile devices and 60 FPS on desktop browsers using WebGL / Three.js.

* **3D-PERF-003 (Alternative Fallback Input): If the user's browser or device fails to render WebGL, the system MUST gracefully fall back to a 2D interactive SVG body map without interrupting the assessment journey.



6. Arabic Localization & RTL Rules
* **3D-LOC-001 (Arabic Region Labels): All tooltips, anatomical names, and selection prompts MUST be rendered natively in Arabic.

* **3D-LOC-002 (Anatomical Accuracy in Arabic): Anatomical labels MUST use clear, everyday Arabic terms alongside standard medical terms where necessary (e.g., أعلى البطن - فم المعدة).



## 7. Requirement Traceability Matrix

| Requirement ID | Module | Primary User | Associated Journey |
| --- | --- | --- | --- |
| 3D-NAV-001 to 004 | 3D Engine | Guest / Registered | Journey 1 & 2 (Step 3) |
| 3D-CTRL-001 to 003 | Viewport UI | Guest / Registered | Journey 1 & 2 (Step 3) |
| 3D-DATA-001 to 002 | AI Bridge | System Engine | Journey 1 & 2 (Step 3 to Step 4) |
| 3D-PERF-001 to 003 | Performance | WebGL / Engine | NFR Performance (PERF-002) |
