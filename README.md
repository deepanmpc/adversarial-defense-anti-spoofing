# ClaimShield — Adversarial Defense & Anti-Spoofing Platform

> **Guidewire DEVTrails 2026 — Phase 2: Scale**
> Parametric Insurance Platform for Gig & Delivery Workers

**Live Demo:** Open `index.html` in browser after cloning — zero dependencies, runs instantly.

---

## Why This Project Is Different

Phase 1 saw 40% of teams build near-identical "GigShield" clones. This project was not built from a prompt template.

The thinking here comes from someone who builds AI systems for vulnerable, underserved populations — specifically, a therapeutic AI companion (LaRa) for specially-abled children that must be simultaneously robust, fair, and transparent. That background shapes every design decision in ClaimShield:

- **Fairness under uncertainty** isn't a checkbox — it's an architectural constraint. The same principle that says "don't punish a child for a sensor misread" becomes "don't reject a gig worker's claim because of a network drop in a storm."
- **Explainability for humans** isn't a feature — it's the product. LLM output here is an audit trail for reviewers, not a detection mechanism.
- **Graceful degradation** is non-negotiable. Systems serving people who are already vulnerable must fail safe, not fail open.

This is what it looks like when fraud detection is designed by someone who thinks about trust, not just thresholds.

---

## What Is ClaimShield?

ClaimShield is a 5-layer adversarial defense system for parametric insurance platforms. It addresses a specific, economically devastating attack: coordinated GPS spoofing by fraud rings of 500+ workers who simultaneously fake distress locations to drain platform liquidity pools.

**The core insight:** GPS coordinates are a claim, not proof. Traditional platforms fail because they treat GPS as ground truth and evaluate claims in isolation. ClaimShield replaces GPS verification with **behavioral coherence scoring** — and replaces per-claim analysis with **network-level ring detection**.

> *Fraud must fake signals. Reality produces them naturally.*

---

## Features

- **Interactive Claim Analyzer** — Toggle 6 real device signals and watch a live fraud score compute in real time with full verdict reasoning
- **5-Layer Detection Pipeline** — Rule Engine → Sensor Fusion (LightGBM) → Graph Intelligence (Louvain) → Computer Vision → LLM Reasoning
- **Tiered Response Model** — Auto-Approve / Soft Challenge / Human Review with 2-hour grace queue for offline workers
- **Failure Mode Handling** — Every layer has a documented fallback. The system never blocks a legitimate claim due to its own infrastructure failure.
- **Expected Impact Metrics** — >90% ring detection, 70–85% false payout reduction, <3% false positive rate, ~40% reviewer load reduction

---

## How to Run

No server. No npm install. No build step.

```bash
git clone https://github.com/deepanmpc/adversarial-defense-anti-spoofing.git
cd adversarial-defense-anti-spoofing
open index.html        # macOS
# Windows/Linux: double-click index.html
```

Open in any modern browser. The entire prototype is a single HTML file.

---

## The Core Problem

**GPS coordinates are a claim, not proof. We verify behavior, not location.**

Traditional parametric insurance platforms treat GPS as ground truth. That assumption fails the moment a $2 spoofing app enters the picture. A worker sitting at home can broadcast a perfect storm-zone coordinate, pass every threshold check, and trigger a payout — all without the platform raising a flag.

The deeper problem is architectural: systems designed to detect *individual* bad actors are not built to detect *coordinated* ones. A single fraudulent claim looks like noise. Five hundred simultaneous claims from the same spoofing toolkit, triggered by the same Telegram alert, at the same geofence — that's a signal. Most platforms never look for it.

This system does.

---

## What Makes This Different From Every Other Submission

Most fraud detection pipelines ask: *"Is this claim suspicious?"*

This system asks: *"Is this claim consistent — across physics, behavior, history, and network context — with a human being in genuine distress?"*

