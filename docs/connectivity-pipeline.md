# Connectivity Validation Pipeline

## Overview

After endpoint configurations have been collected and stored inside the database, the Connectivity Validation Pipeline verifies whether each candidate endpoint is actually usable.

Unlike static configuration validators, MPROXY performs runtime validation by launching a real Xray instance for every tested endpoint.

This approach verifies practical connectivity rather than configuration syntax alone.

---

## Pipeline Flow

```text
SQLite Database
        │
        ▼
Claim Processing Batch
        │
        ▼
Worker Process
        │
        ▼
parse_config()
        │
        ▼
Configuration Builder
        │
        ▼
Temporary Xray Configuration
        │
        ▼
Launch Xray Runtime
        │
        ▼
Local SOCKS Proxy
        │
        ▼
Real Network Connectivity Test
        │
        ▼
Normalize Result
        │
        ▼
Persist Validation Result
        │
        ▼
SQLite Database
```

---

## Stage 1 — Batch Selection

Workers begin by requesting a batch of eligible endpoint records from the database.

Each worker receives an independent processing batch, allowing multiple workers to execute simultaneously without processing the same records.

The database acts as the coordination layer between workers.

---

## Stage 2 — Protocol Parsing

### parse_config()

Each stored configuration is parsed into a protocol-specific internal representation.

Unlike the ingestion parser, this parser extracts protocol parameters required for runtime execution.

Its responsibility is to transform a stored configuration into structured runtime data.

---

## Stage 3 — Runtime Configuration Builder

The parsed endpoint is converted into a complete Xray runtime configuration.

Protocol-specific builders generate executable configuration files that can be consumed directly by the Xray runtime.

This abstraction allows different protocols to share the same validation workflow.

---

## Stage 4 — Temporary Configuration

Instead of modifying permanent configuration files, MPROXY generates a temporary runtime configuration for each validation task.

Temporary configurations are isolated and discarded after testing completes.

This prevents interference between concurrent validation processes.

---

## Stage 5 — Runtime Execution

The platform launches an isolated Xray process using the generated configuration.

Each validation task executes inside its own runtime instance.

No shared runtime is reused across concurrent workers.

This process-level isolation improves reliability and fault containment.

---

## Stage 6 — Local Proxy Creation

The launched Xray instance exposes a temporary local SOCKS proxy.

Subsequent validation traffic is routed through this local proxy.

The endpoint is therefore tested exactly as it would be used by a real client.

---

## Stage 7 — Connectivity Validation

Network validation is performed through the temporary proxy.

Successful validation confirms that:

- the runtime started correctly
- the endpoint accepted traffic
- network communication completed successfully

At this stage the platform validates connectivity only.

No quality measurements are calculated yet.

---

## Stage 8 — Result Normalization

Raw validation results are converted into a standardized internal representation.

This normalization ensures that downstream processing stages receive consistent connectivity information regardless of protocol type.

---

## Stage 9 — Persistence

Validation results are written back into the database.

The database becomes the authoritative source of endpoint connectivity status.

Subsequent processing stages never repeat the connectivity test unless required.

---

## Engineering Characteristics

The Connectivity Validation Pipeline is designed around several engineering principles:

- Runtime-based validation
- Independent worker execution
- Process isolation
- Temporary runtime environments
- Database-driven coordination
- Protocol abstraction
- Automatic resource cleanup

---

## Why Runtime Validation?

Many endpoint validation systems perform only syntax checking or lightweight protocol inspection.

MPROXY validates endpoints by executing them inside a real Xray runtime.

This allows the platform to verify actual network connectivity instead of merely confirming that a configuration appears syntactically correct.

As a result, downstream processing operates only on endpoints that have successfully completed real runtime validation.

---

## Current Scope

The Connectivity Validation Pipeline is responsible only for determining whether an endpoint can establish a successful network connection.

Quality evaluation—including latency measurement, stability analysis and production scoring—is performed later by the Quality Assessment Pipeline.

