⬡ DataGuard Cloud — Redundancy Removal System

An interactive, browser-based simulation of a cloud data validation pipeline. DataGuard Cloud models how a real ingestion system would classify incoming infrastructure records as unique, a duplicate, or a false positive before allowing them into a "cloud database" — using hash fingerprinting and weighted field-similarity scoring, with an AI layer that explains each decision in plain language.

📊 Overview

The app simulates a stream of cloud compute records (EC2, Lambda, RDS, S3, ECS, EKS, CloudFront, DynamoDB instances) being submitted for storage. Every new record is run through a two-stage detection pipeline against the existing dataset:

1. Hash fingerprint check — instant exact-duplicate detection
2. Weighted similarity scoring — field-by-field comparison to catch near-duplicates and distinguish them from coincidentally similar but genuinely distinct records (false positives)

Only records classified as unique are appended to the database. Everything else is blocked or flagged, and every decision is logged for audit.

✨ Features

 ◈ Submit Panel

- Form for instance ID, CPU %, memory %, storage (GB), region, service, and status
- Quick-fill tools for testing the engine instantly:
  - `Random` — generates a fresh, unique-looking record
  - `Clone Exact` — copies an existing record verbatim (forces an exact duplicate)
  - `Near Clone` — copies a record with small CPU/memory drift (forces a near-duplicate)
- Live hash preview that updates as you type

🔍 Validation Pipeline

- Animated "Scanning Database" state with four simulated checks: hash fingerprint check, exact match detection, similarity scoring, near-duplicate analysis
- Real-time classification result with:
  - Outcome badge (Exact Duplicate / Near Duplicate / False Positive / Unique) and similarity %
  - Submitted record detail card (instance, service, region, status, CPU/memory/storage, hash)
  - Field-by-field weight & match breakdown (which fields matched the closest existing record, and how much each one counted toward the score)
  - Side-by-side comparison with the matched record, when one exists
  - AI Analysis — a short natural-language explanation of the classification and a recommended action, generated via the Claude API

 ⊞ Database Panel

- Live, searchable view of every stored record (search by instance ID, service, or region)
- Per-record delete
- Purge Duplicates — bulk hash-based deduplication pass across the whole database, with an AI-generated summary of the cleanup impact

 ≡ Audit Log

- Chronological log of every validation event (instance, service, region, classification, similarity score, timestamp)
- Clearable on demand

📈 Analytics Dashboard

- Average CPU and memory utilization across stored records
- Breakdown by service and by instance status
- Region count
- Validation outcomes chart (Unique vs. Blocked vs. Flagged), once activity exists
- Live header counters: Total Records, Appended, Blocked, Flagged

🧠 Classification Engine

Hash Fingerprinting (Exact-Duplicate Detection)

Each record is reduced to a hash over its core fields:

```
instanceId | region | service | cpuUsage | memoryUsage | storageGB | status
```

If an incoming record's hash matches an existing record's hash exactly, it's classified Exact Duplicate with 100% confidence and blocked immediately — no further scoring needed.

Weighted Field Similarity (Near-Duplicate / False-Positive Detection)

When no exact hash match exists, the record is compared against every existing record using a weighted similarity score:

| Field | Weight | Match Rule |
|---|---|---|
| Instance ID | 35% | Exact match |
| Region | 15% | Exact match |
| Service | 15% | Exact match |
| CPU Usage | 10% | Within 5% of existing value |
| Memory Usage | 10% | Within 5% of existing value |
| Storage (GB) | 10% | Exact match |
| Status | 5% | Exact match |

The highest-scoring existing record becomes the "best match," and its score determines the outcome:

| Similarity Score | Classification | Action |
|---|---|---|
| 100% (hash match) | Exact Duplicate | 🚫 Blocked |
| ≥ 75% | Near Duplicate | 🚫 Blocked |
| 45% – 74% | False Positive | ⚠️ Flagged for manual review (not inserted) |
| < 45% | Unique | ✅ Appended to database |

🏆 Outcomes Tracked

- Appended — verified unique records written to the database
- Blocked — exact or near-duplicates rejected before insertion
- Flagged — false positives held for review instead of being silently discarded or wrongly inserted

🛠️ Technologies Used

Frontend
- HTML5
- CSS3 (custom dark/terminal-style UI, no framework)
- Vanilla JavaScript (no build step, no dependencies)
- Google Fonts — Share Tech Mono, Rajdhani

AI Layer
- Anthropic Claude API (`claude-sonnet-4-20250514`) — generates the plain-language "AI Analysis" explanation for each classification result and the summary after a bulk purge

🏗️ Architecture Notes

This is a single-file client-side prototype: `data-redundancy-system-final.html` contains all markup, styling, and logic. The "cloud database" is an in-memory JavaScript array seeded with five sample records — it resets on page reload and isn't backed by a real database or server. It's built to demonstrate and let you experiment with the detection logic and UX of a redundancy-removal pipeline, not to serve as a production backend.

 🚀 Getting Started

No installation or build step required — it's a single static HTML file.

```bash
# Just open it in a browser
open https://dataredundancysystem.edgeone.app/
```

> The AI Analysis feature calls the Anthropic API directly from the browser. If that endpoint isn't reachable in your environment, the UI degrades gracefully and shows "AI analysis unavailable" while the rest of the validation pipeline keeps working normally.

🔍 Key Insights

Exact and Near Duplicates Aren't the Same Risk
Hash matching catches identical records instantly, while weighted similarity scoring is needed to catch records that are almost the same but not byte-for-byte equal.

False Positives Deserve a Third Outcome, Not Just Yes/No
Treating "somewhat similar" records as an automatic block risks discarding legitimate data. Routing them to a flagged/review state instead of a binary accept-or-reject avoids that failure mode.

Field Weighting Reflects Real-World Identity
Not all fields are equally diagnostic of duplication — Instance ID carries far more weight (35%) than status (5%), mirroring how a real system would prioritize identifying fields over volatile metrics.

Explainability Builds Trust in Automated Rejection
Surfacing why a record was blocked or flagged (field-by-field match breakdown plus an AI-generated explanation) makes the system's decisions auditable instead of opaque.

🎯 Learning Objectives

This project demonstrates:

- Hash-based exact-duplicate detection
- Weighted multi-field similarity scoring
- Redundant vs. false-positive classification logic
- Append-only, validation-gated data ingestion
- Audit logging and analytics for data quality monitoring
- Using an LLM to generate human-readable explanations of algorithmic decisions

🔮 Future Improvements

- Persist the database to a real cloud backend (e.g., Firebase, DynamoDB, Postgres) instead of in-memory state
- Configurable similarity thresholds and field weights via the UI
- Fuzzy/semantic text similarity for free-text or descriptive fields
- Batch import and bulk validation of CSV/JSON record sets
- Exportable audit log (CSV/JSON)
- Authentication and multi-user record ownership
- Server-side validation to prevent client-side bypass of the duplicate checks

 ⭐ Feedback

If you found this project useful, consider giving the repository a star and sharing your feedback!
