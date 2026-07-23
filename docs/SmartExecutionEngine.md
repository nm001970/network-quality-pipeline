Smart Execution Engine
Overview

The Smart Execution Engine is the orchestration layer responsible for controlling the entire production validation cycle.

Unlike the lower-level pipelines that execute individual tests, this layer does not measure endpoint quality itself.

Instead, it decides:

which endpoints should be tested,
how system resources are distributed,
when each pipeline should execute,
how workloads are balanced across the database,
and how validated endpoints move toward production.

The Smart Execution Engine is therefore the decision-making layer that coordinates all lower-level validation components.

Why Smart Execution Exists

Running Connectivity and Quality tests over every stored endpoint during every cycle is computationally expensive and does not scale as the database grows.

Simply testing the newest endpoints continuously would also create a different problem:

old endpoints would never be revalidated,
expired endpoints would remain inside the active pool,
database size would increase indefinitely,
production quality would gradually decrease.

To solve this problem, the platform introduces a Smart Execution strategy.

Instead of testing endpoints sequentially or purely by insertion order, every execution cycle distributes available testing capacity across multiple endpoint categories.

This guarantees that:

new endpoints continuously enter the system,
existing healthy endpoints are periodically revalidated,
previously failed endpoints may recover,
permanently unstable endpoints are gradually removed from production eligibility.

The objective is not maximizing the number of tests.

The objective is maximizing long-term production quality while keeping computational cost predictable.

Execution Flow
Update Configs
        │
        ▼
Smart Batch Planning
        │
        ▼
Claim Selected Configurations
        │
        ▼
Smart Connectivity Pipeline
        │
        ▼
Smart Quality Pipeline
        │
        ▼
Public Export
        │
        ▼
Export Validation
        │
        ▼
Optional Audit

---

# Smart Execution Cycle

The Smart Execution Engine is implemented through the `run_smart_mode()` orchestration routine.

Unlike lower-level components that perform individual tasks, this routine does not execute network validation itself.

Its responsibility is coordinating every stage of the production validation cycle in the correct order.

Each stage depends on the successful completion of the previous stage.

The execution order is intentionally fixed to guarantee database consistency throughout the cycle.

---

# Execution Sequence

The Smart Execution cycle performs the following stages:

1. Update endpoint database
2. Build a smart execution plan
3. Claim selected endpoints
4. Execute Connectivity validation
5. Execute Quality validation
6. Export production endpoints
7. Validate exported subscriptions
8. Execute optional audit tasks

Each stage has a single responsibility and never overlaps with another stage.

---

# Stage 1 — Update Configurations

```text
GitHub Sources
        │
        ▼
Collector
        │
        ▼
Sanitizer
        │
        ▼
Database
```

The execution cycle always begins by synchronizing the database with external configuration sources.

This stage imports newly discovered endpoints while preventing duplicate entries through configuration hashing.

No connectivity or quality validation is performed during this phase.

Its only responsibility is maintaining an up-to-date endpoint repository.

---

# Stage 2 — Smart Batch Planning

After the database has been updated, the engine determines which endpoints should be tested during the current execution cycle.

This decision is performed by the Smart Batch Selector.

Unlike a simple FIFO scheduler, the selector distributes the available testing capacity across multiple endpoint categories.

The objective is maintaining long-term database health rather than continuously testing only recently discovered endpoints.

At this stage no network activity occurs.

Only a runtime execution plan is generated.

---

# Stage 3 — Claim Selected Endpoints

Once the execution plan has been generated, the selected endpoints are atomically claimed inside the database.

Claiming prevents multiple workers from processing the same endpoint simultaneously.

Only successfully claimed endpoints become part of the current execution cycle.

The generated claim list becomes the input for all subsequent validation stages.

---

# Smart Batch Selector

The Smart Batch Selector is responsible for deciding **which endpoints enter the current execution cycle**.

It does not perform any network validation.

It does not measure endpoint quality.

It does not execute Connectivity or Quality workers.

Its responsibility is limited to constructing an execution plan that maximizes resource utilization while maintaining long-term database health.

After the execution plan is generated, the selected endpoints are atomically claimed and passed to the validation pipelines.

---

# Why Batch Selection Is Necessary

As the endpoint database grows, executing validation against every stored configuration during every cycle becomes increasingly inefficient.

A naïve strategy such as continuously selecting only the newest endpoints would eventually produce several problems:

- recently imported endpoints would dominate testing resources;
- previously healthy endpoints would never be revalidated;
- temporarily failed endpoints would never be given another opportunity;
- permanently unstable endpoints would remain inside the active database indefinitely;
- computational cost would increase without improving production quality.

The Smart Batch Selector was designed to solve these problems by distributing the available testing capacity across different categories of endpoints.

The objective is not maximizing the number of tested configurations.

