HotManagerPipeline.md
# Hot Manager Pipeline

## Purpose

The Hot Manager is responsible for maintaining the highest-quality configurations in the **Hot Pool**.

Unlike the Reserve pipeline, which focuses on admission, the Hot Manager continuously evaluates whether configurations deserve to remain in the highest-priority delivery pool.

Its responsibilities are:

- Promote qualified Reserve configurations into Hot.
- Periodically revalidate existing Hot configurations.
- Detect degradation.
- Demote unstable Hot configurations back to Reserve.
- Maintain the configured Hot pool size.
- Prevent correlated failures from collapsing the Hot pool.

The Hot Manager never bypasses the previous pipeline stages.

Every Hot configuration has already passed:


Connectivity
↓
Quality
↓
Reserve Admission Gate
↓
Reserve Health
↓
Hot Manager


---

# Pipeline Position


Raw
│
▼
Connectivity Test
│
▼
Quality Validation
│
▼
Reserve Admission Gate
│
▼
Reserve Pool
│
▼
Reserve Health
│
▼
Hot Manager
│
▼
Hot Pool
│
▼
Public Export


---

# Primary Responsibilities

The Hot Manager performs five major responsibilities.

---

## 1. Fill Missing Hot Capacity

The Hot pool has a configured target size.

Example:


HOT_TARGET = 5


If only three healthy Hot configurations remain:


Current Hot = 3
Target Hot = 5


The Hot Manager searches Reserve for promotion candidates.

---

## 2. Promote Reserve Configurations

Promotion candidates are selected from Reserve according to quality ranking.

Typical selection order includes:

- highest quality score
- lowest latency
- valid Quality status
- accepted Gate status
- healthy Reserve status

Only candidates satisfying all promotion policies enter the promotion process.

---

## 3. Periodic Hot Validation

Hot configurations are never assumed to remain healthy forever.

Each Hot configuration is periodically revalidated.

The validation process includes:

- multiple real connection attempts
- multiple download measurements
- exit IP verification
- stability verification

Example:


Round 1
Round 2
Round 3


Promotion only succeeds if enough rounds succeed.

Example:


Rounds = 3
Minimum Successful = 2


---

## 4. Automatic Demotion

If a Hot configuration repeatedly fails validation it is removed from Hot.

Typical causes:

- download degradation
- unstable connectivity
- invalid exit IP
- repeated failures
- structural failures

The configuration returns to Reserve or Quarantine depending on failure severity.

---

## 5. Protect Hot Capacity

One important design goal is preventing catastrophic Hot collapse.

Imagine:


5 Hot configs


Network outage occurs.

All five fail simultaneously.

Without protection:


Hot → Empty


Public delivery immediately loses every high-priority configuration.

Instead, the Hot Manager evaluates correlated failures.

Only configurations that exceed safety thresholds are demoted.

This prevents large-scale false positives.

---

# Promotion Pipeline

Promotion follows a strict sequence.


Reserve Candidate
│
▼
Claim Promotion
│
▼
Long Validation
│
▼
Enough Successful Rounds?
│
┌────┴────┐
│ │
Yes No
│ │
▼ ▼
Promote Failure
│ │
▼ ▼
Hot Retry Later


---

# Long Validation

Unlike Reserve admission, Hot promotion uses repeated validation.

Example:


Round 1
Download = 8.4 Mbps

Round 2
Download = 7.9 Mbps

Round 3
Download = 8.1 Mbps


Average performance is evaluated.

Only sufficiently stable configurations are promoted.

---

# Failure Policy

Failures are divided into two categories.

## Temporary Failure

Examples:

- temporary packet loss
- unstable network
- slow download

Result:


Retry later


---

## Structural Failure

Examples:

- invalid configuration
- dead server
- wrong exit IP
- protocol failure

Result:


Immediate demotion


---

# Failure Streak

Each Hot configuration tracks consecutive failures.

Example:


Failure #1

↓

Failure #2

↓

Failure Limit Reached

↓

Demotion


The limit is configurable.

Example:


FAILURE_LIMIT = 2


---

# Hot Health States

Typical Hot states include:


Healthy


Configuration is suitable for public delivery.

---


Checking


Currently being validated.

---


Failed


Validation failed.

---


Demoted


Removed from Hot.

---

# Promotion Retry Age

Configurations that fail promotion are not immediately retried.

A cooldown prevents endless promotion loops.

Example:


PROMOTION_RETRY_MIN_AGE_MINUTES = 120


---

# Correlated Failure Protection

The Hot Manager evaluates overall Hot pool health.

Example:


Hot Pool = 5

Failures = 4


Demoting every configuration simultaneously could unnecessarily destroy the Hot pool.

Instead, configurable thresholds determine whether failures are likely correlated.

Example parameters:


MIN_HOT_CAPACITY

CORRELATED_FAILURE_MIN_COUNT

CORRELATED_FAILURE_RATIO


These prevent cascading demotions caused by temporary infrastructure problems.

---

# Lock Protection

Only one Hot Manager instance may operate on an operator at a time.

Operator lock:


hot_manager_irancell.lock


This prevents:

- duplicate promotions
- simultaneous demotions
- race conditions
- conflicting database updates

---

# Startup Recovery

If a previous Hot Manager crashes while processing:


health_status = checking


or


promotion_status = checking


those stale claims are automatically recovered during startup.

Recovery parameters:


STUCK_MINUTES

RECOVER_LIMIT


---

# Batch Execution

The Hot Manager executes in batches.

Typical configuration:


HOT_TARGET = 5

MAX_CANDIDATES = 10


Every cycle:

1. Recover stale claims.
2. Inspect current Hot pool.
3. Fill missing capacity.
4. Validate existing Hot configurations.
5. Promote new candidates.
6. Demote failed configurations.
7. Write logs.

---

# Operator Isolation

Each operator is processed independently.

Example:


Irancell Hot


and


Hamrah Aval Hot


never share:

- locks
- candidate queues
- Hot pools
- validation state

---

# Design Principles

The Hot Manager is designed around several principles:

- Strict promotion requirements.
- Continuous validation.
- Automatic recovery.
- Safe demotion.
- Correlated failure protection.
- Stable Hot capacity.
- Independent operator isolation.
- Single-instance execution.
- Complete auditability through logging.
- No configuration bypasses earlier pipeline stages.

The result is a Hot pool that prioritizes long-term stability over short-term performan