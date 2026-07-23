# Raw To Reserve Pipeline

## Purpose

The Raw → Reserve pipeline is responsible for continuously producing validated Reserve endpoints.

It is the entry point of the Version 2 architecture.

No endpoint can enter the Reserve pool without successfully passing this pipeline.

---

# Responsibilities

The pipeline is responsible for:

- Selecting new endpoint candidates
- Running connectivity validation
- Running quality validation
- Running Reserve admission validation
- Creating Reserve entries
- Rejecting invalid endpoints
- Recovering interrupted claims

The pipeline never performs:

- Reserve maintenance
- Hot promotion
- Publishing

These responsibilities belong to independent workers.

---

# High-Level Flow

```
Raw Config

↓

Smart Selection

↓

Connectivity

↓

Quality

↓

Reserve Admission Gate

↓

Reserve Pool
```

---

# Processing Model

Unlike Version 1, processing is continuous.

The worker never waits for a full platform cycle.

Instead, it continuously executes small batches.

```
while True

↓

Select

↓

Validate

↓

Reserve

↓

Repeat
```

This significantly reduces latency between endpoint discovery and Reserve availability.

---

# Smart Selection

Candidate selection is performed before any validation begins.

The planner builds a balanced processing batch using multiple categories.

Typical categories include:

- New endpoints
- Previously healthy endpoints
- Retry candidates

Selection ratios are configurable.

The planner only selects candidates.

It never modifies database state.

---

# Claim Lifecycle

Processing ownership is established immediately before an endpoint enters the pipeline.

Each endpoint is claimed individually.

```
Select

↓

Claim

↓

Process

↓

Complete
```

This prevents duplicate processing by concurrent workers.

If processing cannot continue, the claim is released safely.

---

# Stage 1 — Connectivity Validation

Purpose:

Verify that the endpoint is reachable.

Checks include:

- Connection establishment
- Basic network availability
- Exit IP detection
- Latency measurement

Endpoints failing connectivity are rejected immediately.

No additional resources are consumed.

---

# Stage 2 — Quality Validation

Only endpoints with successful connectivity continue.

Quality validation evaluates the endpoint performance.

Typical measurements include:

- Ping
- Download speed
- Upload speed (when applicable)

The endpoint must satisfy Quality policy before continuing.

Quality failures terminate the pipeline.

---

# Stage 3 — Reserve Admission

Qualified endpoints are evaluated for Reserve admission.

The Admission Gate performs an independent validation before creating a Reserve entry.

Successful admission creates an accepted Reserve endpoint.

Rejected endpoints never enter the Reserve pool.

---

# Reserve Creation

Only the Admission stage is allowed to create Reserve entries.

This provides a single write path into the Reserve pool.

No other worker creates Reserve endpoints.

---

# Failure Strategy

The pipeline follows a Fail-Fast design.

```
Connectivity

↓

Fail

↓

Stop
```

```
Quality

↓

Fail

↓

Stop
```

```
Gate

↓

Fail

↓

Stop
```

Later stages are never executed after a failed validation.

---

# Result Codes

The pipeline returns standardized processing results.

Examples include:

- reserve_added
- connectivity_rejected
- quality_ping_rejected
- quality_download_rejected
- quality_upload_rejected
- quality_rejected
- gate_rejected
- config_not_found
- quality_error

Workers use these result codes for logging, monitoring and statistics.

---

# Exception Handling

Unexpected infrastructure failures are not hidden.

Exceptions propagate to the pipeline controller.

The controller is responsible for:

- releasing unfinished claims
- recording failures
- restarting processing when appropriate

This keeps the admission pipeline deterministic.

---

# Processing Guarantees

The pipeline guarantees:

- Every endpoint follows the same validation sequence.
- Validation order is never skipped.
- Reserve receives only fully validated endpoints.
- Each endpoint is processed by at most one worker at a time.
- Interrupted processing can be recovered safely.

---

# Worker Boundaries

This pipeline owns only endpoint production.

It does not maintain Reserve quality.

It does not maintain Hot capacity.

It does not publish configurations.

Those responsibilities belong to independent workers described in:

- ReserveHealthPipeline.md
- HotManagerPipeline.md
- PublishPipeline.md