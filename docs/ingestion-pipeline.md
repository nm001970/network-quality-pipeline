# Data Ingestion Pipeline

## Overview

The Data Ingestion Pipeline is responsible for collecting endpoint configurations from external sources and preparing them for downstream processing.

Rather than validating or testing endpoints immediately, this stage focuses on acquiring, normalizing and safely storing candidate configurations.

Every subsequent processing stage operates on the data produced by this pipeline.

---

## Pipeline Flow

```text
GitHub Repository
        │
        ▼
download_configs()
        │
        ▼
parse_configs()
        │
        ▼
sanitize_configs_for_storage()
        │
        ▼
add_hashes()
        │
        ▼
save_configs()
        │
        ▼
SQLite Database
```

---

## Stage 1 — Configuration Collection

### download_configs()

The ingestion process begins by downloading raw endpoint lists from external GitHub repositories.

At this stage no validation is performed.

The downloaded content is treated as untrusted raw input.

**Input**

- Remote GitHub repository

**Output**

- Raw text containing endpoint configurations

---

## Stage 2 — Protocol Detection

### parse_configs()

The parser scans the downloaded text and extracts supported endpoint configurations.

Only supported protocols are accepted.

Currently supported protocols include:

- VMess
- VLESS
- Trojan
- Shadowsocks

Each accepted configuration is converted into a lightweight structured object.

Example:

```python
{
    "protocol": "vless",
    "config": "vless://..."
}
```

This stage performs protocol identification only.

It does not parse protocol-specific parameters.

---

## Stage 3 — Configuration Sanitization

### sanitize_configs_for_storage()

Before persistence, every configuration is normalized.

Typical operations include:

- Removing unnecessary metadata
- Removing promotional tags
- Normalizing configuration format
- Preparing a canonical representation for storage

The sanitizer produces a cleaned configuration while preserving protocol compatibility.

---

## Stage 4 — Hash Generation

### add_hashes()

Every sanitized configuration receives a deterministic cryptographic hash.

The hash uniquely identifies the configuration regardless of its source.

The generated hash is later used for duplicate detection and database integrity.

Example:

```
config_hash
113dfd7dc4edddf52bd44028d4ce69337e91e7eea69255cde0e8b5de3bd06a57
```

---

## Stage 5 — Persistent Storage

### save_configs()

The final ingestion stage stores sanitized configurations inside the platform database.

Each record contains:

| Field | Description |
|--------|-------------|
| config_hash | Unique identifier |
| protocol | Endpoint protocol |
| config_text | Sanitized endpoint configuration |

Duplicate configurations are automatically rejected using the unique configuration hash.

No downstream worker communicates directly with GitHub.

All subsequent processing begins by reading records from the database.

---

## Why Database-First Processing?

MPROXY intentionally stores candidate configurations before connectivity validation begins.

This design provides several engineering advantages:

- Durable ingestion
- Crash recovery
- Repeatable processing
- Independent workers
- Duplicate prevention
- Simplified scheduling

Instead of streaming configurations directly into runtime validators, the platform builds a persistent processing queue inside the database.

This database-first architecture forms the foundation of the remaining processing pipeline.

---

## Result

After the ingestion pipeline completes, the platform owns a persistent collection of sanitized endpoint configurations.

At this point:

- Protocols are identified.
- Configurations are normalized.
- Duplicate detection is available.
- Every endpoint has a unique identity.
- Candidate endpoints are ready for connectivity validation.

No connectivity or quality testing has been performed yet.

---

## Current Implementation

The current implementation ingests endpoint configurations from a single GitHub repository.

The ingestion pipeline is intentionally designed as an isolated processing stage, allowing additional data sources to be integrated in future versions without changing downstream processing components.

Only the ingestion layer would require modification, while the remainder of the platform would continue operating unchanged.