| Design Principle | Standard Approach | ClaimShield |
|---|---|---|
| Verification | GPS threshold | Behavioral coherence across 7 signals |
| Scope | Per-claim | Network-level ring detection |
| Detection timing | Reactive (post-payout) | Pre-payout cluster interception |
| False positive handling | Manual review | Tiered UX + grace queue + appeals |
| Explainability | Score only | LLM plain-language audit trail |
| Infrastructure failure | Fail open | Fail safe at every layer |
| Cost scaling | Linear per claim | Sub-linear — rings share compute |

---

## Layer 1: Rule Engine — Zero-Latency Hard Checks

Deterministic. Runs in microseconds. Gates all downstream compute.

| Rule | Condition | Action |
|---|---|---|
| Velocity violation | Last verified location > 30 km away < 10 mins ago | Instant escalation |
| Pre-alert volume spike | Claim surge before official weather alert | Ring flag triggered |
| Network contradiction | Storm-zone GPS + strong stable connection | Soft challenge |
| Temporal clustering | 5+ claims from same geofence within 5 minutes | Graph analysis triggered |
| Device integrity | Root detection positive or known spoofing APK | Immediate escalation |

---

## Layer 2: Sensor Fusion — Behavioral Coherence Scoring

**Real distress is noisy. Fraud is suspiciously clean.**

| Signal | Genuine Pattern | Fraud Pattern |
|---|---|---|
| Accelerometer / gyroscope | Irregular, weather-consistent motion | Near-stationary or artificially smooth |
| Battery drain rate | Elevated — cold, screen-on, active GPS | Normal or on charge |
| RF signal quality | Degraded — storm interference | Strong, stable home network |
| Cell tower triangulation (MLAT) | Corroborates GPS within drift | Contradicts spoofed coordinate |
| Wi-Fi BSSID geolocation | Field-consistent SSIDs | Home router SSID detected |
| Historical delivery corridors | Worker has prior routes in zone | No historical presence |
| App interaction cadence | Distress-consistent frequent taps | Scripted or absent |

A **gradient-boosted classifier** (LightGBM) produces a `fraud_probability` from this feature vector, learning cross-signal inconsistencies — not just individual anomalies.

Spoofing GPS is trivial.
Spoofing **multi-modal physical reality across independent sensors** is exponentially harder.
Our system exploits this asymmetry.

---

## Layer 3: Graph Intelligence — Detecting the Ring, Not the Individual

**This is the differentiator. No standard platform does this.**

```
Workers --> Claims --> Devices --> Locations --> Weather Events
```

Fraud rings create structural signatures in this graph invisible at the claim level but unmistakable at the network level:

- **Device clusters** — Identical `(OS build + APK install hash)` across accounts = same spoofing toolkit
- **Temporal co-occurrence** — Synchronized claim windows inconsistent with independent organic distress
- **Pre-alert coordination** — Claim surge before official weather escalation = Telegram tip-off, not real emergency
- **Shared infrastructure** — Common IP ranges, recycled device IDs across supposedly independent workers

**Louvain community detection** surfaces dense, tightly-connected subgraphs in near real-time. When a cluster crosses a density threshold, every claim in it is frozen simultaneously.

This enables **early interception** — the entire fraud cluster is frozen before the first payout is processed.

---

## Layer 4: Computer Vision — Evidence Integrity Check

1. **EXIF consistency** — Does embedded GPS match the claimed location and timestamp?
2. **Manipulation detection** — Perceptual hashing + compression artifact analysis for edited/recycled images
3. **Environmental plausibility** — Scene classification: does this look like a storm, or a living room?

Secondary corroboration signal — adds weight to coherence score, provides concrete evidence for reviewers.

---

## Layer 5: LLM Reasoning — Auditability, Not Detection

The LLM has one job: **make decisions explainable to humans.**

It synthesizes upstream layer outputs into a plain-language summary a reviewer can act on in under 30 seconds.

Example:
> *"Claim flagged. GPS places worker in storm zone, but cell tower triangulation shows location 12 km away in a residential area. Accelerometer shows no movement for 40 minutes. Device shares an identical APK signature with 22 other workers who submitted claims within a 4-minute window. Confidence: coordinated fraud."*

This is a decision support tool for the reviewer — not a detection mechanism. It also creates a documented audit trail critical for regulatory and insurance compliance.

---

## Compute Architecture: Cost-Aware Under Attack

