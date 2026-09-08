# General Principles

[Specification index](../README.md)

## Scope and requirement levels

This draft defines observable behavior for trusted, multi-user server applications that hold an environment secret and locally evaluate a complete environment data set.

**MUST** means required behavior, **SHOULD** means recommended behavior whose omission needs a documented reason, and **MAY** means optional capability.

The core specification consists of seven modules. Protocol fields and deterministic evaluation rules in the supporting references are also compatibility contracts. No particular class hierarchy, internal interface, locking scheme, thread model, queue library, or framework integration is required.

## Protect application behavior

Public APIs MUST contain recoverable SDK failures. Evaluation returns the caller's fallback on failure; other operations use non-throwing results, status, or diagnostics as described in [Public API](public-api.md#error-behavior). Invalid configuration must be reported clearly before background work begins.

A failure in one responsibility must not unnecessarily disable another:

| Failure | Required boundary |
| --- | --- |
| Streaming outage | Retain usable local data and recover according to the reconnect policy. |
| Invalid synchronization data | Isolate messages or entities using the documented skip rules. |
| Evaluation failure | Affect only the current flag; preserve other evaluations and committed data. |
| Event collection or delivery failure | Preserve evaluation results and synchronization. |
| Application callback or logger failure | Contain recoverable errors and preserve SDK operation. |
| Close or cancellation | Complete bounded cleanup without restarting background activity. |

Runtime-fatal failures remain subject to host-language rules. This specification does not require an SDK to recover from process termination or an unrecoverable runtime state.

## Predictable resource use

Evaluation MUST use local data without network requests or waiting for events delivery. Reads and updates must remain coherent under the runtime's supported concurrent use.

SDKs MUST bound event retention, network attempts, caller waits, and shutdown work. Outages and repeated errors must not cause unlimited memory growth or continuous busy retries. Changes to caller-owned inputs must not silently alter recorded events or committed SDK state.

SDKs may use threads, tasks, event loops, or other native mechanisms. The requirement is safe, predictable application behavior, including simultaneous evaluation, synchronization, flush, and close.

## Diagnostics and privacy

Expose useful diagnostics for initialization failure, connection loss, rejected data, fallback reasons, event loss, and shutdown timeout. Status must distinguish initialization from current connection health.

Use application-controlled logging. Never log environment secrets, authorization headers, or credential-bearing URLs. User attributes and raw data/event payloads must not be logged by default. Diagnostic detail must be safe, and repeated failures must not produce an unbounded workload.

## Cross-language compatibility

The same flag data, user attributes, and requested value type must produce equivalent results within documented numeric ranges. Preserve targeting order, missing-attribute semantics, wire fields, rollout assignment, reason categories, and event eligibility.

Changes to these contracts require explicit cross-SDK review and corresponding fixture updates. Runtime-specific API spelling, regex dialect limitations, numeric ranges, and optional extensions must be documented.

The [.NET Server SDK](https://github.com/featbit/featbit-dotnet-sdk) helps explain existing behavior, but its internal implementation and current limitations do not automatically become requirements. This document is a shared design target, not a claim that any existing SDK already satisfies every requirement.
