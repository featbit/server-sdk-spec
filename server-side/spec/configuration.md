# User Configuration

[Specification index](../README.md) | [General principles](general.md)

Configuration identifies the FeatBit environment and controls how the SDK connects and uses resources. Option names and configuration styles may follow each language's conventions.

## Common settings

| Setting | Expected behavior |
| --- | --- |
| Environment secret | Required for online mode; authenticates access to one environment. Never expose it in diagnostics. |
| Streaming server URL | Configurable WebSocket base URL using `ws` or `wss`. |
| Synchronization mode | WebSocket by default and required in every SDK; user-selectable Polling when implemented. |
| Event server URL | Configurable HTTP base URL using `http` or `https`; may differ from the streaming server. |
| Offline mode | Disabled by default. When enabled, use local bootstrap data and perform no network communication. |
| Disable events | False by default. When enabled, events (flag evaluation events and metric events for A/B testing) will no longer be sent to the event server. |
| Bootstrap data | Local flags and segments for offline use. |
| Startup wait | A bounded initial wait for usable data; expiry does not stop background recovery. |
| WebSocket connection timeout and reconnect policy | Limit WebSocket connection attempts and control retry frequency during outages. |
| Event flush interval | Control how often pending events are sent automatically. |
| Event capacity and batch size | Bound retained events and the size of each delivery request. |
| Event retry policy and request timeout | Bound retry attempts and time spent on failed delivery. |
| Shutdown timeout | Bound the final event-processing attempt and resource cleanup. |
| Logging | Use application-controlled logging, disabled or silent by default. |

SDKs MUST expose credentials, server URLs, offline mode, event enablement, startup waiting, and the main event capacity/flush settings. Lower-level timeout and retry controls MAY be grouped or supplied as documented defaults. Exact durations, worker counts, and retry schedules are not cross-language requirements.

SDK documentation MUST state defaults, units, valid ranges, and URL path handling. A deployment prefix such as `/featbit/` must be supported without silently routing requests to a different path. Use secure URL schemes when communicating over untrusted networks.

## Polling settings (optional)

SDKs implementing [Polling mode](synchronization.md#polling-mode-optional) MUST expose synchronization mode, polling server URL, and polling interval. The names below describe configuration concepts; public option names may follow each language's conventions.

| Setting | Expected behavior |
| --- | --- |
| Synchronization mode | Select `WebSocket` or `Polling` at client creation; defaults to `WebSocket`. Selecting `Polling` starts only the polling worker. Reject `Polling` if the SDK does not implement it. |
| Polling server URL | HTTP base URL using `http` or `https`, pointing to the Evaluation Server or FeatBit Agent. Resolve `api/public/sdk/server/latest-all` beneath this base, preserving deployment prefixes. Document any default; if none is provided, require an explicit URL in Polling mode. |
| Polling interval | Positive duration between completion of one successful polling cycle and the next request. The initial request starts promptly without waiting for this interval. SDKs MUST provide and document a default, units, and supported range. Requests MUST NOT overlap. |
| Polling request timeout | Positive, finite duration bounding each HTTP request, including response-body retrieval. A timeout follows polling failure and recovery rules. May be exposed separately or supplied as a documented default. |
| Polling retry policy | Controls delays after transient failures. Document the retry schedule, delay limits, and any jitter; honor applicable `Retry-After` responses. Recovery continues until success, terminal rejection, or close, regardless of startup wait expiry. After success, resume the normal polling interval. May be grouped with other timeout/retry controls or supplied as documented defaults. |

Polling uses the common environment secret, startup wait, logging, and shutdown timeout. Its endpoint and scheduling are independent of event delivery settings. Offline mode starts no polling worker. Exact duration defaults remain SDK-specific, consistent with the common settings policy.

Validate the polling URL, interval, and request timeout before starting Polling mode. A polling client does not require a streaming URL. The selected mode and effective polling settings remain immutable after initialization.

## Validation

Validate settings before starting background activity. Online mode requires a usable secret and a valid URL for the selected synchronization mode: a streaming URL for WebSocket or an HTTP polling URL for Polling. Reject unsupported synchronization modes. Event delivery requires a valid event URL. Capacity and batch limits must be positive, and timeout/retry values must have documented, valid meanings.

Invalid configuration or malformed bootstrap input MUST be reported through a non-throwing validation or creation result carrying a clear, actionable error message identifying the invalid setting and why it failed, consistent with [public API error behavior](public-api.md#error-behavior). Do not start a partially configured client or silently connect to a different environment.

A running client's effective configuration MUST remain immutable after initialization.

## Initialization and offline use

Online startup begins the selected synchronization mode and optionally waits for the configured startup period. A connected socket or successful HTTP status alone is insufficient: usable data must be committed before the client becomes ready. See [Data Synchronization](synchronization.md).

Offline initialization loads a complete bootstrap data set, or an empty data set if none is provided, and becomes ready without a secret or remote connection. An unknown flag then returns a `flag-not-found` fallback rather than a not-ready fallback.

Online bootstrap, persistent caches, custom transports, and dynamic configuration are optional extensions. An SDK offering them MUST explain their effect on readiness, freshness, and resource ownership.
