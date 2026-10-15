# Comment examples

Each excerpt assumes the behavior stated beside it. Verify those assumptions in the code. Apply SKILL.md's coverage and minima to the whole task; excerpts are not complete files. File-level examples are in [file-overviews.md](file-overviews.md).

## Definitions and calling rules

Definition tier: label is the configured display label. Private visibility still requires documentation.

```go
// name returns the display label configured for this provider.
func (p Provider) name() string { return p.label }
```

Calling-rule tier: Read returns shared cached bytes and ErrNotExist for a missing file. The copying obligation belongs at the API.

```go
// Read returns the cached contents of path, or ErrNotExist when the file is missing.
// The returned bytes are shared. Callers must copy them before modifying them.
```

## Design and paragraph order

Design tier: Acquire owns queue admission and cancellation, returns an idempotent release function, and uses zero to disable the limit. The paragraphs separate the normal result, cancellation race, and cleanup.

```go
// Acquire waits for a slot in the account queue and returns a release function
// after the request reaches the front. A zero limit lets the request proceed
// immediately without occupying a slot.
//
// Cancellation removes a waiting request from the queue. If cancellation races
// with a grant, Acquire returns the unused slot before waking another waiter.
//
// Callers invoke release after the vendor response has been read or closed.
// Repeated release calls have no further effect on the queue or occupied slots.
```

## Fields and shared rules

Image is a type definition; its fields identify a MIME type and base64 encoding. The separate Fields example assumes a sink reads asynchronously: verify that ownership rule and name the sink if it is consumer-specific.

```go
// Image is the encoded image sent in an upstream request.
type Image struct {
    MediaType string // image MIME type
    Data      string // base64
}
```

```go
// Fields holds event attributes. After enqueueing the event, callers must leave
// this map and its mutable values unchanged until the sink finishes reading them.
Fields map[string]any
```

Shared-rule tier: both durations start at request arrival and use zero for an unobserved event. Explain the shared rule once; each field names its endpoint.

```go
// FirstContent and FirstText measure elapsed time from request arrival to their
// respective events. Zero means the event has not been observed for this request.
FirstContent time.Duration // first text, reasoning, or tool-call content
FirstText    time.Duration // first text content
```

Independent field-rule tier: the Ops sink prefers a nonempty string attribute, then Component, then "app". Verify conversions and whitespace handling at that consumer.

```go
// Component names the source. The Ops sink prefers a nonempty Fields["component"]
// string and uses "app" when both sources are empty.
Component string
```

## Constants and local decisions

Assume the sender counts dispatches and the receiver counts arrivals; a new connection delays the first arrival. This explains the margin, without promising it covers every possible delay.

```go
const (
    // ArrivalSlack accounts for connection setup delaying the first arrival.
    // Counting dispatch times alone can admit another request inside the receiver's window.
    ArrivalSlack = time.Second
)
```

Assume cancellation arrived after the waiter received its slot. This required local explanation identifies why returning the slot precedes waking another waiter.

```go
// Cancellation arrived after this waiter received a slot. Return its unused slot
// before waking the next request in the same account queue.
lane.busy--
lane.grant()
```

Short-operation tier: the recorder already increased its count and now caps interval storage alone. An additional coordination or recovery mechanism would require the longer tier.

```go
// Keep counting after interval storage reaches its cap.
if len(intervals) < maxIntervals {
    intervals = append(intervals, next)
}
```

Assume processes lock an opened file and release the OS lock by closing its descriptor. Keeping the path prevents a future cleanup change from splitting holders across different files.

```go
// Keep the lock file: removing it lets another process create and lock a new
// file while an existing holder still has the old one open.
file.Close()
```

## Defaults and wrappers

The constructor owns the numeric default; the field identifies its role. Ordinary default handling needs no local paraphrase.

```go
// NewCache constructs an empty cache using ttl as the lifetime of each stored
// entry. Nonpositive durations select thirty seconds. Expiration is checked
// during cache reads.
func NewCache(ttl time.Duration) *Cache {
    if ttl <= 0 {
        ttl = 30 * time.Second
    }
    return &Cache{ttl: ttl}
}
```

```go
ttl time.Duration // stored entry lifetime
```

