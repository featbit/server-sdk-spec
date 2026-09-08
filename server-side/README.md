# FeatBit Server-Side SDK Specification

This specification describes the behavior of FeatBit server-side SDKs across languages. It focuses on what applications can configure, how data and events are handled, and what applications can expect from the public API.

**Status: Draft.** Version 1.0 is not yet a released conformance target.

## Core specification

| Module | Scope |
| --- | --- |
| 1. [User Configuration](spec/configuration.md) | Environment secret, server URLs, startup, offline mode, and event settings. |
| 2. [Data Synchronization](spec/synchronization.md) | Required WebSocket and optional Polling modes, initialization, updates, recovery, and readiness. |
| 3. [Data Storage](spec/storage.md) | Local data, full replacement, versioned updates, and consistent reads. |
| 4. [Feature Flag Evaluation](spec/evaluation.md) | User context, targeting order, percentage rollout, results, and fallback. |
| 5. [Event Processing](spec/events.md) | Event collection, bounded buffering, batch delivery, retries, and flush. |
| 6. [Public API](spec/public-api.md) | Application-facing operations and their observable behavior. |
| 7. [General Principles](spec/general.md) | Error containment, independence, resource use, diagnostics, and compatibility. |

Read the modules in this order for an overview. Requirement levels and scope are defined in [General Principles](spec/general.md). All seven modules apply together.

## Supporting references

The [server protocol](reference/protocol.md) and [evaluation compatibility rules](reference/evaluation.md) retain exact details needed for interoperability. These are shared contracts; they do not prescribe a language's classes, locks, queues, threads, or asynchronous APIs.

The [acceptance checklist](spec/conformance.md) summarizes observable scenarios to verify. [Shared fixtures](fixtures/README.md) are planned; no executable shared fixture suite is currently included.

The .NET Server SDK is a behavioral reference, not a mandatory architecture or proof of conformance. Existing SDK behavior must be checked against this draft before claiming compliance.
