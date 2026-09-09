# Data Synchronization

[Specification index](../README.md) | [General principles](general.md)

Synchronization obtains flag results already evaluated by FeatBit for the active user. Both online modes use the same identity, storage, and readiness rules.

## Initialization and readiness

Online initialization completes only after an accepted synchronization response for the active context is committed. A valid empty response also completes initialization. A socket connection or HTTP success without a valid response is insufficient.

Startup timeout ends only the caller's wait. Continue recovery after transient failures; settle pending waits on terminal rejection or close. Applications must be able to observe later recovery even after an earlier wait timed out.

Distinguish data availability from synchronization state:

| State | Meaning | Local evaluation |
| --- | --- | --- |
| Waiting | Initial synchronization or a new user's refresh is pending. | Use matching cache/bootstrap when available; otherwise fallback. |
| Ready | The active context has synchronized successfully, or offline initialization completed. | Read local values. |
| Stale | The active context synchronized previously, but synchronization is now interrupted. | Continue using its retained values. |
| Closed | The client has permanently stopped background activity. | Retained values remain readable, without analytics. |

Equivalent names or separate status properties are allowed. On Identify, freshness must describe the new context; the previous user's readiness does not establish readiness for it. Terminal synchronization rejection must be distinguishable from a recoverable outage through status or an error outcome.

An accepted response restores Ready after commit, even when it changes no flag values. Routine intervals between successful polls do not make data Stale. Cached or bootstrap values alone do not prove online freshness.

## WebSocket mode (required)

SDKs MUST support WebSocket and use it by default:

1. Connect using the client SDK key.
2. Request synchronization with the active user and that context's committed cursor, or zero when a full refresh is needed.
3. Accept applicable full or patch responses and commit them using [storage rules](storage.md#updates).
4. Publish readiness, complete applicable waits, and notify the application after committed data is visible.

Maintain the protocol's application keepalive. Transient failures and server disconnects, including normal closure (`1000`), trigger paced automatic reconnection. Server rejection (`4003`) and explicit close stop reconnection. Retain current data and its cursor during outages and resynchronize after reconnecting.

Optional initial Polling fallback follows [configuration](configuration.md#optional-polling-settings). A stopped or replaced connection cannot later become an active source again through delayed work.

## Polling mode (optional)

When implemented, Polling MUST be selectable at creation. Send the active user to the client polling endpoint, with the committed cursor or zero, then process the same full/patch envelope used by WebSocket. Exact requests are in the [protocol reference](../reference/protocol.md#polling).

Start promptly and serialize requests. Use finite request deadlines, a positive polling interval, and paced retries. Network failures, timeouts, and retryable server responses retain local data; terminal rejection stops recovery and settles waiting operations. Respect applicable retry delays supplied by the service.

A `304` response changes no data or cursor. It may confirm a matching, previously server-synchronized data set, including a cached one; it cannot establish a new data set from bootstrap values or an empty uninitialized store.

## Invalid and unrelated data

Ignore unknown top-level message types for forward compatibility. Malformed JSON, an invalid envelope, or an unsupported full/patch type MUST NOT change data, cursor, readiness, or Identify completion. Ignore unrelated or superseded user responses under [User Identity](identity.md#isolation-and-overlapping-changes).

Within a valid, applicable envelope, decode flags independently. Skip entries with unusable keys, values, or versions while accepting valid siblings:

- A full response replaces the remote data with valid entries. Skipped entries and their old remote copies are absent.
- A patch leaves skipped entries unchanged. Skipped versions do not advance the cursor.
- An otherwise valid response follows normal readiness rules, including an all-skipped response.

Report ignored messages and skipped entries through safe diagnostics. Later valid updates must continue to work. Skipped records are not guaranteed to be replayed; recovery may require a later update or full replacement.

## Closing

Close stops synchronization and recovery, finishes pending waits, and prevents late responses from committing data or publishing readiness. It must be safe before initialization and during Identify, retry, or a transport change. See [lifecycle](public-api.md#lifecycle).