The objective is maintaining a continuously healthy production dataset.

---

# Batch Allocation Strategy

Instead of treating every endpoint equally, the selector divides each execution cycle into several independent allocation buckets.

Each bucket serves a different operational purpose.

The allocation ratios are configurable, allowing the execution policy to evolve without changing the pipeline architecture.

At the beginning of every execution cycle, the selector calculates how many endpoints should be assigned to each bucket before any database rows are selected.

Only after this allocation step are actual database candidates retrieved.

---

# Selection Principles

The Smart Batch Selector follows several design principles:

- avoid repeatedly testing only newly imported endpoints;
- periodically revalidate previously healthy endpoints;
- allow previously failed endpoints to recover;
- prevent duplicate selection inside the same execution cycle;
- keep execution cost predictable as the database grows;
- maintain a balanced distribution between discovery and maintenance.

These principles ensure that the execution cycle continuously refreshes production quality instead of simply expanding the database.

---

# Allocation Buckets

The Smart Batch Selector distributes every execution cycle across four independent allocation buckets.

Each bucket has a different responsibility within the lifecycle of the endpoint database.

The buckets are executed sequentially.

Previously selected endpoints are excluded from subsequent buckets, ensuring that each endpoint appears only once within the current execution cycle.

---

## 1. New Endpoint Bucket

Purpose:

Introduce recently imported endpoints into the validation pipeline as quickly as possible.

This bucket primarily selects configurations that have never completed a Connectivity validation.

Its objective is maintaining continuous discovery of newly collected endpoints.

---

## 2. Alive Revalidation Bucket

Purpose:

Continuously verify endpoints that are currently considered healthy.

Production endpoints are not assumed to remain healthy forever.

Periodic revalidation detects configuration expiration, routing changes, server failures and other runtime degradations before they affect exported subscriptions.

This bucket prevents old production endpoints from remaining trusted indefinitely.

---

## 3. Retry Bucket

Purpose:

Reevaluate endpoints that previously failed Connectivity validation.

Many failures are temporary.

Examples include:

- temporary server outages
- transient routing problems
- provider instability
- temporary network failures

Instead of permanently discarding these endpoints, the Retry Bucket periodically gives them another opportunity to re-enter the validation pipeline.

---

## 4. Fill Bucket

The previous buckets may not always consume the entire execution capacity.

Instead of leaving workers idle, the remaining capacity is filled automatically.

The Fill Bucket performs a final database scan using the same Connectivity eligibility rules while excluding every endpoint already selected during the current execution cycle.

Its purpose is maximizing resource utilization.

The Fill Bucket is not limited to newly imported endpoints.

Previously stored endpoints may also be selected whenever they satisfy the selection rules and have not already been claimed during the current cycle.

This guarantees that available processing capacity is never wasted.

---

# Smart Connectivity Execution

After the claim phase has completed, execution enters the Connectivity stage.

Unlike the legacy execution pipeline, the Smart Connectivity Runner never performs endpoint selection.

Its responsibility is limited to executing the workload that has already been prepared.

---

# Runtime Input

The Connectivity stage loads the runtime manifest produced during the claim phase.

```text
Smart Batch Selector
        │
        ▼
Claim Layer
        │
        ▼
smart_batch_ids.json
        │
        ▼
run_smart_connectivity()
        │
        ▼
run_connectivity_ids()
```

The runtime manifest becomes the single source of truth for the current execution cycle.

No additional scheduling decisions are made.

---

# No Database Scheduling

One of the architectural goals of Smart Mode is removing scheduling responsibility from execution workers.

Therefore the Connectivity Runner never performs operations such as:

- selecting candidate endpoints;
- filtering endpoint status;
- calculating execution priority;
- deciding batch composition;
- claiming additional database rows.

All of these decisions have already been completed before execution begins.

---

# Deterministic Execution

Because every worker receives a predefined list of endpoint identifiers, the execution becomes deterministic.

Given the same runtime manifest, every execution cycle processes exactly the same workload.

This property simplifies:

- debugging;
- monitoring;
- performance analysis;
- execution reproducibility.

---

# Worker Scheduling

The runtime manifest is passed directly into the Connectivity Runner.

The runner divides the endpoint identifiers into batches according to the configured worker count.

Each endpoint is then assigned to an independent worker process.

```text
Claimed IDs
      │
      ▼
Batch Generator
      │
      ▼
Worker 1
Worker 2
Worker 3
Worker N
```

Each worker executes exactly one endpoint at a time.

Workers never communicate with one another.

---

# Responsibility Boundary

The Smart Connectivity Runner has only one responsibility:

Execute Connectivity validation for the endpoint identifiers provided by the runtime manifest.

Everything before this stage belongs to scheduling.

Everything after this stage belongs to result processing and Quality validation.

This strict separation significantly reduces coupling between orchestration logic and validation logic.