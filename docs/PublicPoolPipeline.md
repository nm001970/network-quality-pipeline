عالی. با توجه به تمام بخش‌هایی که بررسی کردیم، این مستند باید فقط مربوط به **چرخه ساخت Public Pool روی یک دستگاه** باشد و **هیچ اشاره‌ای به معماری دوماشینه، PAN یا Merge نکند**. آن بخش در فصل بعدی مستند می‌آید.

پیشنهاد می‌کنم متن به شکل زیر باشد:

---

# Public Pool Pipeline

## Overview

The Public Pool Pipeline is responsible for transforming validated endpoints into a production-ready public subscription.

Unlike the Connectivity Pipeline and the Quality Pipeline, which only evaluate endpoint behavior, this layer makes the first production-level decisions.

Its responsibility is to determine which validated endpoints are eligible for public delivery, rank them according to operational metrics, sanitize sensitive information, generate the public subscription dataset, and verify that the exported subscription is internally consistent before publication.

This pipeline represents the final processing stage performed by a single execution node before any higher-level synchronization or deployment logic begins.

---

# Pipeline Responsibilities

The pipeline performs the following responsibilities:

* Read validated endpoints from the Pool database
* Apply Pool ownership rules
* Apply Health status filtering
* Calculate delivery priority
* Rank production candidates
* Select Hot endpoints
* Select Reserve endpoints
* Sanitize delivery configuration
* Generate production JSON
* Validate exported subscription
* Produce a publish-ready artifact

No network testing is performed in this stage.

All decisions are based on previously collected runtime measurements.

---

# Pipeline Flow

```text
Validated Pool
        │
        ▼
Pool Candidate Selection
        │
        ▼
Health Filtering
        │
        ▼
Metric Aggregation
        │
        ▼
Delivery Score Calculation
        │
        ▼
Ranking
        │
        ├──────────────► Hot Pool
        │
        └──────────────► Reserve Pool
                     │
                     ▼
Configuration Sanitization
                     │
                     ▼
Public JSON Generation
                     │
                     ▼
Safe Export Validation
                     │
                     ▼
Publish Ready Dataset
```

---

# Pool Candidate Selection

The pipeline never reads directly from the Connectivity or Quality queues.

Instead, it operates exclusively on endpoints that have already entered the Pool layer.

Only endpoints satisfying all Pool requirements are considered.

Typical requirements include:

* Accepted by Pool Gate
* Belonging to the current operator
* Existing inside the Pool database
* Valid Health status
* Valid Quality information

The Pool therefore becomes the authoritative production source.

---

# Health Filtering

Endpoints are filtered according to their operational health.

Only endpoints classified as exportable continue.

Typical exportable states include:

* healthy
* unknown
* suspect

Endpoints marked as unhealthy or rejected never enter the public subscription.

---

# Effective Runtime Metrics

Delivery decisions are not based solely on Quality measurements.

Whenever Pool Health statistics are available, they become the authoritative performance source.

The pipeline builds an effective runtime profile using:

* Pool median download speed
* Pool average download speed
* Quality latency
* Quality upload speed
* Quality score

If Pool Health statistics are unavailable, Quality measurements are used as fallback.

---

# Delivery Score

Each candidate receives a Delivery Score.

The Delivery Score represents production suitability rather than laboratory quality.

Unlike the Quality Score, which measures endpoint capability, the Delivery Score estimates how desirable an endpoint is for public distribution.

The score combines multiple runtime characteristics including:

* Quality Score
* Download throughput
* Upload throughput
* Average latency
* Average ping
* Runtime penalties
* Health-derived metrics

This score is recalculated every export cycle.

---

# Hot Pool Selection

The Hot Pool represents the production subscription.

Selection is not performed by simple sorting.

If Pool slot assignments exist, Hot endpoints are selected according to their predefined slot ordering.

Otherwise, Delivery Score becomes the primary ranking mechanism.

The number of exported Hot endpoints is fixed by configuration.

---

# Reserve Pool Selection

Endpoints that are not selected for the Hot Pool may still enter the Reserve Pool.

Reserve candidates are ranked using Health metrics and Delivery Score.

The Reserve Pool provides production-ready replacements without requiring another Quality evaluation.

---

# Configuration Sanitization

Before publication, every selected endpoint passes through the Delivery Sanitizer.

Sensitive information is removed or normalized while preserving protocol compatibility.

This guarantees that public subscriptions expose only the information required by clients.

---

# Public JSON Generation

After ranking completes, the pipeline generates the public subscription structure.

The generated dataset contains:

* metadata
* generation timestamp
* operator identifier
* Hot endpoints
* Reserve endpoints

The file is first written to a temporary candidate location.

Only validated candidates may replace the active production subscription.

---

# Safe Export Validation

Replacing the production subscription is performed through a protected export mechanism.

The export process validates:

* JSON integrity
* Required metadata
* Minimum Hot endpoint count
* Duplicate endpoint detection
* Blocked marker detection
* Delivery metadata
* Configuration remark consistency
* Protocol metadata
* Structural correctness

Only fully validated exports are accepted.

---

# Atomic Publication

The active subscription file is never modified directly.

Instead:

1. A candidate subscription is generated.
2. The candidate is validated.
3. Validation succeeds.
4. The production file is atomically replaced.
5. A Last Good Backup is updated.

If validation fails, the production subscription remains unchanged.

---

# Failure Recovery

The pipeline always preserves the last verified production dataset.

If a new export fails:

* the invalid candidate is discarded;
* the previous verified dataset remains available;
* optional fallback can restore the previous version automatically.

This guarantees that production subscriptions are never replaced by corrupted or incomplete exports.

---

# Design Goals

The Public Pool Pipeline is designed around several architectural principles:

* Production decisions occur only after Quality validation.
* Pool ownership is the authoritative production source.
* Runtime Health has higher priority than historical Quality when available.
* Publication is deterministic and reproducible.
* Production files are replaced atomically.
* Export validation prevents corrupted subscriptions.
* Recovery is always possible through the last verified dataset.

The pipeline therefore serves as the bridge between endpoint validation and production delivery, ensuring that only stable, verified, and operationally suitable endpoints become part of the public subscription.
