# Safe Public Export Pipeline

## Overview

The Safe Public Export Pipeline is the final protection layer executed before a production subscription becomes available.

Unlike previous pipelines that evaluate, rank or select endpoints, this layer performs no connectivity tests and no quality analysis.

Its responsibility is to guarantee that the generated subscription is structurally valid, internally consistent, and can safely replace the currently published production dataset.

This layer prevents corrupted, incomplete or invalid exports from reaching production.

---

# Responsibilities

The Safe Public Export Pipeline is responsible for:

- Executing the production export process
- Creating an isolated candidate subscription
- Validating the candidate dataset
- Running optional delivery validation
- Atomically replacing the production file
- Preserving the last verified export
- Recovering automatically after failures
- Recording export status

This layer is designed for reliability rather than endpoint evaluation.

---

# Pipeline Flow

```text
Public Pool
      │
      ▼
Backup Current Export
      │
      ▼
Generate Candidate Export
      │
      ▼
Candidate Validation
      │
      ▼
Optional Delivery Validation
      │
      ▼
Atomic Replace
      │
      ▼
Final Validation
      │
      ▼
Update Last Good Backup
      │
      ▼
Publish Ready Subscription
```

---

# Why Safe Export Exists

Replacing the production subscription directly is inherently dangerous.

A partially written file, corrupted JSON, interrupted export process or invalid configuration could immediately affect every client using the subscription.

For this reason the production file is never generated directly.

Instead, every export passes through an isolated validation pipeline before becoming the active subscription.

---

# Candidate Export

The export process first generates a temporary candidate file.

Typical workflow:

1. Execute the Pool export process.
2. Write the generated subscription into a temporary candidate file.
3. Never modify the active production file during generation.

If export generation fails, the production subscription remains untouched.

---

# Candidate Validation

The generated candidate must pass a complete validation process before publication.

Validation includes:

- JSON parsing
- Structural verification
- Required metadata
- Hot section validation
- Reserve section validation
- Protocol verification
- Metadata verification
- Duplicate detection
- Blocked marker detection
- Configuration remark validation

Only a fully valid candidate may continue.

---

# Export Integrity Verification

The validation process verifies that the exported subscription satisfies all production requirements.

Examples include:

- required Hot endpoint count
- valid protocol metadata
- valid server metadata
- valid port metadata
- network type
- transport security
- duplicate configuration detection
- prohibited delivery markers
- subscription remark consistency

This validation is independent from the Pool selection logic.

---

# Delivery Validation

An optional delivery validation stage may be enabled.

This stage performs additional production-oriented checks before the candidate is accepted.

Because it is optional, deployments may enable or disable this verification without modifying the export architecture itself.

---

# Atomic Replacement

The production subscription is never overwritten directly.

Instead the pipeline performs:

1. Copy candidate into a temporary replacement file.
2. Atomically replace the production file.
3. Verify the newly published file.

Atomic replacement guarantees that clients never observe partially written subscriptions.

---

# Last Good Backup

Every successful publication updates the Last Good backup.

The backup always represents the most recently verified production subscription.

It serves as the recovery source whenever future exports fail.

The backup is updated only after successful validation.

---

# Failure Recovery

If candidate generation or validation fails:

- the invalid candidate is discarded;
- the active production subscription remains unchanged;
- the Last Good backup remains available.

If configured, the pipeline may automatically restore the previous verified subscription.

This recovery mechanism guarantees continuous availability even when export generation fails.

---

# Export Status

Every execution generates a status record describing the export result.

Typical information includes:

- execution status
- timestamp
- export paths
- validation result
- candidate statistics
- recovery information
- failure details

The status file provides operational visibility without modifying the production subscription itself.

---

# Failure Isolation

The pipeline isolates failures into separate stages.

Typical failure sources include:

- export generation
- malformed JSON
- validation failure
- delivery validation failure
- replacement failure
- recovery failure

Each stage is handled independently to minimize production impact.

---

# Design Principles

The Safe Public Export Pipeline follows several architectural principles:

- Production files are never written directly.
- Every export is validated before publication.
- Publication is atomic.
- Recovery is deterministic.
- Previously verified exports are preserved.
- Invalid candidates are never published.
- Validation is independent from endpoint selection.

---

# Boundary of this Layer

This pipeline does **not**:

- perform connectivity testing;
- perform quality measurements;
- calculate delivery scores;
- select Pool candidates;
- rank endpoints.

Those responsibilities belong to previous architectural layers.

The Safe Public Export Pipeline begins only after the production dataset has already been generated.

Its responsibility ends when a verified production subscription is safely published and ready for downstream delivery.