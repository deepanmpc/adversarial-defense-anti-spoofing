# ClaimShield — Adversarial Defense & Anti-Spoofing Strategy

> **Guidewire DEVTrails 2026 — Phase 1 Submission**
> Parametric Insurance Platform for Gig & Delivery Workers

**Live Demo:** [Open index.html in browser after cloning](#how-to-run)

---

## What Is This Project?

ClaimShield is a prototype adversarial defense system for parametric insurance platforms serving gig and delivery workers. It addresses a critical vulnerability: coordinated GPS spoofing attacks where fraud rings of 500+ workers simultaneously fake distress locations to drain platform liquidity pools.

**The core insight:** GPS coordinates are a claim, not proof. Traditional platforms fail because they treat GPS as ground truth. ClaimShield replaces GPS verification with behavioral coherence scoring — verifying whether a claim is physically, contextually, and network-consistent with a real human being in genuine distress.

> *Fraud must fake signals. Reality produces them naturally.*

---

## Features

- **Interactive Claim Analyzer** — Toggle 6 device signals (GPS, motion, Wi-Fi BSSID, cell tower, APK signature, cluster membership) and see a real-time fraud score with verdict and reasoning
- **5-Layer Detection Pipeline** visualization — Rule Engine → Sensor Fusion → Graph Intelligence → Computer Vision → LLM Reasoning
- **Tiered Response Model** — Auto-Approve / Soft Challenge / Human Review with grace queue logic for offline workers
- **Expected Impact Metrics** — >90% ring detection, 70–85% false payout reduction, <3% false positive rate
- **Failure Mode Awareness** — System designed to fail safe, not fail open

---

## How to Run

No server, no dependencies, no build step required.

```bash
git clone https://github.com/deepanmpc/adversarial-defense-anti-spoofing.git
cd adversarial-defense-anti-spoofing
open index.html        # macOS
# or double-click index.html in your file explorer (Windows/Linux)
```

That's it. The prototype runs entirely in the browser.

---

## The Problem With GPS-Based Verification

**GPS coordinates are a claim, not proof. We verify behavior, not location.**

Traditional parametric insurance platforms treat GPS as ground truth. That assumption fails the moment a $2 spoofing app enters the picture. A worker sitting at home can broadcast a perfect storm-zone coordinate, pass every threshold check, and trigger a payout — all without the platform raising a flag.

The deeper problem is architectural: systems designed to detect *individual* bad actors are not built to detect *coordinated* ones. A single fraudulent claim looks like noise. Five hundred simultaneous fraudulent claims from the same spoofing toolkit, triggered by the same Telegram alert, at the same geofence — that's a signal. Most platforms never look for it.

This system does.

---

## What Makes This Different

Most fraud detection pipelines ask: *"Is this claim suspicious?"*

This system asks: *"Is this claim consistent — across physics, behavior, history, and network context — with a human being in distress in a real weather event?"*

Key distinctions:

- **Proactive, not reactive.** Fraud rings are detected at the network level before individual claims are fully processed.
- **Multi-modal, not single-signal.** Spoofing GPS is easy. Simultaneously spoofing GPS, accelerometer patterns, network signal quality, Wi-Fi BSSID, battery telemetry, and historical route data is not.
- **Designed for coordinated attacks.** The graph intelligence layer specifically targets synchronized, ring-level fraud behavior.
- **Cost-aware by design.** Heavy compute is only triggered when lighter layers cannot resolve the claim.

---

## Layer 1: Rule Engine — Zero-Latency Hard Checks

The first line of defense is deterministic and runs in microseconds. No model inference. No latency.

| Rule | Condition | Action |
|---|---|---|
| Velocity violation | Last verified location > 30 km away < 10 mins ago | Instant escalation |
| Pre-alert volume spike | Claim surge before official weather alert | Ring flag triggered |
| Network contradiction | Storm-zone GPS + strong stable connection | Soft challenge |
| Temporal clustering | 5+ claims from same geofence within 5-minute window | Graph analysis triggered |
| Device integrity | Root detection positive or known spoofing APK | Immediate escalation |

---

## Layer 2: Sensor Fusion — Behavioral Coherence Scoring

**Real distress is noisy. Fraud is suspiciously clean.**

| Signal | Genuine Pattern | Fraud Pattern |
|---|---|---|
| Accelerometer / gyroscope | Irregular, weather-consistent motion | Near-stationary or artificially smooth |
| Battery drain rate | Elevated — cold environment, active GPS | Normal or on charge |
| RF signal quality | Degraded — storm interference | Strong, stable home network |
| Cell tower triangulation (MLAT) | Corroborates GPS within drift | Contradicts spoofed coordinate |
| Wi-Fi BSSID geolocation | Field-consistent SSIDs | Home router SSID detected |
| Historical delivery corridors | Prior routes in this zone | No historical presence |
| App interaction cadence | Distress-consistent behavior | Scripted or absent |

A **gradient-boosted classifier** (LightGBM) produces a `fraud_probability` score from this feature vector.

Spoofing GPS is trivial.
Spoofing **multi-modal physical reality across independent sensors** is exponentially harder.
Our system exploits this asymmetry.

---

## Layer 3: Graph Intelligence — Detecting the Ring, Not Just the Individual

**This is the layer that catches coordinated fraud.**

```
Workers --> Claims --> Devices --> Locations --> Weather Events
```

**Louvain community detection** surfaces dense subgraphs — clusters too tightly interconnected to be coincidental. This enables **early interception** — the system can freeze an entire fraud cluster before the first payout in that cluster is processed.

---

## Layer 4: Computer Vision — Evidence Integrity Check

1. **EXIF consistency** — GPS metadata vs claimed location
2. **Manipulation detection** — Perceptual hashing + compression artifact analysis
3. **Environmental plausibility** — Scene classification (storm vs living room)

---

## Layer 5: LLM Reasoning — Auditability, Not Detection

The LLM layer has one job: **make fraud decisions explainable to humans.**

Example output:
> *"Claim flagged. GPS places worker in storm zone, but cell tower triangulation shows location 12 km away. Accelerometer shows no movement for 40 minutes. Device is part of a cluster of 23 workers with identical app install signatures who submitted claims within a 4-minute window. Confidence: coordinated fraud."*

---

## Compute Architecture: Efficiency Under Load

```
Every Claim
    |
    v
[Rule Engine]  <-- microseconds, deterministic
    |
    +-- Clean? --> Coherence Score (LightGBM)  <-- milliseconds
                       |
                       +-- Low risk?  --> Auto-Approve
                       +-- Med risk?  --> Soft Challenge
                       +-- High risk? --> Graph Analysis  <-- on-demand
                                              |
                                              +-- CV Layer (if evidence)
                                              +-- LLM Summary for Review
```

- ~70% of legitimate claims never exit the Rule Engine + LightGBM layers
- Graph analysis triggered once per cluster, not per claim
- LLM inference runs only on escalated claims

---

## UX: Do Not Penalize Uncertainty — Resolve It

### Tier 1 — Auto-Approve `(fraud_score < 0.25)`
Instant payout within SLA. Zero friction.

### Tier 2 — Soft Challenge `(0.25 ≤ fraud_score < 0.65)`
One-tap confirmation. If offline due to network degradation → **2-hour grace queue**. No penalty.

### Tier 3 — Human Review `(fraud_score ≥ 0.65)`
LLM summary pre-loaded for reviewer. No punitive action until human confirms. Cleared claims leave zero permanent record.

### Appeals
48-hour human review SLA. False positive rate tracked as first-class KPI.

---

## ⚠️ Failure Mode Handling

The system is designed to **fail safe, not fail open.**

| Failure | Fallback Behavior |
|---|---|
| Graph analysis delayed | Per-claim ML + rule engine continue independently |
| Sensor data partially missing | LightGBM degrades gracefully on available signals |
| LLM service unavailable | Deterministic fraud scores sufficient for auto-approve/escalate |
| CV layer unavailable | Evidence held for manual review; claim not blocked |
| Network outage for worker | Grace queue activated; resumes on reconnection |

---

## 📊 Expected Impact

- Detects **>90% of coordinated fraud rings** before payout stage
- Reduces **false payouts by 70–85%** in high-risk geographies
- Maintains **<3% false positive rate** through tiered UX and grace handling
- Reduces **human review load by ~40%** via LLM-assisted explanations

---

## Why This System Wins

| Capability | Standard Platform | This System |
|---|---|---|
| GPS spoofing detection | GPS threshold check | Multi-signal behavioral coherence |
| Individual fraud | Rule-based flags | ML anomaly detection across 7+ signals |
| Coordinated ring detection | None | Real-time graph clustering + early interception |
| Explainability | Black-box score | LLM plain-language audit trail |
| False positive protection | None / manual | Tiered response + grace queue + appeals |
| Compute efficiency | Uniform cost per claim | Tiered — heavy layers on-demand only |
| Infrastructure resilience | Single point of failure | Graceful degradation at every layer |

> **We don't try to out-detect fraud — we make it economically and operationally unscalable.**

---

*Built for Guidewire DEVTrails 2026 — Phase 1. Deployable as a pluggable defense layer with no GPS infrastructure changes required.*
