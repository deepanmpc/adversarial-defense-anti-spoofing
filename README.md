# Adversarial Defense & Anti-Spoofing Strategy

> **Guidewire DEVTrails 2026 — Phase 1 Submission**
> Parametric Insurance Platform for Gig & Delivery Workers

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

That shift — from threshold-based verification to behavioral coherence scoring — is what separates this architecture from standard approaches.

Key distinctions:

- **Proactive, not reactive.** Fraud rings are detected at the network level before individual claims are fully processed.
- **Multi-modal, not single-signal.** No single data point makes or breaks a decision. Spoofing GPS is easy. Simultaneously spoofing GPS, accelerometer patterns, network signal quality, Wi-Fi BSSID, battery telemetry, and historical route data is not.
- **Designed for coordinated attacks.** Individual anomaly detection is table stakes. The graph intelligence layer specifically targets synchronized, ring-level fraud behavior.
- **Cost-aware by design.** Heavy compute (LLM reasoning, graph analytics) is only triggered when lighter layers cannot resolve the claim. The majority of legitimate claims never touch those layers.

---

## Layer 1: Rule Engine — Zero-Latency Hard Checks

The first line of defense is deterministic and runs in microseconds. No model inference. No latency.

Rules that fire immediately and escalate or block the claim:

| Rule | Condition | Action |
|---|---|---|
| Velocity violation | Last verified location > 30 km away < 10 mins ago | Instant escalation |
| Pre-alert volume spike | Claim surge in a region before official weather alert | Ring flag triggered |
| Network contradiction | Claimed GPS in storm zone + strong stable connection | Soft challenge |
| Temporal clustering | 5+ claims from same geofence within 5-minute window | Graph analysis triggered |
| Device integrity | Root detection positive or known spoofing APK signature | Immediate escalation |

Rules are cheap. They catch the obvious cases and gate the more expensive downstream layers.

---

## Layer 2: Sensor Fusion — Behavioral Coherence Scoring

**Real distress is noisy. Fraud is suspiciously clean.**

A genuine stranded delivery worker in a severe weather event produces chaotic, inconsistent sensor data — erratic movement, degraded connectivity, elevated battery drain, a device fighting to maintain GPS lock. A fraudster sitting comfortably at home produces smooth, stable, low-effort telemetry.

We fuse the following signals into a single behavioral coherence score:

| Signal | Genuine Pattern | Fraud Pattern |
|---|---|---|
| Accelerometer / gyroscope | Irregular motion consistent with weather/environment | Near-stationary or artificially smooth |
| Battery drain rate | Elevated — cold environment, screen-on, active GPS | Normal or on charge |
| RF signal quality | Degraded — consistent with storm interference | Strong, stable home network |
| Cell tower triangulation (MLAT) | Corroborates GPS within acceptable drift | Contradicts spoofed coordinate |
| Wi-Fi BSSID geolocation | No home SSID visible, or SSIDs consistent with field location | Home router SSID detected |
| Historical delivery corridors | Worker has documented prior routes in this zone | No historical presence in area |
| App interaction cadence | Distress-consistent: frequent taps, help-seeking behavior | Scripted, low-frequency, or absent |

A **gradient-boosted classifier** (LightGBM) trained on historical claim data produces a `fraud_probability` score from this feature vector. The model learns not just individual signal anomalies but *cross-signal inconsistencies* — the combination of a perfectly stable accelerometer and a claimed storm-zone GPS is far more telling than either signal alone.

---

## Layer 3: Graph Intelligence — Detecting the Ring, Not Just the Individual

**This is the layer that catches coordinated fraud. It is the most important differentiator.**

Individual claim analysis misses organized fraud rings entirely. The graph layer operates above the individual claim — it models the entire claims network in real time.

### Graph Structure

```
Workers --> Claims --> Devices --> Locations --> Weather Events
```

Every entity and every relationship is a node and edge. Fraud rings create structural signatures in this graph that are invisible at the individual claim level but unmistakable at the network level:

- **Device clusters:** Multiple worker accounts linked to identical `(OS build + app install hash)` signatures — the same spoofing toolkit, deployed in bulk.
- **Temporal co-occurrence:** Claims that arrive in tight synchronized windows, inconsistent with the natural randomness of independent distress events.
- **Pre-alert coordination signal:** A surge in claims from a region *before* the official weather escalation threshold is crossed. Genuine workers respond to real conditions. Fraud rings respond to Telegram alerts.
- **Shared infrastructure fingerprints:** Common IP ranges, device IDs recycled across accounts, or BSSID signatures appearing across supposedly independent workers in separate locations.

**Louvain community detection** runs on this graph in near real-time, surfacing dense subgraphs — clusters of workers, devices, and claims that are too tightly interconnected to be coincidental. When a cluster crosses a configurable density threshold, every claim within it is held and escalated together.

This is what makes the system resilient to mass attacks. A ring of 500 workers doesn't get processed as 500 independent claims. It gets detected as one coordinated event and neutralized before the liquidity pool is touched.

---

## Layer 4: Computer Vision — Evidence Integrity Check

When workers submit photographic evidence, the vision layer runs three checks:

1. **EXIF consistency:** Does the embedded GPS metadata match the claimed location? Does the timestamp align with the claim window?
2. **Manipulation detection:** Perceptual hashing and compression artifact analysis to surface edited or recycled images.
3. **Environmental plausibility:** Scene classification to verify that the visual environment (road conditions, sky, surroundings) is consistent with the claimed weather event — not a living room or a clear sunny day.