Calling-rule tier: an optional flusher changes the caller-visible effect. Branch count does not determine length; the nil check itself needs no narration.

```go
// flush sends pending output through the writer's optional flusher.
// Writers without a flusher keep their existing buffering behavior, so callers
// can invoke this helper for either kind of writer.
func (w *writer) flush() {
    if w.flusher != nil {
        w.flusher.Flush()
    }
}
```

Calling-rule tier: Borrow delegates synchronization to a library pool and replaces unusable entries. The borrowing contract belongs at the methods; the file can explain why storage is reused. A Get call alone needs no synchronization tutorial.

```go
// Borrow returns a reusable buffer whose contents may come from an earlier call.
// Return it with Release after its consumer finishes, then stop accessing its
// storage. An unusable pool entry produces a fresh buffer.
```

## Tests and unusual setup

Assume a delayed first arrival tests the receiver's rolling window. The entry explains the scenario and observable behavior; the setup note explains the injected delay. Routine setup stays uncommented.

```go
// TestArrivalWindow models a slow first connection followed by reused connections.
// The vendor must still receive no more than its limit in a rolling window,
// even when the first request arrives later than its dispatch time suggests.
```

```go
// Delay the first arrival to model connection setup while the later requests
// reuse an established connection and reach the receiver sooner.
vendor.firstDelay = 150 * time.Millisecond
```

## Guarantees and exceptions

Calling-rule tier: Write delegates replacement/recovery. Ordinary files are atomically replaced; hard-linked files are written in place after preserving the new bytes. The implementation component owns the design explanation; callers still need this exception.

```go
// Write replaces path through a temporary file and rename. Hard-linked files
// are written in place to preserve their links. That path can leave a partial
// write after a crash, with the complete new contents retained in a temporary file.
```

A claim of an independent copy fails when FinishedAt remains shared. Assume CopyRecord delegates collection copying and has calling-rule obligations; describe the ownership precisely:

```go
// CopyRecord copies the record's slices and timings map so callers can modify
// those collections independently. FinishedAt remains shared; callers must treat
// the pointed-to time as immutable.
```

## Review failures

Review the whole explanation before its counts. The reader should understand the problem and the decisions to preserve; required details should support that explanation.

| Failure | Correction |
| --- | --- |
| "Timings use unions. Context helpers are optional. The collector has a mutex. Headers have a limit." | Start with the filename and its responsibility, then explain one design question through its consequences, as in the elapsed-work overview. |
| "Stop before the next dependency exceeds the header limit." | Explain what remains available after truncation; translating the comparison adds no fact. |
| The Acquire example flattened into one paragraph | Keep normal results, cancellation, and cleanup as separate reader questions. Word count alone cannot make the paragraph usable. |
| A long Component description merely calling it the event's source | Document the consumer's override order, as in the field example. |
| "CopyRecord returns an independent copy of all its data." | Qualify shared references as in the ownership example. Testing a copied slice says nothing about a retained pointer. |
| Repeating the borrowing contract at the overview, type, methods, and Get call | Give each location its own role. Keep caller obligations at public methods and local reasons at exceptional operations. |
| Overview lists Refresh and Snapshot's individual jobs | State the file's overall responsibility first, then explain availability during replacement; let Snapshot document its return value, empty case, and shared ownership. |
| Overview begins with background, a domain label, or `This file` | Start the first complete sentence with the actual filename and a concrete responsibility. A filename heading alone is insufficient. |
| An uncommented private getter or a "state fields" label | Restore required declaration coverage or the actual shared rule and individual meanings. |
| Padding, line wrapping, or praise used to reach a minimum | Count substantive prose only; recheck role and facts. Preserve design-overview paragraphs. |
| "Preserves concrete types" after checking only ordinary pointers | Verify named pointers and interface cases before promising type preservation. |
| "Requires an explicit port" when only supplied ports are checked | State the actual condition. |
| Overview moved into a declaration block | Restore its required position after formatting. |

For the truncation case above, assume fixed metrics and a dependency total were emitted before sorted detail entries:

```go
// Fixed metrics and the dependency total remain; this dependency and later
// details are omitted when the next entry would exceed the header limit.
if len(candidate) > maxHeaderLength {
    break
}
```
