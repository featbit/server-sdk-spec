# User Configuration

[Specification index](../README.md) | [General principles](general.md)

Configuration identifies the FeatBit environment and controls how the SDK connects and uses resources. Option names and configuration styles may follow each language's conventions.

## Common settings

| Setting | Expected behavior |
| --- | --- |
| Environment secret | Required for online mode; authenticates access to one environment. Never expose it in diagnostics. |
| Streaming server URL | Configurable WebSocket base URL using `ws` or `wss`. |
| Event server URL | Configurable HTTP base URL using `http` or `https`; may differ from the streaming server. |
| Offline mode | Disabled by default. When enabled, use local bootstrap data and perform no network communication. |
| Disable events | False by default. Suppress analytics while retaining normal online synchronization and evaluation. |
| Bootstrap data | Initial local flags and segments for offline use; an empty data set is valid. |
| Startup wait | A bounded initial wait for usable data; expiry does not stop background recovery. |
| Connection timeout and reconnect policy | Limit connection attempts and control retry frequency during outages. |
| Event flush interval | Control how often pending events are sent automatically. |
| Event capacity and batch size | Bound retained events and the size of each delivery request. |
| Event retry policy and request timeout | Bound retry attempts and time spent on failed delivery. |
| Shutdown timeout | Bound the final event-processing attempt and resource cleanup. |
| Logging | Use application-controlled logging, disabled or silent by default. |

SDKs MUST expose credentials, server URLs, offline mode, event enablement, startup waiting, and the main event capacity/flush settings. Lower-level timeout and retry controls MAY be grouped or supplied as documented defaults. Exact durations, worker counts, and retry schedules are not cross-language requirements.

SDK documentation MUST state defaults, units, valid ranges, and URL path handling. A deployment prefix such as `/featbit/` must be supported without silently routing requests to a different path. Use secure URL schemes when communicating over untrusted networks.

## Validation

Validate settings before starting background activity. Online mode requires a usable secret and a valid streaming URL; event delivery requires a valid event URL. Capacity and batch limits must be positive, and timeout/retry values must have documented, valid meanings.

Invalid configuration or malformed bootstrap input MUST be reported through a non-throwing validation or creation result, consistent with [public API error behavior](public-api.md#error-behavior). Do not start a partially configured client or silently connect to a different environment.

A running client's effective configuration MUST remain stable when the caller later edits the original options or collections. Dynamic reconfiguration is an optional extension.

## Initialization and offline use

Online startup begins synchronization and optionally waits for the configured startup period. A connected socket alone is insufficient: usable data must be committed before the client becomes ready. See [Data Synchronization](synchronization.md).

Offline initialization loads a complete bootstrap data set, or an empty data set if none is provided, and becomes ready without a secret or remote connection. An unknown flag then returns a flag-not-found fallback rather than a not-ready fallback.

Online bootstrap, persistent caches, custom transports, and dynamic configuration are optional extensions. An SDK offering them MUST explain their effect on readiness, freshness, and resource ownership.
