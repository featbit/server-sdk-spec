# FeatBit Server-Side SDK Design Specification

This specification is organized by module. Start with [General Requirements](spec/general.md) for the shared scope, requirement levels, architecture, and reference baseline, then use the module documents below for implementation and review.

**Status: Draft.** This repository owns the shared server-side SDK specification. Version 1.0 is not yet a released conformance target; unresolved design decisions remain open.

The specification was migrated from [featbit/dotnet-server-sdk](https://github.com/featbit/dotnet-server-sdk/tree/ba83f029abdd512f907ba6dcbcdde62221f613c1/docs). The migration preserves existing behavior requirements. See [CHANGELOG](CHANGELOG.md) and [shared fixtures](fixtures/README.md).

## Modules

| Document | Scope |
| --- | --- |
| [General Requirements](spec/general.md) | Scope, requirement levels, architecture, public API, diagnostics, performance, and extensibility. |
| [Configuration and Initialization](spec/configuration.md) | Options, defaults, validation, startup, offline mode, and bootstrap. |
| [Data Synchronization and Reconnect](spec/synchronization.md) | Authentication, WebSocket protocol, full/patch updates, states, and reconnect policy. |
| [Data Model, Storage, and Retrieval](spec/storage.md) | Entity schema, versions, tombstones, store operations, and snapshot consistency. |
| [User Context and Feature Flag Evaluation](spec/evaluation.md) | User attributes, targeting, operators, segments, rollout, conversions, and experiment eligibility. |
| [Event Collection and Delivery](spec/events.md) | Collection rules, HTTP payloads, buffering, batching, retries, and flush semantics. |
| [Data-Change Notifications (Optional)](spec/notifications.md) | Preliminary .NET implementation that may change. Other SDKs MAY omit this capability for now. |
| [Error Isolation](spec/error_isolation.md) | Failure boundaries, malformed entities, dependency errors, quarantine, and recovery. |
| [Concurrency Requirements](spec/concurrency.md) | Shared-state invariants, synchronization, ownership, and race prevention. |
| [Shutdown and Resource Ownership](spec/shutdown.md) | Idempotent close, cancellation, worker joining, deadlines, and resource disposal. |
| [Conformance and Acceptance Criteria](spec/conformance.md) | Release-gate test matrix, shared fixtures, integration tests, and documentation requirements. |
| [Reference Implementation Notes](spec/reference.md) | Observed .NET implementation gaps, portability decisions, and source/test references. |

## Using this specification

- Apply the general requirements together with the relevant module requirements. Data-change notifications are optional; their requirements apply only to SDKs that implement the capability.
- Use [Conformance and Acceptance Criteria](spec/conformance.md) as the implementation and release checklist.
- Consult [Reference Implementation Notes](spec/reference.md) to distinguish compatibility behavior from required hardening.
- Link to individual module files or their named heading anchors when citing a requirement.
