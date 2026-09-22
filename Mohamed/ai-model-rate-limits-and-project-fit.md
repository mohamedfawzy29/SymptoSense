# AI Model Rate Limits & Project Fit — SymptoSense

**Document Version:** 1.0
**Status:** Reference — For Team Review
**Last Updated:** 2026-09-22
**Related Documents:** `15-ai-reuirements.md`, `13-non-functional-requirements.md`

---

## 1. Purpose

This document evaluates the models currently visible on the team's Google AI Studio account against SymptoSense's needs: Arabic/Egyptian Arabic conversation, structured symptom extraction, and the throughput required to support real assessment sessions.

It is meant to sit alongside the AI Requirements document (Section 11–14), not replace it. Google AI Studio's Gemini/Gemma models are **not** on the currently approved candidate shortlist (Qwen, Llama, MedGemma) — this doc treats them as a possible prototyping option and flags where they align or conflict with the approved AI strategy (`AI-021` — Open-Weight Model direction).

---

## 2. Before Reading the Numbers

### 2.1 These limits are a shared pool, not per-user

RPM (requests/minute), TPM (tokens/minute), and RPD (requests/day) apply to the **entire application**, not to each individual user. If a model shows `15 RPM`, that means the whole app — every user combined — gets 15 requests per minute total, not 15 requests per user.

### 2.2 This is the free tier

Every non-zero quota in the source table is small enough (single-digit to low double-digit RPM, 20–500 RPD for the chat models) to be Google AI Studio's **free tier**. These numbers are appropriate for prototyping and internal testing, not for a production launch.

### 2.3 Assumptions used for the estimates below

| Assumption | Value | Why |
|---|---|---|
| Requests per full assessment | ~10 | Per `AI Requirements` §4 & `Business Rules` (BR-019, BR-025): initial extraction + a handful of dynamic follow-up questions + result explanation |
| Requests/minute per actively-chatting user | ~3 | Updated to model a faster-paced conversation — the assistant asks up to 3 questions per minute per user instead of 2 |
| Concurrent users supported | RPM ÷ 3 | Rounded down |
| Assessments possible per day | RPD ÷ 10 | |
| Unique users per month | (RPD ÷ 10) × 30 | Assumes each user completes one assessment, one day |

These are estimates for planning purposes, not guaranteed throughput — actual token/request usage will vary once the real conversation flow (health context + 3D localization payload + dynamic questions per `AICHAT-003`) is implemented.

---

## 3. Quick Comparison Table

| Model | RPM | TPM | RPD | ~Concurrent users | ~Assessments/day | ~Unique users/month | Time to exhaust daily quota |
|---|---:|---:|---:|---:|---:|---:|---:|
| Gemini 2.5 / 3 / 3.5 / 3.6 / 3.7 / 3.8 Flash | 5 | 250K | 20 | ~1 | ~2 | ~60 | 4 min |
| Gemini 2.5 Flash Lite | 10 | 250K | 20 | ~3 | ~2 | ~60 | 2 min |
| **Gemini 3.1 Flash Lite / 3.5 Flash Lite** | **15** | **250K** | **500** | **~5** | **~50** | **~1,500** | **33 min** |
| **Gemma 4 26B / 31B** | **30** | **16K** | **14,400** | **~10** | **~1,440** | **~43,200** | **~8 hrs** |
| Gemini Embedding 1 / 2 | 100 | 30K | 1,000 | n/a — not a chat model | n/a | n/a | 10 min |

Models not included above (Antigravity, TTS models, Live/audio models, image models, `Gemini 2.5 Pro`, `Gemini 3.1 Pro`, Computer Use, Deep Research, etc.) are covered in §4.5–4.6 or currently sit at `0/0/0` on the account, meaning they're disabled and unusable until enabled.

---

## 4. Model-by-Model Breakdown

### 4.1 Standard Flash Models
*(Gemini 2.5 Flash, 3 Flash, 3.5 Flash, 3.6 Flash, 3.7 Flash, 3.8 Flash)*

**Limits:** 5 RPM · 250K TPM · 20 RPD
**Advantages:** Google's general-purpose conversational models, strong reasoning-per-cost tradeoff, large context window (TPM is never the bottleneck here).
**Fit for SymptoSense:** Poor as a working prototype tier — 20 requests/day is only ~2 full assessments before the app stops responding for *everyone* until the next day. Only useful for a single-developer smoke test, not team or user testing.

---

### 4.2 Flash Lite Models — Best Free-Tier Option

#### 4.2.1 Gemini 2.5 Flash Lite
**Limits:** 10 RPM · 250K TPM · 20 RPD
Doubles the per-minute throughput of standard Flash but the daily cap (20 RPD) is unchanged, so it still only supports ~2 assessments/day. Marginal improvement over §4.1.

#### 4.2.2 Gemini 3.1 Flash Lite & Gemini 3.5 Flash Lite
**Limits:** 15 RPM · 250K TPM · 500 RPD
**Advantages:** By far the strongest chat-model quota on the account — 25x the daily cap of the standard Flash tier, while keeping the large 250K TPM window. "Lite" models trade some reasoning depth for speed and cost, which is a reasonable tradeoff for symptom extraction and clarification questions (bounded, structured tasks) rather than open-ended reasoning.
**Fit for SymptoSense:** The most usable option in this list for internal team testing and early UX prototyping — roughly 50 completed assessments/day, ~1,500/month. Still far short of the `SCAL-002` target of 500 concurrent sessions, but adequate for a closed pilot or demo.

