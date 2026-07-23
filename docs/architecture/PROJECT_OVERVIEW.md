# MN Proxy Platform
## High-Level Architecture Overview

> This document provides a high-level overview of the platform architecture.
>
> It intentionally omits implementation details, internal algorithms,
> operational policies, infrastructure topology, and proprietary optimization
> techniques.
>
> Detailed technical documentation is available only under a formal
> collaboration or review process.

---

# Project Purpose

MN Proxy Platform is an automated V2Ray configuration management platform.

The platform continuously discovers, evaluates, validates, promotes and
maintains high quality proxy configurations with minimal manual intervention.

The system is designed around autonomous maintenance pipelines instead of
traditional static subscription generation.

---

# Design Philosophy

Instead of publishing every discovered configuration, the platform applies
multiple independent validation stages before allowing a configuration to
become publicly available.

Every stage has a different responsibility.

This separation keeps the system maintainable while preventing low quality
configurations from reaching end users.

---

# High Level Lifecycle

```
Internet
    │
    ▼
Discovery
    │
    ▼
Connectivity Verification
    │
    ▼
Quality Evaluation
    │
    ▼
Reserve Admission
    │
    ▼
Reserve Monitoring
    │
    ▼
Hot Promotion
    │
    ▼
Public Export
```

Each stage has independent responsibilities and validation policies.

---

# Platform Components

## Discovery Layer

Responsible for collecting candidate configurations.

Responsibilities include:

- discovering new configurations
- initial storage
- duplicate filtering
- metadata generation

---

## Connectivity Layer

Responsible for determining whether a configuration is technically usable.

Typical outputs include:

- alive
- dead
- unreachable

Only alive configurations continue.

---

## Quality Layer

Evaluates usable configurations.

Typical metrics include:

- latency
- download capability
- upload capability
- stability

The result determines whether a configuration satisfies platform quality
requirements.

---

## Reserve Layer

Accepted configurations are placed inside the Reserve pool.

Reserve acts as the platform's protected inventory.

Responsibilities include:

- keeping validated backup configurations
- continuous health verification
- removing degraded configurations
- supplying promotion candidates

---

## Hot Layer

Hot configurations are the highest quality configurations currently available.

The Hot layer performs continuous validation before exposing configurations
to users.

Responsibilities include:

- promotion
- continuous monitoring
- automatic replacement
- maintaining stable public availability

---

## Public Export

Public subscriptions are generated only from Hot configurations.

No configuration bypasses previous validation stages.

---

# Autonomous Maintenance

The platform is composed of independent maintenance services.

Typical responsibilities include:

- quality evaluation
- reserve maintenance
- hot maintenance
- subscription generation
- monitoring
- recovery

Each service operates independently while sharing a common data model.

---

# Fault Isolation

Every maintenance stage is isolated.

Failure in one stage does not interrupt unrelated platform components.

This allows:

- graceful recovery
- easier debugging
- predictable behaviour
- modular maintenance

---

# Scalability

The architecture is designed to support:

- multiple operators
- multiple protocol types
- multiple validation policies
- multiple export targets

without changing the overall workflow.

---

# Security

Several internal protection mechanisms are intentionally omitted from this
document.

Examples include:

- internal validation policies
- promotion logic
- scheduling policies
- recovery strategies
- quality heuristics
- synchronization mechanisms
- deployment topology

These are considered implementation details.

---

# Repository Structure

The repository is organized around independent functional modules rather
than one large execution pipeline.

Typical modules include:

- configuration processing
- quality evaluation
- reserve management
- hot management
- subscription export
- database layer
- monitoring
- maintenance services

---

# Development Principles

The platform follows several engineering principles.

- Separation of responsibilities
- Independent maintenance services
- Database driven state transitions
- Deterministic processing
- Failure recovery
- Reusable components
- Observable execution

---

# Documentation

Public documentation intentionally focuses on architecture.

Detailed implementation documents are maintained separately.

Examples include:

- Architecture
- Raw Pipeline
- Reserve Pipeline
- Hot Pipeline
- Operational Procedures

Additional documentation may be provided during technical collaboration.

---

# Collaboration

This repository intentionally exposes the architectural design while keeping
implementation details private.

Organizations interested in reviewing the platform for collaboration,
integration, licensing or technical evaluation are welcome to contact the
project owner.

Additional documentation and implementation details can be shared under an
appropriate review process.