# Data Synchronization

[Specification index](../README.md) | [General principles](general.md)

The SDK maintains local environment data using **WebSocket only**. Polling is not a supported synchronization mode in this specification. HTTP is used separately for event delivery.

## Connection and initialization

1. Connect to the configured streaming server using the environment secret.
2. Request a full data set when no synchronization cursor exists; otherwise request changes using the committed cursor.
3. Receive and process feature flags and segments.
4. Make the accepted update available locally, then publish readiness and complete initialization.

An empty valid data set also initializes the client. Startup timeout ends only the caller's wait; recovery continues in the background. Terminal rejection or explicit close must finish pending initialization waits with an appropriate outcome.

The server can send a full replacement even when changes were requested. Both full and patch messages follow [storage update rules](storage.md#updates). The exact authentication, message format, cursor representation, and application keepalive are in the [protocol reference](../reference/protocol.md).

## Readiness and freshness

| Public state | Meaning | Evaluation |
| --- | --- | --- |
| NotReady | Initial usable data has not been established. | Return the caller's fallback. |
| Ready | Initial data is available and synchronization is healthy, or offline initialization succeeded. | Evaluate local data. |
| Stale | Previously initialized online data remains available, but synchronization is interrupted. | Continue evaluating the retained data. |
| Closed | Synchronization has permanently stopped for this client. | Retained initialized data remains locally evaluable; events are suppressed. |

Equivalent state names are allowed. Initialization describes whether usable initial data was established; it does not mean the connection is currently healthy. Reconnection alone MUST NOT change Stale to Ready: a valid synchronization response is required. A valid patch with no effective changes can restore readiness.

## Recovery

Transient connection failures and server disconnects MUST trigger automatic reconnection, including normal server closure (`1000`). Server rejection (`4003`) and explicit client close stop reconnection for that client instance.

Retries SHOULD be paced to avoid excessive load during an outage. The schedule, delay limits, and optional jitter are SDK choices and must be documented. Retryable outages must not permanently stop recovery merely because an initial wait expired.

Retain the local data and cursor across connection loss. On reconnect, request synchronization again. Closing the client must prevent pending connection work from reopening it.

## Invalid data

Ignore unknown top-level message types for forward compatibility. Invalid JSON, an invalid synchronization envelope, or an unsupported full/patch event type MUST be logged and ignored without changing data, cursor, initialization, or status. Continue receiving later messages.

Within a valid envelope, decode each flag and segment independently. If an entity cannot be decoded, including an unusable identity or version, log and skip that entity while processing valid siblings:

- A full response replaces local data with the successfully decoded entities. Skipped entities and their previous copies are absent.
- A patch applies valid entities using version rules. Skipped entities leave previous copies unchanged.
- Skipped versions do not contribute to the cursor. A valid response still follows normal readiness rules, even if all entities were skipped.

Thus an all-skipped full response leaves an empty store; an all-skipped patch changes no data. Skipping does not guarantee replay: other valid entities may advance the cursor beyond the skipped version. Recovery depends on a later applicable update or full replacement; forced resynchronization is not required.

SDKs need not duplicate the server's complete business validation. Errors encountered while evaluating stored data are contained to the affected flag, as described in [evaluation](evaluation.md#failure-behavior).

Parsing diagnostics MUST identify whether a message was ignored or an entity skipped, with its safe key/ID or position when available. Use the configured logger's error level and filtering. Repeated errors may be summarized, but the first and continuing failures must remain observable. Do not expose credentials or raw payloads.