---

### 4.3 Gemma 4 (26B / 31B) — Open-Weight Candidate

**Limits:** 30 RPM · 16K TPM · 14,400 RPD
**Advantages:**
- By a wide margin the highest RPM and RPD of anything on the account — ~1,440 assessments/day possible.
- **Open-weight**, same category as the team's approved candidates (Qwen, Llama, MedGemma per `AI-021`). Self-hostable later, so early testing on AI Studio doesn't lock the team into a closed-model dependency.
- Two sizes available (26B/31B) for a speed-vs-quality comparison, consistent with how the AI Requirements doc already treats Qwen/Llama/MedGemma as multiple variants to benchmark.

**Caution:**
- **TPM is only 16K**, far below the 250K the Gemini models get. A single request carrying a full conversation history + health context + medical knowledge snippets could approach that ceiling — this needs to be tested with realistic payload sizes, not assumed safe just because RPM/RPD look generous.
- Not one of the three officially shortlisted candidates. Egyptian Arabic performance, medical terminology handling, and structured-output reliability are **unverified** — the same "must be tested" caveat the AI Requirements doc already applies to Qwen/Llama/MedGemma (§12) should apply here before relying on it for anything beyond internal testing.

**Fit for SymptoSense:** Strong candidate to add as a **fourth open-weight option in the evaluation matrix** (AI Requirements §12), specifically because it's already accessible without additional procurement and its free-tier quota is generous enough for genuine multi-user pilot testing rather than single-developer smoke tests.

---

### 4.4 Embedding Models
*(Gemini Embedding 1, Gemini Embedding 2)*

**Limits:** 100 RPM · 30K TPM · 1,000 RPD
**Advantages:** Not a chat/completion model — used to convert text into vectors for semantic search/retrieval.
**Fit for SymptoSense:** Not relevant to the conversational assessment flow itself, but potentially useful later for searching the **Medical Knowledge Base** (`BR-075`, `BR-081`) or matching symptom descriptions against the approved MVP Medical Coverage documentation — worth keeping in mind for the Assessment Engine's data layer rather than the AI conversational layer.

---

### 4.5 Live API / Voice Models
*(Gemini 2.5 Flash Native Audio Dialog, Gemini 3 Flash Live, Gemini 3.5 Live Translate, Gemini 3.5 Transcribe / Transcribe Live, Gemini 3.8 Live, Gemini 3.8 Live Extended Thinking)*

**Limits:** Mostly "Unlimited" RPM/RPD with capped TPM (20K–65K), except Transcribe (3 RPM / 25 RPD).
**Fit for SymptoSense:** Likely **not the right tool** for this architecture. The AI Requirements doc (§4.4) deliberately separates Speech-to-Text from the AI understanding layer — voice is converted to text first, then that text flows through the same natural-language pipeline as typed input. A dedicated STT service fits that architecture better than routing voice through a Live/real-time dialog API designed for open-ended spoken conversation.

---

### 4.6 Disabled / Zero-Quota Models

The following show `0/0/0` on the account and are not currently usable: Antigravity's Deep Research Pro Preview, Gemini 2 Flash, Gemini 2 Flash Lite, Computer Use Preview, Gemini 2.5 Pro, Gemini 2.5 Pro TTS, Gemini 3.1 Pro, all "Nano Banana" image models, Gemini Omni (Flash / 1.1 Flash), Lyria 3 (Clip/Pro), Veo 3 Fast Generate. These would need to be explicitly enabled/requested before they're usable, and none are core to the text-based assessment flow regardless.

---

## 5. Fit Against SymptoSense Requirements

| Requirement | Target | Best model here | Gap |
|---|---|---|---|
| `PERF-003` — AI response latency | Streaming/display within 2.5s | Not measurable from this table (latency ≠ rate limit) | Needs separate latency testing per candidate |
| `SCAL-002` — Concurrent sessions | 500 concurrent | Gemma 4 (~10 concurrent est.) | ~50x short |
| `AI-021` — Open-Weight strategy | Qwen / Llama / MedGemma | Gemma 4 (open-weight, not yet shortlisted) | Aligned in spirit, not on the approved list |
| `AI-004` — Egyptian Arabic evaluation | Required before production use | None verified here | All models in this table still need Egyptian Arabic testing |

---

## 6. Recommendation

- **For internal prototyping / UX testing today:** Gemini 3.1 Flash Lite or 3.5 Flash Lite — best out-of-the-box balance of RPM and RPD among the closed Gemini models.
- **For a slightly larger pilot, and to stay aligned with the approved Open-Weight direction:** Gemma 4 (26B or 31B) — highest quota by far, and it's a legitimate candidate to add to the AI Requirements evaluation matrix (§12) rather than a detour from it. Its 16K TPM ceiling should be stress-tested with realistic assessment payloads before relying on it.
- **For production (500 concurrent sessions per `SCAL-002`):** None of these free-tier quotas are sufficient. The team will need either a paid Google tier or — more consistent with the existing `AI-021` decision — self-hosted infrastructure for whichever of Qwen, Llama, MedGemma, or Gemma wins the formal evaluation.

This document should be treated as a **rate-limit and quota reference**, not a substitute for the model evaluation strategy already defined in `15-ai-reuirements.md` §15 (Arabic understanding, symptom extraction accuracy, hallucination rate, safety behavior, etc.).