This layer is a secondary corroboration signal, not a primary decision-maker. It adds weight to the coherence score and provides concrete evidence for human reviewers.

---

## Layer 5: LLM Reasoning — Auditability, Not Detection

The LLM layer has one job: **make fraud decisions explainable to humans.**

It does not detect fraud. The upstream layers do that. What it does is take the structured output from all four preceding layers — the rule flags, the coherence score, the graph cluster membership, the vision findings — and synthesize them into a concise, plain-language summary that a human reviewer can read and act on in under 30 seconds.

Example output:

> *"Claim flagged. Worker's GPS places them in the storm zone, but cell tower triangulation shows a location 12 km away in a residential area. Accelerometer data shows no movement for 40 minutes. Device is part of a cluster of 23 workers with identical app install signatures who submitted claims within a 4-minute window. Confidence: coordinated fraud."*

This is not a summary for the worker. It is a decision support tool for the reviewer. It reduces review time, improves consistency across reviewers, and creates a documented audit trail for every flagged claim — critical for regulatory and insurance compliance.

---

## Compute Architecture: Efficiency Under Load

The system is explicitly tiered to minimize cost and maximize throughput. Under a mass attack, the platform does not grind to a halt running LLM inference on every claim.

```
Every Claim
    |
    v
[Rule Engine]  <-- microseconds, deterministic
    |
    +-- Clean? --> Coherence Score (LightGBM)  <-- milliseconds
                       |
                       +-- Low risk? --> Auto-Approve
                       |
                       +-- Medium risk? --> Soft Challenge
                       |
                       +-- High risk? --> Graph Analysis  <-- triggered on-demand
                                              |
                                              +-- CV Layer (if evidence submitted)
                                              |
                                              +-- LLM Summary for Human Review
```

**Cost distribution under a 500-claim coordinated attack:**
- ~70% of legitimate claims never exit the Rule Engine + LightGBM layers
- Graph analysis is triggered once per detected cluster, not per claim
- LLM inference runs only on claims escalated to human review

This means the system scales linearly with legitimate claim volume and sub-linearly with fraud volume — fraud rings trigger shared compute, not per-fraudster compute.

---

## UX: Do Not Penalize Uncertainty — Resolve It

The detection system is only as good as its treatment of the people it serves. A genuine worker in a real emergency who gets wrongly flagged — and then gets rejected without recourse — is not a UX problem. It is a trust-destroying, platform-killing failure.

The tiered response model ensures that friction is proportional to actual, evidence-backed risk:

### Tier 1 — Auto-Approve `(fraud_score < 0.25)`
Claim processes instantly. Payout within SLA. No friction, no challenge. This covers the clear majority of claims.

### Tier 2 — Soft Challenge `(0.25 ≤ fraud_score < 0.65)`
A single one-tap re-confirmation is sent. The message is transparent and non-accusatory:

> *"We're seeing unusual activity in your area. Please confirm you're currently at [location] to process your claim."*

**Critical edge case — offline worker:** If the worker cannot respond due to network degradation (a completely legitimate scenario in a severe weather event), the claim enters a **2-hour grace queue** and resumes automatically when connectivity returns. The system assumes good faith by default during network outages. No rejection, no penalty.

### Tier 3 — Human Review `(fraud_score ≥ 0.65)`
Claim is routed to the reviewer dashboard with the LLM-generated summary pre-loaded. The worker is informed immediately:

> *"Your claim is under a brief manual review due to unusual activity in your area. This typically resolves within [X] hours. You will be notified as soon as it is processed."*

No punitive action occurs until a human confirms fraud. A flagged-and-cleared claim leaves **zero permanent record** on the worker's trust profile.

### Appeals
Every rejected claim generates a one-tap appeal. Human review SLA: 48 hours. False positive rate is a first-class KPI, reviewed weekly. Any reviewer who clears a flagged claim can annotate the reason — feeding back into model retraining.

---

## Why This System Wins

| Capability | Standard Platform | This System |
|---|---|---|
| GPS spoofing detection | GPS threshold check | Multi-signal behavioral coherence |
| Individual fraud | Rule-based flags | ML anomaly detection across 7+ signals |
| Coordinated ring detection | None | Real-time graph clustering |
| Explainability | Black box score | LLM-generated plain-language audit trail |
| False positive protection | None / manual | Tiered response + grace queue + appeals |
| Compute efficiency | Uniform cost per claim | Tiered cost — heavy layers on-demand only |
| Proactive detection | Reactive (post-payout) | Pre-payout, ring-level cluster detection |

**Practical impact:**
- A 500-person fraud ring is detected as a single coordinated event, not 500 individual claims
- Liquidity drain is stopped before payouts are processed, not after
- Genuine workers in genuine distress are protected, not punished for network failures
- Every decision is auditable — critical for insurance regulation compliance
- The system degrades gracefully: if graph analysis is unavailable, ML + rules still function independently

**Robustness under adversarial adaptation:**
Fraud rings that attempt to defeat this system must simultaneously spoof GPS, fake sensor telemetry, vary device fingerprints, stagger claim timing, and avoid behavioral correlation — while still coordinating fast enough to exploit a weather event window. The cost and complexity of that attack far exceeds the expected payout. That asymmetry is the defense.

---