# Connectivity Validation Pipeline

## Overview

After endpoint configurations have been collected and stored inside the database, the Connectivity Validation Pipeline verifies whether each candidate endpoint is actually usable.

Unlike static configuration validators, MPROXY performs runtime validation by launching a real Xray instance for every tested endpoint.

This approach verifies practical connectivity rather than configuration syntax alone.

---

## Pipeline Flow

```text
SQLite Database
        │
        ▼
Claim Processing Batch
        │
        ▼
Worker Process
        │
        ▼
parse_config()
        │
        ▼
Configuration Builder
        │
        ▼
Temporary Xray Configuration
        │
        ▼
Launch Xray Runtime
        │
        ▼
Local SOCKS Proxy
        │
        ▼
Real Network Connectivity Test
        │
        ▼
Normalize Result
        │
        ▼
Persist Validation Result
        │
        ▼
SQLite Database
```

---

## Stage 1 — Batch Selection

Workers begin by requesting a batch of eligible endpoint records from the database.

Each worker receives an independent processing batch, allowing multiple workers to execute simultaneously without processing the same records.

The database acts as the coordination layer between workers.

---

## Stage 2 — Protocol Parsing

### parse_config()

Each stored configuration is parsed into a protocol-specific internal representation.

Unlike the ingestion parser, this parser extracts protocol parameters required for runtime execution.

Its responsibility is to transform a stored configuration into structured runtime data.

---

## Stage 3 — Runtime Configuration Builder

The parsed endpoint is converted into a complete Xray runtime configuration.

Protocol-specific builders generate executable configuration files that can be consumed directly by the Xray runtime.

This abstraction allows different protocols to share the same validation workflow.

---

## Stage 4 — Temporary Configuration

Instead of modifying permanent configuration files, MPROXY generates a temporary runtime configuration for each validation task.

Temporary configurations are isolated and discarded after testing completes.

This prevents interference between concurrent validation processes.

---

## Stage 5 — Runtime Execution

The platform launches an isolated Xray process using the generated configuration.

Each validation task executes inside its own runtime instance.

No shared runtime is reused across concurrent workers.

This process-level isolation improves reliability and fault containment.

---

## Stage 6 — Local Proxy Creation

The launched Xray instance exposes a temporary local SOCKS proxy.

Subsequent validation traffic is routed through this local proxy.

The endpoint is therefore tested exactly as it would be used by a real client.

---

## Stage 7 — Connectivity Validation

Network validation is performed through the temporary proxy.

Successful validation confirms that:

- the runtime started correctly
- the endpoint accepted traffic
- network communication completed successfully

At this stage the platform validates connectivity only.

No quality measurements are calculated yet.

---

## Stage 8 — Result Normalization

Raw validation results are converted into a standardized internal representation.

This normalization ensures that downstream processing stages receive consistent connectivity information regardless of protocol type.

---

## Stage 9 — Persistence

Validation results are written back into the database.

The database becomes the authoritative source of endpoint connectivity status.

Subsequent processing stages never repeat the connectivity test unless required.

---

## Engineering Characteristics

The Connectivity Validation Pipeline is designed around several engineering principles:

- Runtime-based validation
- Independent worker execution
- Process isolation
- Temporary runtime environments
- Database-driven coordination
- Protocol abstraction
- Automatic resource cleanup

---

## Why Runtime Validation?

Many endpoint validation systems perform only syntax checking or lightweight protocol inspection.

MPROXY validates endpoints by executing them inside a real Xray runtime.

This allows the platform to verify actual network connectivity instead of merely confirming that a configuration appears syntactically correct.

As a result, downstream processing operates only on endpoints that have successfully completed real runtime validation.

---

## Current Scope

The Connectivity Validation Pipeline is responsible only for determining whether an endpoint can establish a successful network connection.

Quality evaluation—including latency measurement, stability analysis and production scoring—is performed later by the Quality Assessment Pipeline.