```
Every Claim
    |
    v
[Rule Engine]           <-- microseconds, deterministic, always runs
    |
    v
[LightGBM Scorer]       <-- milliseconds, runs if rules don't resolve
    |
    +-- score < 0.25  --> Auto-Approve
    +-- score 0.25–0.65 -> Soft Challenge
    +-- score > 0.65  --> Graph Analysis  <-- triggered on-demand, once per cluster
                              |
                              +-- CV Layer    (only if evidence submitted)
                              +-- LLM Summary (only if escalated to human review)
```

**Under a 500-claim coordinated attack:**
- ~70% of legitimate claims resolve at Rule Engine + LightGBM — never touch expensive layers
- Graph analysis runs **once per cluster**, not per claim — 500 fraudsters = 1 graph call
- LLM inference only on escalated claims

The system scales sub-linearly with fraud volume. Fraud rings don't multiply the compute cost.

---

## UX: Do Not Penalize Uncertainty — Resolve It

### Tier 1 — Auto-Approve `(fraud_score < 0.25)`
Instant payout within SLA. Zero friction. Covers the clear majority of claims.

### Tier 2 — Soft Challenge `(0.25 ≤ fraud_score < 0.65)`
One-tap re-confirmation sent. If worker is **offline due to network degradation** (legitimate in a storm), claim enters a **2-hour grace queue** — resumes automatically on reconnect. No rejection, no penalty.

### Tier 3 — Human Review `(fraud_score ≥ 0.65)`
Routed to reviewer dashboard with LLM summary pre-loaded. Worker is informed immediately, non-accusatorily. No punitive action until human confirms. Cleared claims leave zero permanent record on worker profile.

### Appeals
Every rejected claim → one-tap appeal, 48-hour human review SLA. False positive rate is a weekly-reviewed first-class KPI.

---

## ⚠️ Failure Mode Handling

This system is designed to **fail safe, not fail open.**

| Failure | Fallback Behavior |
|---|---|
| Graph analysis delayed | Per-claim ML + rules continue independently |
| Sensor data partially missing | LightGBM degrades gracefully on available signals |
| LLM service unavailable | Deterministic fraud scores drive auto-approve/escalate; review queue operates without summaries |
| CV layer unavailable | Evidence held for manual review; claim not blocked |
| Network outage for worker | Grace queue activated; resumes on reconnection |

Uncertainty is held, not rejected. A system that punishes its own failures by blocking legitimate payouts is not production-ready.

---

## 📊 Expected Impact

| Metric | Target |
|---|---|
| Coordinated fraud ring detection | >90% caught before payout stage |
| False payout reduction | 70–85% in high-risk geographies |
| False positive rate | <3% through tiered UX and grace handling |
| Human review load reduction | ~40% via LLM-assisted summaries |

Both financial protection (liquidity safety) and worker trust retention — because a platform that innocent workers stop trusting is already failing.

---

## Tech Stack

- **Frontend:** Vanilla HTML/CSS/JS — zero dependencies, instant load
- **ML Layer:** LightGBM / XGBoost (behavioral coherence scoring)
- **Graph Layer:** Louvain community detection on bipartite worker-claim graph
- **CV Layer:** EXIF analysis + perceptual hashing + scene classification
- **LLM Layer:** Prompted structured reasoning for audit trail generation
- **Rule Engine:** Deterministic velocity, plausibility, and device integrity checks

---

## Why This System Wins

> **We don't try to out-detect fraud — we make it economically and operationally unscalable.**

Fraud rings that want to defeat ClaimShield must simultaneously spoof GPS, fake accelerometer and battery telemetry, suppress home Wi-Fi SSIDs, defeat cell tower triangulation, vary device fingerprints, stagger claim timing to avoid temporal clustering, and avoid graph-level connectivity — all while coordinating fast enough to exploit a weather event window.

The cost and complexity of that attack far exceeds the expected payout. That asymmetry is the defense.

---

*Built for Guidewire DEVTrails 2026 — Phase 2: Scale. Designed as a pluggable defense layer requiring no GPS infrastructure changes.*
