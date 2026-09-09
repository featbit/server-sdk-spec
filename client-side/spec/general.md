# General Principles

[Specification index](../README.md)

## Scope and requirement levels

This draft defines observable behavior for client-side SDKs used in browsers, mobile applications, and similar user-facing runtimes. The model is one active user context per client instance, with targeting evaluated by FeatBit and results read locally.

**MUST** means required behavior, **SHOULD** means recommended behavior whose omission needs a documented reason, and **MAY** means optional capability.

All eight core modules apply together. Supporting protocol fields are compatibility contracts. No particular classes, internal interfaces, threads, locks, queues, persistence technology, or framework integration are required.

## Protect application behavior

SDK failures must remain contained so the application can continue using an available value or its chosen fallback.

| Failure | Required boundary |
| --- | --- |
| Synchronization outage | Keep applicable local values and recover under the retry policy. |
| Malformed response | Preserve state or skip individual entries under the synchronization rules. |
| User refresh failure | Never substitute the previous user's values for the active context. |
| Persistent storage failure | Continue with in-memory data and report the loss of persistence. |
| Evaluation failure | Affect only the current flag; return the caller's fallback. |
| Event collection or delivery failure | Preserve evaluation results and synchronization. |
| Application callback or logger failure | Contain the error and preserve SDK operation. |
| Close or cancellation | Finish bounded cleanup without restarting background activity. |

See [Public API](public-api.md#error-behavior) for the common error contract. Runtime-fatal errors are outside the recovery guarantee.

## Predictable resource use

Local evaluation MUST avoid network requests and waiting for delivery. Keep reads coherent during synchronization, identity changes, flush, and close under the runtime's supported concurrent use.

Bound event retention, network request duration, caller waits, retries per delivery attempt, and shutdown work. Long-lived synchronization recovery may continue with paced attempts until success, terminal rejection, or close. Outages must not cause unlimited memory growth or continuous busy retries.

Respect browser and mobile runtime limits on networking, storage, and background execution. Resource ownership and optional suspension/resume behavior must be documented.

## Diagnostics and privacy

Expose useful diagnostics for configuration errors, initialization timeout, connection loss, rejected updates, fallback reasons, event loss, and shutdown timeout. Keep first-time initialization distinct from current-user freshness and connection health.

Never log SDK keys, authorization headers, or token-bearing URLs. Do not log user attributes or raw synchronization/event payloads by default. Use application-controlled logging and limit repetitive diagnostics.

Client-side applications and their local storage are accessible to the end user. Flag values must not contain server secrets, and a client-side flag result must not replace server-side authorization. SDK documentation must explain what user information synchronization and analytics transmit and how persistent user data can be cleared.

## Cross-language compatibility

Equivalent user contexts, flag results, update sequences, requested types, and fallbacks MUST produce equivalent observable results within documented numeric ranges. Preserve context isolation, equal-version update acceptance, archive handling, conversion behavior, reason categories, and event eligibility/deduplication.

Document supported runtimes, configuration defaults, numeric limits, optional capabilities, callback behavior, and best-effort delivery. Changes to shared behavioral or wire contracts require explicit cross-SDK review and corresponding acceptance-case updates.
