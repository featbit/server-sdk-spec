# Data Synchronization

[Specification index](../README.md) | [General principles](general.md)

## Synchronization modes

SDKs MUST support **WebSocket** synchronization and MAY also support **Polling** over HTTP. If Polling is implemented, users MUST be able to select WebSocket or Polling when configuring the client. WebSocket is the default. Only the selected mode runs for an online client; offline mode starts neither. Event delivery is configured independently of the synchronization mode.

Configuration is defined in [User Configuration](configuration.md), including [Polling settings](configuration.md#polling-settings-optional). The shared behavior below applies to both online modes.

## Shared synchronization behavior

### Initialization

Usable data MUST be committed before readiness is published and initialization completes. An empty valid data set also initializes the client. Startup timeout ends only the caller's wait; recovery continues in the background. Terminal rejection or explicit close must finish pending initialization waits with an appropriate outcome.

### Readiness and freshness

| Public state | Meaning | Evaluation |
| --- | --- | --- |
| NotReady | Initial usable data has not been established. | Return the caller's fallback. |
| Ready | Initial data is available and synchronization is healthy, or offline initialization succeeded. | Evaluate local data. |
| Stale | Previously initialized online data remains available, but synchronization is interrupted. | Continue evaluating the retained data. |
| Closed | Synchronization has permanently stopped for this client. | Retained initialized data remains locally evaluable; events are suppressed. |

Equivalent state names are allowed. Initialization describes whether usable initial data was established; it does not mean the transport is currently healthy. Reconnection or HTTP success alone MUST NOT change Stale to Ready: a valid synchronization response is required. A valid patch with no effective changes can restore readiness.

### Invalid data

Ignore unknown top-level message types for forward compatibility. Invalid JSON, an invalid synchronization envelope, or an unsupported full/patch event type MUST be logged and ignored without changing data, cursor, initialization, or status. Continue processing later synchronization responses.

Within a valid envelope, decode each flag and segment independently. If an entity cannot be decoded, including an unusable identity or version, log and skip that entity while processing valid siblings:

- A full response replaces local data with the successfully decoded entities. Skipped entities and their previous copies are absent.
- A patch applies valid entities using version rules. Skipped entities leave previous copies unchanged.
- Skipped versions do not contribute to the cursor. A valid response still follows normal readiness rules, even if all entities were skipped.

Thus an all-skipped full response leaves an empty store; an all-skipped patch changes no data. Skipping does not guarantee replay: other valid entities may advance the cursor beyond the skipped version. Recovery depends on a later applicable update or full replacement; forced resynchronization is not required.

Errors encountered while evaluating stored data are contained to the affected flag, as described in [evaluation](evaluation.md#failure-behavior).

Parsing diagnostics MUST identify whether a message was ignored or an entity skipped, with its safe key/ID or position when available. Use the configured logger's error level and filtering. Repeated errors may be summarized, but the first and continuing failures must remain observable. Do not expose credentials or raw payloads.

## WebSocket mode (required)

### Connection and initialization

1. Connect to the configured streaming server using the environment secret.
2. Request a full data set when no synchronization cursor exists; otherwise request changes using the committed cursor.
3. Receive and process feature flags and segments.
4. Make the accepted update available locally, then publish readiness and complete initialization.

Both full and patch messages follow [storage update rules](storage.md#updates). The exact authentication, message format, cursor representation, and application keepalive are in the [protocol reference](../reference/protocol.md).

### Recovery and shutdown

Transient connection failures and server disconnects MUST trigger automatic reconnection, including normal server closure (`1000`). Server rejection (`4003`) and explicit client close stop reconnection for that client instance.

Retries SHOULD be paced to avoid excessive load during an outage. The schedule, delay limits, and optional jitter are SDK choices and must be documented. Retryable outages must not permanently stop recovery merely because an initial wait expired.

Retain the local data and cursor across connection loss. On reconnect, request synchronization again. Closing the client must prevent pending connection work from reopening it.

## Polling mode (optional)

An SDK implementing Polling MUST satisfy the following requirements in addition to the shared storage, readiness, and invalid-data rules. See the [Retrieve feature flags API documentation](https://docs.featbit.co/sdk/retrieve-feature-flags#server-side) for the server-side HTTP API.

### Requests and updates

1. Send an initial `GET <polling-base>/api/public/sdk/server/latest-all` without a `timestamp` query parameter. The base URL points to the Evaluation Server or FeatBit Agent and MUST preserve configured deployment prefixes.
2. Set `Authorization` to the raw server environment secret, without a `Bearer` prefix or WebSocket connection-token encoding. No user context is required: evaluation remains local.
3. Process the JSON `data-sync` envelope containing `data.eventType`, `data.featureFlags`, and `data.segments`. Apply `full` and `patch` according to [storage update rules](storage.md#updates), including archive markers.
4. Commit data and its cursor consistently before publishing readiness. Derive the cursor from committed `updatedAt` versions across both flags and segments, converted to Unix milliseconds; never use the request time or local clock. An empty full response initializes an empty store with cursor zero. An empty patch retains the existing data and cursor.
5. After initialization, repeat the request with `?timestamp=<committed-cursor>` at the configured polling interval. Failed or ignored responses MUST NOT advance the cursor. Skipped entities follow the shared [invalid-data rules](#invalid-data).

### Scheduling

Use the [Polling settings](configuration.md#polling-settings-optional) for the endpoint, interval, request timeout, and retry policy. Start the first request promptly at online startup. Serialize requests and update application so a delayed response cannot overwrite newer data; do not accumulate overlapping requests when the server is slow.

### Recovery and freshness

A timeout, network failure, or unsuccessful HTTP response MUST retain committed data and cursor. An uninitialized client remains `NotReady`; an initialized client becomes `Stale` on such a failure. Retry transient failures automatically with bounded request duration and a documented paced retry policy; respect `Retry-After` when provided for throttling or temporary unavailability. Document terminal rejection handling and complete pending initialization waits when synchronization permanently stops.

A valid synchronization response restores Ready after its update is committed, including a patch with no effective changes. Invalid payloads follow the [invalid-data rules](#invalid-data) and do not change readiness. The normal interval between successful polls does not itself make data Stale. Startup wait expiry MUST NOT stop polling or recovery.

### Shutdown

Closing the client MUST stop scheduled polls and retries, cancel or bound in-flight requests, and prevent late responses from committing data or restoring readiness after close. Retained initialized data remains evaluable under the shared Closed-state rules.
