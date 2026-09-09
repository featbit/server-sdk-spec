# User Identity

[Specification index](../README.md) | [General principles](general.md)

Each client instance has one active user context. Evaluation and Track use that context without requiring a user argument on every call.

## User information

A user has a non-empty, stable string key, an optional name (empty by default), and custom attributes whose non-null values are strings. Preserve key and attribute case. Reserved fields `keyId` and `name` cannot be overridden through custom attributes. Custom attribute names must be non-empty and unique.

Custom attribute values may be null or omitted. During custom-attribute rule evaluation, a condition referencing an absent attribute immediately fails to match, including negative comparisons; the server must not substitute an empty string for the missing attribute. A present attribute with a null or omitted value is compared as an empty string. Explicit empty strings are compared normally, and the literal string `"null"` remains distinct. See the [user payload contract](../reference/protocol.md#user-payload); the SDK does not perform this rule evaluation locally.

The application supplies only the attributes needed for targeting and analytics. Creating a user locally does not itself send an event; synchronization transmits the user so the server can evaluate flags. Disabling analytics does not suppress this synchronization payload.

An anonymous user is still a user with a stable key. Automatic anonymous-key generation and persistence MAY be offered, but are not required. If offered, document when that key changes. Logging in or out requires an explicit identity change; the SDK must not infer account transitions from application state.

## Changing the active user

Identify replaces the whole active context, including its attributes. Changing attributes under the same key requires fresh server evaluation, just as changing the key does. An omitted old attribute is removed rather than implicitly retained.

An accepted identity change MUST:

1. Validate and capture the new context. Invalid input leaves the current user and values unchanged.
2. Switch local reads to data belonging to the new context. Until those values are available, return the caller's fallback or applicable bootstrap values.
3. Online, request results for that context. Use a cursor only with a matching cached data set; otherwise request a full refresh.
4. Complete successfully only after an applicable server response is accepted and its results are visible. Offline, complete after the local context is established.

For example, after switching from Alice to Bob, an immediate read may use Bob's cache, an applicable local default, or the supplied fallback. It must never return Alice's result as Bob's.

Report completion, timeout, failure, supersession, or close through an ordinary operation outcome. A timeout ends the caller's wait; recovery may continue for the current context. A failed refresh does not silently switch back to the previous user.

## Isolation and overlapping changes

The most recently accepted Identify call defines the active context. A late response for an earlier context MUST NOT replace current data, advance its cursor, emit a current-user change notification, or complete a newer Identify call.

This rule also covers attribute changes under the same key and sequences such as Alice → Bob → Alice. Matching a response's user key alone is insufficient to establish that it belongs to the latest context. SDKs must preserve this behavior without requiring a particular concurrency mechanism.

Each evaluation and event uses one consistent context. An event recorded before Identify keeps its original user and attributes when sent later. Mutating a caller-owned user object cannot change stored identity or queued events.
