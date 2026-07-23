# V2 Architecture

## Overview

Version 2 replaces the sequential processing model of Version 1 with a distributed pipeline architecture.

Instead of waiting for one complete processing cycle to finish before the next cycle begins, every stage operates as an independent long-running worker that continuously maintains its own responsibility.

The entire system is designed around persistent endpoint buffers rather than temporary processing stages.

---

# Core Design Goals

Version 2 was designed to solve the limitations discovered in Version 1.

Primary objectives:

- Continuous endpoint production
- Eliminate long sequential processing cycles
- Keep Reserve continuously populated
- Maintain Hot pool automatically
- Reduce idle processing time
- Isolate worker responsibilities
- Improve fault tolerance
- Support continuous publishing

---

# High-Level Architecture

```
                Raw Configs
                     │
                     ▼
        Raw Producer Pipeline
                     │
                     ▼
              Reserve Pool
                     ▲
                     │
        Reserve Health Worker
                     │
                     ▼
              Hot Manager
                     │
                     ▼
             Publish Pipeline
                     │
                     ▼
            Public Subscription
```

---

# System Philosophy

Unlike Version 1, every processing stage owns a permanent responsibility.

Workers do not execute a full platform cycle.

Instead, they continuously maintain a single layer of the architecture.

---

# Processing Layers

## Layer 1

Raw Production

Responsible for continuously producing new Reserve candidates.

Documentation:

- RawToReservePipeline.md

---

## Layer 2

Reserve Maintenance

Responsible for maintaining Reserve quality.

Documentation:

- ReserveHealthPipeline.md

---

## Layer 3

Hot Management

Responsible for maintaining the production-ready Hot pool.

Documentation:

- HotManagerPipeline.md

---

## Layer 4

Publishing

Responsible for exporting public configurations.

Documentation:

- PublishPipeline.md

---

# Independent Workers

Every worker is an independent process.

Workers communicate only through the database.

Workers never call each other directly.

Each worker owns its own processing loop.

Each worker can restart independently.

Failure of one worker does not stop the remaining workers.

---

# Buffer-Based Architecture

Version 2 introduces persistent endpoint buffers.

```
Raw
 │
 ▼
Reserve
 │
 ▼
Hot
 │
 ▼
Published
```

Each buffer is continuously maintained by its dedicated worker.

Buffers always contain validated endpoints.

---

# Admission Strategy

Endpoints never skip layers.

Every endpoint must pass every validation stage before entering the next buffer.

```
Connectivity

↓

Quality

↓

Reserve Admission

↓

Reserve

↓

Reserve Health

↓

Hot Promotion

↓

Hot

↓

Publishing
```

---

# Worker Responsibilities

## Raw Producer

Produces Reserve candidates.

Creates new Reserve entries.

Never modifies Hot.

---

## Reserve Health

Maintains Reserve quality.

Removes degraded endpoints.

Protects Reserve integrity.

Never performs publishing.

---

## Hot Manager

Promotes Reserve endpoints.

Maintains Hot capacity.

Runs long validation.

Detects correlated failures.

Removes degraded Hot endpoints.

---

## Publisher

Exports validated public endpoints.

Does not perform Quality testing.

Does not maintain Reserve.

Does not maintain Hot.

---

# Fault Isolation

Every worker owns its own:

- Lock
- Recovery
- Retry policy
- Claim lifecycle

Failures remain isolated inside the owning worker.

---

# Continuous Processing

Unlike Version 1, processing never stops after a complete cycle.

Workers execute continuously.

The system therefore behaves as a continuously maintained platform rather than a batch processor.

---

# Documentation Map

Architecture

- V2_Architecture.md

Pipelines

- RawToReservePipeline.md
- ReserveHealthPipeline.md
- HotManagerPipeline.md
- PublishPipeline.md

Lifecycle

- EndpointLifecycle.md

Contracts

- PipelineContracts.md