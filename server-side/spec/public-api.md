# Public API

[Specification index](../README.md) | [General principles](general.md)

SDKs MUST provide language-appropriate equivalents of the following capabilities. Names, builders, result types, synchronous/asynchronous forms, and resource-management syntax may differ. An operation may be integrated into client creation rather than exposed as a separate method.

## Operations

| Capability | Inputs and outcome |
| --- | --- |
| Configure and create client | Accept environment configuration and start online or offline initialization; report invalid configuration without throwing. |
| Create user context | Accept a stable user key, optional name, and custom attributes. |
| Wait for initialization | Bounded wait or asynchronous equivalent; distinguish ready, timeout, and terminal failure/close. |
| Read initialization state | Report whether an initial usable data state was established. |
| Read client status | Distinguish NotReady, Ready, Stale, and Closed, or equivalent states. |
| Typed variation | Accept flag key, user, and caller fallback; return the selected value or that fallback. |
| Detailed variation | Same evaluation plus flag key, variation ID, reason category, and reason text. |
| All variations | Return raw-string details for active flags; no analytics; isolate errors per flag. |
| Track | Accept user, event name, and optional numeric value, defaulting to 1.0. |
| Flush | Request prompt processing of pending events without waiting. |
| Flush and wait | Wait for previously accepted events to finish processing within a timeout; completion does not guarantee delivery. |
| Close | Stop background activity, attempt a bounded final flush, and release owned resources. |
| Subscribe/unsubscribe (optional) | Observe committed local data changes. |

The typed and detailed APIs MUST agree on value selection and fallback. Numeric ranges and optional JSON helpers follow [evaluation](evaluation.md). Fluent user/configuration builders, dependency injection, and separate float/double overloads are not mandatory.

## Error behavior

Public SDK APIs MUST NOT propagate recoverable SDK errors as exceptions, panics, or rejected asynchronous operations that require exception handling. Use ordinary error results, status, fallback values, and diagnostics appropriate to the language.

Invalid configuration must yield a clear validation/creation error without starting background activity. Invalid evaluation input returns the supplied fallback and an Error detail. Invalid Track input is rejected without affecting evaluation. Bulk calls with unusable input return an empty result with an error outcome or diagnostic.

Status, flush, subscription, and close failures must also remain contained and observable. These guarantees cover recoverable errors; runtime-fatal failures are governed by the host language.

## Lifecycle

Applications SHOULD reuse one client per environment. Different clients must isolate credentials, data, status, and events.

Close MUST be safe before initialization, during connection recovery, and when called repeatedly or concurrently. It ends pending initialization waits and prevents new connections, event requests, and callback invocations after completion. Resources supplied by the caller are released only when ownership was explicitly transferred.

After close, retained initialized data remains available for read-only evaluation and bulk reads, with no analytics. If initialization never succeeded, typed evaluation continues to return fallback. A stopped client is not restarted; create a new client.

## Optional data-change notifications

Notifications are preliminary and MAY be omitted without affecting core conformance. If offered, they identify full versus patch updates and whether flags and/or segments changed:

- A full replacement reports both categories, even for an empty replacement.
- A patch reports only categories with effective inserts, updates, or archives; duplicate/older/skipped records do not trigger a patch notification.
- Segment-only changes are observable because they can affect flag results.

Publish notifications after the data and readiness state are visible, in committed update order. They report data changes, not guaranteed changes to a particular user's evaluated value. Per-flag identification and replay are not required.

Subscriber errors must not affect other subscribers or SDK operation. Slow handlers must not delay synchronization. Handlers must be able to evaluate flags or request close safely. Document callback context, unsubscribe semantics, treatment of callbacks already running at close, and any coalescing policy.

Applications needing an initial view should subscribe and then read the current data; initial synchronization may already have occurred before subscription.
