# Event Processing

[Specification index](../README.md) | [General principles](general.md)

Events support usage analytics and experimentation. Collection must remain inexpensive; delivery runs independently of local evaluation and synchronization.

## What is recorded

| Operation or condition | Event behavior |
| --- | --- |
| Successful individual evaluation of a remote result | Record after conversion succeeds, once the active context has completed online synchronization. |
| Matching remote cache before the active context's first successful synchronization | Return usable values, but suppress evaluation events until synchronization confirms that context. |
| Stale remote result after successful synchronization | Remains eligible; a temporary outage does not change its origin. |
| Local bootstrap result | No evaluation event, even after other flags synchronize. |
| Fallback, conversion failure, or bulk read | No evaluation event. |
| Track | Record a named metric for the active user; numeric value defaults to 1.0. Initialization is not required. |
| Offline, events disabled, or client closing/closed | Suppress new event collection. Offline and disabled modes also suppress delivery. |

Capture the user, attributes, value, and timestamp at the call. Later Identify calls, input mutation, and retries MUST NOT change that event.

For evaluation events, use the server's selected variation metadata and `sendToExperiment` decision. Do not perform experiment sampling locally. If the selected variation cannot be mapped to valid analytics metadata, omit the event and report a diagnostic while preserving the evaluated value.

Track requires a non-empty event name and a finite numeric value. Reject invalid input without affecting evaluation or synchronization.

## Collection and deduplication

Use bounded buffering without waiting for network delivery. When capacity is exhausted, drop the new event and make the loss observable. Bound total retained work, payload sizes, batch sizes, and delivery concurrency.

The client-side event contract requires **deduplication within each flush group**:

- Events are duplicates when their complete logical payloads are equal except for timestamp. Compare user attributes by meaning, not object property order.
- Keep the first occurrence and its original timestamp. Apply this to evaluation events and custom metric events separately.
- Different users, attributes, flag variations, experiment eligibility, event names, or metric values are not duplicates.
- Split into delivery batches after deduplication. Do not deduplicate across separate flush groups.

For example, two identical Track calls in one flush group produce one metric payload; the same call in a later group produces another. **Track therefore does not guarantee one delivered event per invocation.** This behavior differs from the server-side event contract and must be documented for applications that count occurrences.

Send periodically and on explicit flush. Additional size-based triggers MAY be offered. Exact intervals and batch sizes are SDK choices. Invalid events must not block unrelated valid events.

## Delivery and failures

| Outcome | Behavior |
| --- | --- |
| HTTP 2xx | Batch accepted. |
| HTTP 400, 408, 429; server errors; transient network failure | Eligible for bounded retry. |
| Other HTTP 4xx | Stop subsequent event delivery for this client; preserve evaluation and synchronization. |
| Shutdown deadline reached | Stop outstanding delivery within the cleanup budget. |

Use finite attempts, request deadlines, and documented retry delays. Retries preserve the original payload. A terminal delivery failure must prevent subsequent batches from starting, including batches from the same flush group.

Delivery is best effort. Overflow, exhausted retries, application suspension, and process exit can lose events; lost responses can cause duplicate delivery. Flush-group deduplication does not guarantee exactly-once delivery.

## Flush and shutdown

Provide a way to request prompt processing and a bounded way to wait for completion. These may be forms of the same operation. A waiting flush covers events accepted before the call, including earlier in-flight work. Later events need not delay it.

Completion means those events were delivered or reached a final failure/drop outcome. Report timeout separately; do not describe processing completion as guaranteed delivery. Document whether delivery continues after a waiting timeout.

Close stops new collection and attempts a bounded final flush before releasing resources. A full buffer, failed request, or suspended runtime must not create an unbounded close. See [Public API](public-api.md#lifecycle).
