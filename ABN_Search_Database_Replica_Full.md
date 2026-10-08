# ABN Search Database Replica

## Overview

The ABN Search Database Replica is a cloud-native search platform designed to support ABN and Super Fund Lookup services by maintaining a near real-time replica of public Australian Business Register (ABR) data.

### Technologies

- AWS Database Migration Service (DMS)
- Change Data Capture (CDC)
- Amazon Aurora
- Amazon Kinesis
- AWS Lambda
- Event-driven ingestion architecture
- JSONB transformation and storage patterns

### Objectives

- Provide a dedicated search database for lookup workloads.
- Maintain near real-time synchronisation with ABR source data.
- Improve search performance and availability.
- Support future API and SPA-based lookup services.
- Preserve current and historical public ABR data.

---

## Business Context

ABN Lookup and Super Fund Lookup services require a modernised lookup platform aligned with the Australian Business Register (ABR) as the authoritative source of public business registry information.

The solution introduces a dedicated search platform using AWS-native services and near real-time replication to ensure data consistency, availability, and performance.

---

# Scope

## In Scope

### Search Database

- Creation of a new ABN Search Database Replica.
- Support for single-record lookup operations.
- Storage of public ABR information.
- Storage of current and historical records.

### Initial Data Migration

- Full initial migration from the ABR Client Database.
- Migration of current and historical public ABR records.
- Replication into the search database.

### Continuous Replication

- Ongoing Change Data Capture (CDC).
- Near real-time replication.
- Continuous synchronisation with the source database.

### Platform Components

- AWS DMS
- Amazon Aurora
- Amazon Kinesis
- AWS Lambda

## Out of Scope

- Super Fund specific population activities.
- Additional downstream DMS services.
- Additional Kinesis consumers.
- Additional Lambda integrations.
- Additional SQS integrations.
- API presentation logic.
- Consumer-facing suppression rules.

---

# Architecture Principles

## Source of Truth

The ABR Client Database remains the authoritative source of business registry information.

The Search Database is a replicated read-optimised dataset.

## Public Dataset Boundary

Only public ABR information is replicated.

## Historical Records

The platform stores both current and historical records.

## Event-Driven Processing

Replication is event-driven and initiated through change events.

---

# High-Level Architecture

```text
ABR Client Database
        │
        ▼
    Event Table
        │
        ▼
     AWS DMS
        │
        ▼
  Amazon Kinesis
        │
        ▼
Ingest ABN Record Lambda
        │
        ▼
 ABN Search Database
   (Amazon Aurora)
```

---

# Replication Workflow

## Source Updates

Changes made to business entities generate event records.

Events are associated with a Party ID.

## Event Table

The source system updates an Event Table whenever changes occur.

The Event Table acts as the primary trigger mechanism.

## Change Data Capture

AWS DMS continuously monitors the Event Table and transaction logs.

DMS:

- Detects changes
- Captures updates
- Extracts new events
- Publishes CDC data

## Streaming

Amazon Kinesis receives replication events and distributes them to consumers.

## Processing

The Ingest ABN Record Lambda function:

- Reads Kinesis messages.
- Filters non-ABN related events.
- Retrieves the current ABN record.
- Transforms the source data.

## Transformation

Data is transformed into:

- JSONB documents
- Relational database records

## Persistence

The transformed records are written into the Aurora Search Database.

---

# Aurora Search Database

## Characteristics

- High availability
- Low latency queries
- Managed operations
- Scalability
- Search optimisation

## Purpose

Provide a highly performant data store for lookup services.

---

# Initial Full Load

## Process

1. DMS performs a complete migration.
2. Data is copied from source to target.
3. Validation occurs.
4. Search Database mirrors source data.
5. CDC replication begins.

## Outcome

The target database matches the source state prior to ongoing replication.

---

# Change Data Capture (CDC)

## Purpose

Maintain continuous synchronisation.

## Behaviour

CDC:

- Reads transaction logs.
- Captures inserts.
- Captures updates.
- Captures deletes.
- Converts source changes.
- Applies updates to the target database.

## Data Freshness

Changes should appear within acceptable replication latency.

---

# Acceptance Criteria

## Aurora Search Database

GIVEN a database capability is required

WHEN an ABN Details API request occurs

THEN the Aurora Search Database is available and contains all required data attributes.

## Initial Migration

GIVEN an initial load is required

WHEN DMS performs migration

THEN the Search Database mirrors the source database

AND CDC processing can commence.

## Near Real-Time Replication

GIVEN CDC is active

WHEN source changes occur

THEN CDC continuously extracts and applies those changes.

## Data Ordering

GIVEN multiple updates occur

WHEN replication completes

THEN the latest valid state is retained.

## Recovery

GIVEN interruptions occur

WHEN processing resumes

THEN replication resumes from a safe checkpoint without data loss.

---

# Data Integrity

## Requirements

Replicated information must remain:

- Accurate
- Complete
- Consistent

## Concurrent Updates

The final state must always reflect the latest valid update.

## Historical Retention

Historical records must be retained consistently with the source database.

---

# Suppressed Records

Suppressed records remain part of the replica dataset.

Replication preserves suppression indicators.

Exposure rules are enforced by APIs and consuming applications rather than the database.

---

# Performance Requirements

## Data Freshness

Source changes must be reflected within acceptable latency.

## Search Performance

The system must provide:

- Reliable response times
- Low latency
- Consistent query performance

---

# Security Requirements

## Encryption

All data transfers must be encrypted in transit.

## Data Protection

Data must be protected during:

- Transfer
- Processing
- Storage

---

# Monitoring and Observability

Monitoring must provide visibility into:

- Replication health
- Latency
- System failures
- Throughput
- Performance issues

## Alerting

Alerts must be generated for:

- Replication failures
- Processing issues
- Operational degradation

---

# Assumptions

- Only public lookup information is replicated.
- AWS infrastructure is provisioned.
- Monitoring capability exists.
- Logging capability exists.
- Teams can support Aurora and DMS.

---

# Dependencies

- Physical Data Model (PDM)
- Logical Data Model (LDM)
- AWS infrastructure
- Data validation testing
- Performance testing
- Replication testing

---

# Benefits

## Operational

- Automated replication
- Reduced maintenance effort
- Improved resilience

## Data

- Near real-time updates
- Improved consistency
- Improved accuracy

## Performance

- Faster queries
- Better scalability
- Improved availability

---

# Risks

## Technical Complexity

The solution introduces:

- Amazon Aurora
- AWS DMS
- CDC processing
- Event-driven architecture

## Capability Risk

The solution requires expertise in:

- Aurora
- DMS
- Kinesis
- Lambda
- Backend development

---

# Technical Component Summary

## Source Layer

- ABR Client Database
- Event Table

## Replication Layer

- AWS DMS
- CDC

## Streaming Layer

- Amazon Kinesis

## Processing Layer

- Ingest ABN Record Lambda

## Storage Layer

- Amazon Aurora Search Database

## Operational Layer

- Monitoring
- Alerting
- Logging
- Recovery Services

---

# Non-Functional Requirements

- Near real-time replication
- Continuous synchronisation
- Data integrity
- Historical data retention
- High availability
- Low latency search
- Secure transport
- Observability
- Automated recovery
- Scalable architecture
