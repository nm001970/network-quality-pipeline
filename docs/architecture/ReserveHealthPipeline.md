# Reserve Health Pipeline

## Purpose

The Reserve Health pipeline continuously maintains the integrity of the Reserve pool.

Unlike Version 1, Reserve is no longer considered a static storage layer.

Reserve is a living endpoint buffer that is continuously monitored, repaired and cleaned.

---

# Responsibilities

The Reserve Health worker is responsible for:

- Monitoring Reserve endpoint health
- Detecting degraded endpoints
- Revalidating accepted endpoints
- Maintaining Reserve quality
- Recovering interrupted Health claims
- Quarantining unhealthy endpoints

The worker never performs:

- Raw production
- Hot promotion
- Public publishing

---

# High-Level Flow

```
Reserve Pool

↓

Health Selection

↓

Health Claim

↓

Long Validation

↓

Healthy

or

Suspect

or

Quarantined
```

---

# Reserve Philosophy

Reserve is the platform's operational buffer.

It exists to ensure that Hot promotion always has access to validated endpoints.

Reserve should always contain endpoints that are expected to become Hot candidates.

---

# Health Scheduling

Reserve endpoints are not tested continuously.

Each endpoint is revalidated only after reaching its scheduled health interval.

Different health states use different refresh intervals.

Examples include:

- Healthy endpoints
- Suspect endpoints

This minimizes unnecessary testing while ensuring continuous monitoring.

---

# Health Claim

Before an endpoint is tested, ownership is established.

```
Reserve

↓

Claim

↓

Health Validation

↓

Complete
```

Only one worker may evaluate a Reserve endpoint at a time.

Interrupted claims are automatically recoverable.

---

# Health Validation

The Health worker performs operational validation on accepted Reserve endpoints.

Validation focuses on maintaining production readiness rather than admission.

Typical checks include:

- Connectivity
- Download performance
- Long validation stability

---

# Health States

Reserve endpoints may exist in several operational states.

```
Unknown

↓

Healthy

↓

Suspect

↓

Healthy
```

or

```
Healthy

↓

Suspect

↓

Quarantined
```

Health state changes occur automatically based on validation results.

---

# Healthy

Healthy endpoints remain inside the Reserve pool.

They continue participating in future promotion decisions.

Healthy endpoints are rechecked only after their health interval expires.

---

# Suspect

A suspect endpoint is temporarily retained inside Reserve.

It has shown degraded behaviour but has not yet exceeded the failure policy.

Suspect endpoints are revalidated sooner than healthy endpoints.

This prevents unnecessary removal caused by temporary network instability.

---

# Quarantined

Endpoints exceeding the configured failure policy are removed from operational use.

Quarantined endpoints are no longer eligible for Hot promotion.

They remain outside the active Reserve pool until later processing determines otherwise.

---

# Failure Strategy

Reserve Health does not immediately remove an endpoint after a single failure.

Instead, failures accumulate.

```
Failure

↓

Failure Counter

↓

Policy Evaluation

↓

Keep

or

Quarantine
```

This protects the platform from temporary network instability.

---

# Structural Failures

Certain failures indicate that an endpoint is fundamentally invalid.

Examples include:

- Connectivity state mismatch
- Quality state mismatch
- Missing endpoint records

These failures immediately invalidate operational readiness.

---

# Recovery

At startup, the worker automatically recovers interrupted Health claims.

Recovered endpoints return to an eligible state and may be processed again.

This prevents permanent claim ownership after unexpected process termination.

---

# Reserve Integrity

Reserve Health continuously guarantees that Reserve contains only operationally valid endpoints.

As a result:

- Hot promotion always reads from a maintained Reserve.
- Invalid endpoints are removed before reaching Hot.
- Reserve quality remains stable over long-running execution.

---

# Continuous Processing

Reserve Health operates continuously.

```
Select

↓

Validate

↓

Update Health State

↓

Repeat
```

The worker never waits for a complete platform cycle.

---

# Worker Boundaries

Reserve Health owns only Reserve maintenance.

It does not create Reserve entries.

It does not perform Hot promotion.

It does not publish configurations.

These responsibilities belong to:

- RawToReservePipeline.md
- HotManagerPipeline.md
- PublishPipeline.md