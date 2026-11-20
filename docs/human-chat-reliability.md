# Human chat reliability contract

This prerequisite keeps the shared chat provider-free when Konti AI is not
available and refreshes an open foreground feed by bounded polling.

## Exact bounds

- foreground feed interval: 4 seconds;
- at most one active feed request per mounted chat view; initial load, polling,
  reconnect/send refresh and history pagination share the same coordinator;
- one coalesced refresh and one user-requested history operation may wait, and
  execute serially, without either operation replacing the other;
- human Message JSON body: 65,536 bytes maximum, including the envelope;
- human Message text: 20,000 UTF-16 code units maximum (the existing schema
  contract);
- empty and whitespace-only text-only Messages are rejected;
- feed and error responses retain `Cache-Control: no-store`.

The 65,536-byte bound applies to the complete encoded JSON request body. It
does not promise that every possible JSON escaping of a 20,000 UTF-16-unit
string fits under that wire bound; clients must satisfy both independent
limits.

Every feed page carries an opaque snapshot derived from the fresh session,
authVersion, active mode, active roles, participant identity,
membershipGeneration and conversation scope that authorized its source set.
The route validates all these dimensions in one PostgreSQL statement snapshot
following the Message query, and emits no feed or scope payload if they differ.
This statement is the read's linearization boundary: revocations committed before
it are observed together; a concurrent revocation committed after the snapshot
is ordered after this read. It does not promise to recall already-issued bytes.
No locks or transaction span pagination, network waits, or serialization. A revoke/re-add changes the
membership generation and therefore cannot relabel an older result.

Foreground refresh uses a version-1 HMAC-authenticated per-conversation sequence
watermark, bound to the exact authority key and sorted authorized source set.
The signing key uses the existing AUTH_SECRET with a separate purpose prefix;
no new configuration is required. Decode is strict and bounded to 512 sources,
int32 nonnegative sequences, 191-character IDs and 128 KiB encoded input.
Tampering, unknown/duplicate sources, authority changes and future watermarks
fail closed with a generic cursor error. The existing transactionally allocated
sequence determines forward progress; timestamps only determine display order.
Initial watermarks are captured before the message query, so a concurrent commit
can be delivered again but cannot be skipped. Client merge deduplicates by ID. A refresh reads up to five 100-Message pages before yielding and queues
one continuation when more data remains. A partial network failure preserves
the pages already received and the next interval resumes from the last proven
cursor. History pagination uses its independent older cursor.

Polling and authority-probe network failures preserve the last authorized feed
until the existing 15-second authority hard deadline. At the deadline the UI
quarantines the feed and renders none of the retained snapshot. Recovery first
requires a fresh, bit-identical authority snapshot and then an incremental
refresh; a changed authority destroys the retained snapshot. A fresh 401/403
or opaque authority-epoch mismatch quarantines immediately. Returning to a
visible/focused tab and the browser `online` event request immediate authority
revalidation and refresh. Before that asynchronous probe, each return signal
checks wall-clock expiry synchronously and flushes quarantine to the DOM even
if browser background timers have not run. Polling never starts an AI provider call.
Client unread is max(0, latest allocated sequence - acknowledged sequence);
stale polls and read acknowledgements cannot move either cursor backwards.

When AI flags are unavailable, leading `@Konti` text is sent as an ordinary
OPERATIONS_DAY HUMAN Message and the UI does not label it as an AI request.

## Deliberately deferred rate limit

`src/lib/rateLimit.ts` is explicitly process-memory based. It is correct only
for the repository's documented single-instance deployment and would multiply
the effective quota across application instances or reset it on restart. This
PR therefore does not claim a distributed human-chat abuse limit and does not
add Redis/Key Value infrastructure. A shared server-side limiter keyed by
`userId + conversationId`, with deterministic 429 and `Retry-After`, remains a
blocking follow-up before the full production pilot.
