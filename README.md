# FeatBit Server-Side SDK Design Specification

This specification is organized by module. Start with [General Requirements](spec/general.md) for the shared scope, requirement levels, and architecture, then use the module documents below for implementation and review.

**Status: Draft.** This repository owns the shared server-side SDK specification. Version 1.0 is not yet a released conformance target; unresolved design decisions remain open.

See [CHANGELOG](CHANGELOG.md) for history and [shared fixtures](fixtures/README.md) for fixture status.

## Modules

| Document | Scope |
| --- | --- |
| [General Requirements](spec/general.md) | Scope, requirement levels, architecture, public API, diagnostics, performance, and extensibility. |
| [Configuration and Initialization](spec/configuration.md) | Options, defaults, validation, startup, offline mode, and bootstrap. |
| [Data Synchronization and Reconnect](spec/synchronization.md) | Authentication, WebSocket protocol, full/patch updates, states, and reconnect policy. |
| [Data Model, Storage, and Retrieval](spec/storage.md) | Entity schema, versions, tombstones, store operations, and snapshot consistency. |
| [User Context and Feature Flag Evaluation](spec/evaluation.md) | User attributes, targeting, operators, segments, rollout, conversions, and experiment eligibility. |
| [Event Collection and Delivery](spec/events.md) | Collection rules, HTTP payloads, buffering, batching, retries, and flush semantics. |
| [Data-Change Notifications (Optional)](spec/notifications.md) | Preliminary capability that SDKs MAY omit for now. |
| [Error Isolation](spec/error_isolation.md) | Failure boundaries, malformed entities, dependency errors, quarantine, and recovery. |
| [Concurrency Requirements](spec/concurrency.md) | Shared-state invariants, synchronization, ownership, and race prevention. |
| [Shutdown and Resource Ownership](spec/shutdown.md) | Idempotent close, cancellation, worker joining, deadlines, and resource disposal. |
| [Conformance and Acceptance Criteria](spec/conformance.md) | Release-gate test matrix, shared fixtures, integration tests, and documentation requirements. |

## Using this specification

- Apply the general requirements together with the relevant module requirements. Data-change notifications are optional; their requirements apply only to SDKs that implement the capability.
- Use [Conformance and Acceptance Criteria](spec/conformance.md) as the implementation and release checklist.
- Link to individual module files or their named heading anchors when citing a requirement.
