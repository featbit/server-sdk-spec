# Public API

[Specification index](../README.md) | [General principles](general.md)

SDKs MUST provide language-appropriate equivalents of the following capabilities. Names, builders, result types, and synchronous/asynchronous forms may differ. An operation may be part of client creation rather than a separate method.

## Operations

| Capability | Inputs and outcome |
| --- | --- |
| Configure and create | Accept configuration and the initial user; start online or offline initialization. |
| Create user context | Accept a stable key, optional name, and custom attributes with string, null, or omitted values. |
| Wait for initialization | Bounded wait; distinguish success, timeout, terminal failure, and close. |
| Read readiness/status | Distinguish local availability, the current context's synchronization state, and closure. |
| Identify | Replace the active user or attributes; provide a bounded completion outcome after applicable data becomes visible. |
| Typed variation | Accept flag key and fallback; return a local value for the active user or the exact fallback. |
| Detailed variation | Same evaluation plus flag key, reason category, and available explanation. |
| All variations | Return raw-string details for active local flags without analytics. |
| Track | Accept event name and optional numeric value, defaulting to 1.0; use the active user. |
| Flush | Request prompt event processing, with a bounded completion-waiting form. |
| Subscribe/unsubscribe | Observe flag-value changes and readiness/status changes. |
| Close | Stop background work, attempt a bounded final flush, and release owned resources. |

Generic variation, parsed JSON helpers, anonymous-user generation, persistent cache controls, and platform-specific lifecycle hooks may supplement these operations. Framework bindings and dependency injection are not required.

## Error behavior

Public APIs MUST contain recoverable SDK errors. Evaluation returns fallback; other operations use ordinary error results, status, or diagnostics. Such failures must not escape as exceptions, panics, or rejected asynchronous operations requiring exception handling.

Invalid configuration produces a creation error before background work starts. Invalid Identify leaves the active context unchanged. Invalid Track input is rejected locally. Bulk calls without an available data set return an empty collection. Status, subscription, flush, and close errors must also remain contained and observable.

These guarantees concern recoverable SDK failures; runtime-fatal failures remain subject to host-language rules.

## Change notifications

Applications need notifications to update their interface when local results change. SDKs MUST support subscription and unsubscription for changed flag keys and readiness/status changes. A separate per-flag subscription is optional.

Notify when a flag becomes available, disappears, is archived/restored, or changes its effective value or declared type. Full replacement and Identify can also change the visible flag set. An equal-version update with a different value must notify; a duplicate or older ignored update must not. Metadata-only changes need not trigger value notifications.

Publish notifications only after the complete update and related status are visible. Handlers must be able to read the new values immediately. Do not publish a success notification for a rejected user response or partially applied batch.

Preserve committed update order. Subscriber errors must not affect other subscribers or SDK operation. Avoid invoking application code inside state-update work; slow handlers must not cause unbounded pending work. Handlers must be able to evaluate, unsubscribe, or request close safely.

Document callback context, any coalescing, and the treatment of already-running callbacks during unsubscribe or close. Optional coalescing must preserve the affected keys and let the application read the latest committed view. Applications can subscribe and then read current values; a new subscription need not replay earlier changes.

## Lifecycle

Applications SHOULD reuse one client per environment and active application session. Each additional client isolates its configuration, active user, data, status, subscriptions, and events.

Close MUST be safe before initialization, during Identify or recovery, and when called repeatedly or concurrently. It settles pending initialization and Identify waits, prevents new synchronization work, attempts the final event flush, and releases SDK-owned resources within a bounded time. Caller-owned resources are released only when ownership was explicitly transferred.

After close completes, no new network work or notification may begin. Late responses cannot change data. Retained local values remain available for evaluation and bulk reads without analytics; Identify and Track have no effect. A closed instance is not restarted.

Browser unload and mobile suspension may prevent final delivery. Optional lifecycle integration must respect platform limits, document resume behavior, and preserve user isolation. Closing the SDK does not imply logout or cache erasure; persistence and data clearing follow the documented cache policy.
