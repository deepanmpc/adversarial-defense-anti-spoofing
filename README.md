# Adversarial Defense & Anti-Spoofing Strategy

> **Guidewire DEVTrails 2026 — Phase 1 Submission**
> Parametric Insurance Platform for Gig & Delivery Workers

---

## Overview

This document outlines our architectural response to the **Market Crash** threat scenario: a coordinated ring of 500 delivery workers using GPS-spoofing applications to fake distress locations and drain platform liquidity pools via false parametric insurance payouts.

Simple GPS verification is officially obsolete. Our platform replaces it with a **multi-layer behavioral trust pipeline** that processes every claim through four complementary intelligence layers before any payout decision is made.

---

## Adversarial Defense & Anti-Spoofing Strategy

### 1. Differentiation: Genuine Stranded Worker vs. GPS Spoofer

We replace single-point GPS verification with a **multi-signal behavioral trust score** computed at claim time. A genuine stranded worker and a fraudster spoofing from home produce fundamentally different *behavioral fingerprints* across these dimensions:

| Signal | Genuine Worker | Spoofer (at home) |
|---|---|---|
| Accelerometer / gyroscope | Irregular, weather-consistent motion | Stationary or unnaturally smooth |
| Battery drain rate | Elevated (cold, screen-on, background GPS) | Normal / on charge |
| Network signal strength | Degraded — consistent with storm zone | Strong, stable home connection |
| Cell tower + Wi-Fi BSSID geo | Corroborates GPS location | Contradicts spoofed coordinate |
| Historical delivery corridor | Worker has prior routes in this zone | No historical presence |

This anomaly score feeds into an **LLM-based reasoning layer** that receives a structured claim summary — location, weather severity, device signals, historical behavior — and produces a plain-language justification for its fraud/genuine classification. This gives human reviewers an *auditable reasoning trail*, not just a black-box score.

---

### 2. The Data: Detecting a Coordinated Fraud Ring

A lone bad actor is detectable. A *ring of 500* is detectable at the **network and temporal level**. Beyond GPS coordinates, we analyze:

- **Temporal clustering:** Multiple claims from the same geofence within a 5-minute window. Organic emergencies don't cluster this tightly.
- **Device fingerprinting:** Spoofing apps on rooted devices produce repeatable OS/install signatures. A hash of `(device_id + OS build + app install timestamp)` surfaces ring members using the same tooling.
- **Cross-signal contradiction:** A claimed GPS in a red-alert storm zone paired with a strong home Wi-Fi BSSID is a hard, machine-verifiable lie.
- **Physical plausibility check (rule engine):** If a worker's last verified location was 40 km away 8 minutes ago, a rule fires immediately — no ML inference needed.
- **Velocity spike detection:** Claim volume rising in a region *before* official weather escalation signals coordinated tip-off behavior (e.g., Telegram group coordination) rather than organic response to conditions.
- **Graph anomaly detection:** Workers and claim events are modeled as a bipartite graph. Ring behavior produces dense, temporally correlated subgraphs. Community detection (Louvain algorithm) surfaces coordinated clusters in near real-time.
- **Computer vision on submitted evidence:** If workers upload photos as supporting evidence, a vision model checks for EXIF metadata consistency (GPS, timestamp), image manipulation artifacts, and environmental plausibility — does the background match a storm, or a living room?

---

### 3. The UX Balance: Flagging Without Punishing Honest Workers

The system must never become a tool that harms the workers it exists to protect. We use a **tiered response model** so friction is proportional to actual risk:

#### Tier 1 — Auto-Approve `(fraud_score < 0.25)`
Claim processes instantly. Payout within SLA. Zero friction for the clear majority of legitimate claims.

#### Tier 2 — Soft Challenge `(0.25 ≤ fraud_score < 0.65)`
A one-tap confirmation prompt is sent to the worker. If the worker is **offline** (network drop in bad weather — a completely legitimate scenario), the claim enters a **grace queue for up to 2 hours**, resuming verification automatically when connectivity returns. No rejection, no penalty for going dark in a storm.

#### Tier 3 — Human Review `(fraud_score ≥ 0.65)`
Claim is routed to a reviewer dashboard with the LLM-generated reasoning summary pre-loaded. The worker receives a transparent message:

> *"Your claim is under brief manual review due to unusual activity in your area — typically resolved within [X] hours."*

No punitive action occurs until a human confirms fraud. A single flagged-but-cleared claim leaves **zero permanent record** on the worker's trust profile.

#### Appeals
Every rejected claim generates a one-tap appeal with a **48-hour human review SLA**. False positive rate is tracked as a first-class product KPI — not an afterthought.

---

## Architecture Summary

```
Claim Submitted
       |
       v
+-----------------------------+
|   Rule Engine (hard checks) |  <- Velocity, physical plausibility
+-------------+---------------+
              |
              v
+-----------------------------+
|  ML Anomaly Detection Model |  <- Device signals, behavioral fingerprint
+-------------+---------------+
              |
              v
+-----------------------------+
|  Computer Vision Layer      |  <- Evidence photo validation
+-------------+---------------+
              |
              v
+-----------------------------+
|  LLM Reasoning Layer        |  <- Auditable fraud/genuine justification
+-------------+---------------+
              |
              v
     Fraud Score -> Tiered Response (Auto-Approve / Soft Challenge / Human Review)
```

---

## Tech Stack

- **ML Anomaly Detection:** LightGBM / XGBoost trained on behavioral claim signals
- **LLM Reasoning:** Prompted LLM (structured claim summary → plain-language audit trail)
- **Computer Vision:** EXIF analysis + manipulation detection on uploaded evidence
- **Rule Engine:** Physical plausibility & velocity checks (deterministic, zero-latency)
- **Graph Analytics:** Louvain community detection for ring identification

---

*This strategy is designed to be adversarially robust, operationally fair, and deployable as a pluggable defense layer within the existing microservice architecture — no GPS infrastructure changes required.*
