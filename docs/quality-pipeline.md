# Quality Pipeline

## Overview

The Quality Pipeline evaluates the real-world performance of every endpoint that has successfully passed the Connectivity stage.

Unlike simple reachability testing, the Quality Pipeline launches a dedicated Xray runtime for each candidate endpoint and performs real network measurements before deciding whether the endpoint is eligible for production use.

Only endpoints that satisfy every mandatory quality requirement continue to the next decision layer.

---

# Pipeline Flow

```text
Alive Endpoint
        │
        ▼
Quality Candidate Selection
        │
        ▼
Worker Scheduling
        │
        ▼
Configuration Loading
        │
        ▼
Protocol Parsing
        │
        ▼
Xray Configuration Builder
        │
        ▼
Temporary Runtime Generation
        │
        ▼
Launch Xray Runtime
        │
        ▼
HTTP Connectivity Test
        │
        ▼
Download Performance Test
        │
        ▼
Optional Upload Test
        │
        ▼
Quality Normalization
        │
        ▼
Quality Score Calculation
        │
        ▼
Database Update
```

---

# Candidate Selection

The pipeline does not process every stored endpoint.

Only endpoints that satisfy both conditions are selected:

- Connectivity status is **Alive**
- Quality status is **Not Tested**

Candidate selection is performed atomically to prevent multiple workers from processing the same endpoint simultaneously.

---

# Parallel Worker Execution

Selected endpoints are divided into batches.

Each batch is executed by multiple independent worker processes using Python multiprocessing.

Every worker performs a completely isolated quality evaluation for a single endpoint.

This architecture provides:

- parallel execution
- fault isolation
- independent runtime environments
- better CPU utilization

---

# Runtime Preparation

Each worker performs several preparation steps before any network measurement begins.

These include:

1. Load endpoint from database
2. Detect protocol
3. Parse configuration
4. Build executable Xray configuration
5. Allocate a temporary SOCKS port
6. Generate a temporary Xray configuration file

Only after these steps does the actual network runtime start.

---

# Xray Runtime

The platform does not inspect endpoint quality statically.

Instead, every endpoint is executed inside an isolated Xray process.

This creates a real SOCKS proxy through which all quality measurements are performed.

After testing finishes, the runtime is completely destroyed.

This guarantees that every measurement reflects actual runtime behavior.

---

# Quality Evaluation

Every runtime is evaluated through several sequential stages.

## HTTP Connectivity

Verifies that HTTP traffic can successfully pass through the proxy.

---

## Download Performance

Measures practical downstream throughput.

Download performance is considered mandatory.

Endpoints that fail this stage are immediately rejected.

---

## Upload Performance

Upload testing is optional.

If upload measurement is available, the measured value contributes to the final evaluation.

Failure of the upload measurement itself does not automatically reject the endpoint.

---

# Result Normalization

Raw measurements are converted into a normalized Quality Result.

Normalization validates:

- average latency
- download threshold
- upload threshold
- mandatory requirements
- failure reasons

The output is transformed into a consistent internal representation independent of protocol type.

---

# Quality Score

Qualified endpoints receive a numerical Quality Score.

The score is calculated using measured runtime characteristics including:

- latency
- download throughput
- upload throughput
- historical success statistics

The Quality Score is stored for later ranking and selection.

---

# Database Update

After evaluation, the Quality Pipeline updates:

- Quality Status
- Quality Score
- Public Score
- Ping metrics
- Download speed
- Upload speed
- Last Quality Check timestamp
- Failure reason (if rejected)

The pipeline never publishes endpoints directly.

Its responsibility ends after persisting the validated quality information.

Subsequent architectural layers are responsible for production selection and deployment decisions.


----------------------------------------

Quality Validation Pipeline

Only endpoints that successfully pass connectivity validation enter the quality evaluation stage.

The quality pipeline performs real runtime measurements instead of relying on static endpoint metadata.

Each endpoint is processed independently through a dedicated worker, where a temporary Xray runtime environment is created, the endpoint is tested under real network conditions and the resulting measurements are normalized before being stored.

The pipeline is executed in parallel using multiple worker processes, allowing large endpoint collections to be evaluated continuously while maintaining resource isolation between workers.

Quality Validation Flow
Alive Endpoints
        │
        ▼
Claim Quality Batch
        │
        ▼
Parallel Worker Scheduler
        │
        ▼
Independent Quality Workers
        │
        ▼
Protocol Parsing
        │
        ▼
Runtime Xray Configuration
        │
        ▼
Launch Xray Instance
        │
        ▼
HTTP Connectivity Test
        │
        ▼
Download Performance Test
        │
        ▼
Optional Upload Verification
        │
        ▼
Quality Normalization
        │
        ▼
Quality Score Calculation
        │
        ▼
Database Update
Runtime Validation

Each quality worker performs an isolated validation cycle:

Loads endpoint configuration
Detects the protocol automatically
Generates a protocol-specific Xray configuration
Allocates a temporary local SOCKS port
Launches an isolated Xray process
Executes real connectivity measurements
Collects latency and throughput metrics
Normalizes validation results
Updates quality status and production score

Temporary runtime resources are removed immediately after every validation cycle.

Measured Metrics

Each endpoint is evaluated using runtime measurements including:

HTTP Connectivity
Minimum Ping
Average Ping
Download Throughput
Upload Throughput (optional)
Endpoint Stability
Final Quality Score

The Quality Pipeline ends after persisting validated quality information into the database. Subsequent orchestration layers decide when and whether qualified endpoints become part of the production export process