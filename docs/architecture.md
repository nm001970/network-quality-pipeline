# Architecture

## Overview

MPROXY is a long-running backend automation platform that continuously discovers, validates, evaluates and delivers healthy network endpoints.

Instead of implementing all processing inside a single application, the platform separates every major responsibility into independent processing stages.

Each stage performs one dedicated task while exchanging validated data through a shared persistence layer.

This architecture improves maintainability, fault isolation and operational reliability.

---

## High-Level Architecture

The platform is organized into four major processing layers:


                 Data Sources
                      │
                      ▼
             Data Ingestion Layer
                      │
                      ▼
          Connectivity Validation
                      │
                      ▼
            Quality Assessment
                      │
                      ▼
             Pool Management
                      │
                      ▼
             Delivery Services

Processing Philosophy

Every stage of the platform follows the same engineering principles:

Single Responsibility
Independent Workers
Database-driven communication
Continuous automation
Runtime validation
Automatic recovery

Each processing layer is capable of operating independently without requiring direct knowledge of downstream components.

| Layer                   | Responsibility                                         |
| ----------------------- | ------------------------------------------------------ |
| Data Ingestion          | Collects, sanitizes and stores endpoint configurations |
| Connectivity Validation | Executes runtime connectivity validation using Xray    |
| Quality Assessment      | Measures latency, stability and endpoint quality       |
| Pool Management         | Maintains Reserve Pool and Hot Pool                    |
| Delivery Layer          | Publishes production-ready endpoints                   |


Why Database-Centered Processing?

Instead of passing data directly between workers, MPROXY uses the database as the communication layer.

This approach provides several engineering advantages:

Worker isolation
Crash recovery
Processing persistence
Easier debugging
Independent scheduling
Horizontal scalability

Each worker simply reads eligible records, processes them and writes updated results back into the database.

No worker depends directly on another running process.

Worker Model

Workers are designed as long-running background services.

Each worker repeatedly performs the following lifecycle:

Read eligible records
Claim a processing batch
Process independently
Update database state
Release resources
Repeat

This design enables continuous operation while minimizing synchronization complexity.

Runtime Validation

Unlike simple configuration validators, MPROXY validates endpoint configurations by executing them inside a real Xray runtime.

Every candidate endpoint is transformed into an executable Xray configuration before network testing begins.

This runtime-based validation provides significantly more reliable connectivity results than static configuration analysis.

Processing Independence

Every major subsystem is isolated.

Examples include:

Data collection
Connectivity testing
Quality evaluation
Pool management
Publication

Because each subsystem communicates only through persistent storage, failures remain localized and do not interrupt the remainder of the processing pipeline.

This architecture forms the foundation of MPROXY's production-oriented automation model.