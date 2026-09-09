# Feature Flag Evaluation

[Specification index](../README.md) | [General principles](general.md)

Evaluation reads the active user's locally available result and converts its value. It MUST NOT perform targeting, fetch data, or wait for event delivery.

## Evaluation flow

For a typed variation request:

1. Validate the flag key and active user context.
2. Find an applicable local result, including matching cache or bootstrap values allowed by [storage rules](storage.md).
3. If no result is available, return the caller's fallback. Report `ClientNotReady` when neither a local data set nor successful synchronization has been established for the current context; otherwise report `FlagNotFound` for a missing flag.
4. Treat an archived flag as `FlagNotFound`.
5. Convert the stored value to the requested type. Conversion failure returns the exact caller fallback with `WrongType`.
6. Return the result and record an evaluation event only when [event eligibility](events.md#what-is-recorded) permits it.

A known local value can be returned before online initialization. A network outage does not force a usable value to fallback. A server-selected disabled or default variation is also a normal successful result; it is distinct from the application's fallback.

## Values and conversion

SDKs MUST support string, boolean, and numeric results. JSON is available as a string; a parsed JSON helper SHOULD be provided where the language has a suitable representation. A generic variation method using the declared type is optional.

| Requested type | Conversion behavior |
| --- | --- |
| String | Return the stored string unchanged. |
| Boolean | Accept case-insensitive `true` and `false`; do not use truthiness. |
| Number | Parse a complete, locale-independent decimal value, including decimal exponents. Reject empty values, non-finite values, overflow, and unrelated text. |
| Integer, if offered | Additionally reject fractional values and values outside the supported range. |
| Parsed JSON, if offered | Parse valid JSON; malformed JSON returns fallback. JSON `null` is a valid parsed result. |

Numeric parsing ignores surrounding whitespace, but must not interpret hexadecimal or language-specific numeric literals. Document numeric precision and ranges. Convert the selected string rather than rejecting it solely because its declared type differs from the requested type.

## Detailed results

Detailed evaluation MUST include the flag key, returned value, a reason category, and explanatory text when available. Preserve the server's match reason for successful remote results; do not reconstruct targeting reasons locally.

| Reason | Meaning |
| --- | --- |
| Match | Returned an applicable local value successfully. |
| ClientNotReady | No usable data state has been established for the current context. |
| FlagNotFound | The flag is absent or archived in the applicable data. |
| WrongType | The stored value cannot be converted to the requested type. |
| Error | Invalid input or another contained evaluation failure. |

Equivalent language-native names are allowed. SDKs MAY also expose result origin and selected variation ID. A fallback must never claim a successful match or a selected variation ID. A local bootstrap result must not invent a server match reason.

The value-only and detailed APIs MUST agree. A recoverable error affects only that evaluation, preserves committed data, and does not trigger resynchronization. Analytics failure cannot change a successful result.

## All variations

Return raw-string details for all active, locally available flags for the current context, including applicable bootstrap values. Exclude archived flags. An empty store returns an empty collection; this operation does not wait for online initialization and produces no analytics.

Use one coherent local view and isolate errors per flag. An unreadable entry reports an Error detail with no selected variation and an empty string value, without preventing other results. Returned JSON or collection objects must not expose mutable SDK state.
