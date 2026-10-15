# File overview examples

Read these before drafting an overview. Each starts with the example's filename as the subject and states its responsibility, then explains the relevant problem, rule, or design. Use the actual filename, including its extension, in real code. Verify the stated assumptions and apply SKILL.md's purpose/design minima.

The opening is a complete sentence, not a filename heading. `ip_rules.go parses access rules and matches client addresses` identifies a responsibility; `ip_rules.go contains helpers` does not. Use the filename-led structure consistently, and let the actual responsibility determine the verb and object. Background and rationale follow the purpose statement.

## Purpose overview: a refresh failure

Assume `catalog_refresh.go` refreshes the model catalog. A failed refresh retains the previous catalog, and replacement entries are parsed and validated before publication. The reader needs to know which catalog remains available during that work.

```go
// catalog_refresh.go refreshes the model catalog and keeps validated entries
// available for lookups. Cached entries are reused until a replacement has been
// parsed and validated. Refresh errors leave the previous entries available, so
// readers can continue model lookup while a valid replacement is being
// prepared.
```

## Design overview: an account slot's lifetime

Assume `account_slots.go` limits concurrency and queues requests per account. Cancellation can race with admission, and release is idempotent. Explain the lifetime of a slot before describing the coordination that protects it.

```go
// account_slots.go limits concurrent requests per account and queues requests
// until a slot is available. A waiting request can be cancelled without
// stopping other requests. Each account admits requests in arrival order and
// counts a request as active until its caller releases the slot after reading
// or closing the response. Routing may choose another eligible account before a
// request joins the queue.
//
// Queue membership and occupied slots change under the same mutex. Cancellation
// removes a waiter that has not been admitted. When cancellation races with a
// grant, the unused slot is returned before the next waiter is admitted.
// Release is idempotent because normal completion and cleanup may both call it.
```

## Purpose overview: the editable unit

Assume `provider_tables.go` replaces provider tables it generated and preserves other TOML content. Make the unit of replacement and the user's retained content clear.

```go
// provider_tables.go updates generated provider tables in TOML configuration
// while preserving the user's other text. These tables are replaced as complete
// units. Text outside those tables stays as written, including comments and
// ordering, so reconnecting a provider preserves the user's surrounding
// settings.
```

## Purpose overview: a dependency limitation

Assume `proxy_bridge.go` supplies a loopback HTTP proxy for SOCKS5 selections because the plugin runtime accepts HTTP proxies but rejects SOCKS5.

```go
// proxy_bridge.go bridges SOCKS5 proxies to the HTTP proxy interface accepted
// by the plugin runtime. A loopback HTTP proxy forwards plugin requests through
// the selected SOCKS5 server because the runtime cannot use SOCKS5 directly.
```

## Design overview: valid rules and configured rules

Assume `ip_rules.go` parses and matches IP access rules. Compilation skips invalid IP patterns but retains their original count, and the API-key allowlist check rejects unmatched addresses whenever that count is positive. Separate parsing behavior from the configuration decision it supports.

```go
// ip_rules.go parses IP access rules and matches addresses against individual
// IPs or CIDR ranges. CompileIPRules prepares a list for repeated checks;
// MatchesPattern parses one rule for a single check.
//
// Compilation trims whitespace and skips invalid entries, but PatternCount
// includes every supplied entry. API-key allowlist checks use this count to
// distinguish an absent allowlist from a configured list with no valid rules. A
// configured allowlist with no valid rules rejects every address.
//
// Single-pattern matching and validation parse their inputs as supplied.
// NormalizeIP trims an address and removes its port when SplitHostPort
// succeeds.
```

## Purpose overview: the stored records and their index

Assume `ledger_view.go` is a facade over a ledger and index implemented elsewhere. The JSONL ledger is authoritative, readers use immutable index blocks, and an append copies at most the unfinished final block. The facade needs a purpose overview; the component that implements publication and consistency owns the design explanation.

```go
// ledger_view.go exposes ledger records through a shared index for time-window
// queries. The JSONL ledger holds the records, and its in-memory index can be
// rebuilt from them. Readers decode the blocks that overlap their requested
// time range, while appending a record copies at most the unfinished final
// block.
```

## Design overview: a value split across stream events

Assume `placeholder_writer.go` restores known placeholders in streamed responses, holds a possible unfinished placeholder, and continues it only with later text from the same channel. It emits an unmatched tail literally when the stream ends.

```go
// placeholder_writer.go restores placeholders in streamed responses, including
// placeholders split across text events. A placeholder can arrive in pieces:
// one text event ends with {{TOKEN_ and the next begins with name}}. Replacing
// values one event at a time would leave the placeholder visible to the caller,
// even though joining the text first would have found it.
//
// The writer holds a trailing fragment that could begin a placeholder and
// continues it with later text from the same channel. Complete text can be
// emitted immediately. An unfinished fragment stays buffered until it can be
// matched or ruled out; when the stream ends, any unmatched tail is emitted
// literally so the caller receives all of the original text.
```

## Design overview: elapsed work and operation counts

Assume `request_timing.go` records request timings and builds Server-Timing headers. It merges completed intervals, clips them to the request window, and counts each operation once. A mutex protects recording and snapshot creation; merging and formatting use the copied snapshot.

```go
// request_timing.go records request timings and builds Server-Timing headers
// from completed calls. Concurrent calls can overlap, so adding their durations
// overstates how long the request spent waiting. Two calls that each take 10
// milliseconds and overlap for 5 occupy 15 milliseconds together, while still
// counting as two operations. Elapsed work is the union of their intervals
// within the request window; operation counts remain additive.
//
// Recording continues while the response header is prepared. The collector
// copies counts and intervals under its mutex, then releases the mutex before
// merging intervals and formatting the header. This keeps one header's inputs
// consistent while allowing later observations to be recorded for subsequent
// snapshots.
```
