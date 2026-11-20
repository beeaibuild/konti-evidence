# Global daily-chat writing policy

The MASTER_ADMIN system-settings page exposes **Közös chat írása aktív fuvar nélkül is**.
The persisted singleton defaults to off. An absent row also means off. No environment
variable, browser storage or login token carries this setting.

All active internal DRIVER and OPERATOR users (excluding CUSTOMER grants) receive
membership in today's canonical Budapest OPERATIONS_DAY conversation. This read
membership is independent of work, DispatchRun assignment and the write policy.
The existing historical membership and PLANNING/private-conversation boundaries
are unchanged; no historical memberships are backfilled for newly enrolled readers.

Off preserves the existing daily roster/Job/TransportTask/operator write grant,
including its existing within-day sticky behavior after roster removal. The new
`operationsWriteEligible` participant bit separates that grant from the new
read-only enrollment. Migration preserves grants for existing active daily HUMAN
memberships; future read-only memberships default false. On allows active internal
drivers, including dual-role users in DRIVER mode, to write today without that grant.
Historical days remain read-only and DRIVER still cannot enter PLANNING.

Text, replies and private-image uploads share the server's daily write decision.
The image route performs an early authority check and repeats it in the committing
transaction, before any storage operation. Existing media, CSRF and limiter rules
remain in force. The setting uses a shared/exclusive PostgreSQL advisory transaction
lock: a completed off update cannot be bypassed by a later writing transaction.
In-flight writes holding the shared lock finish before an exclusive update commits.
Lock order: session/user → conversation/participant → policy (writes), and
session/user → policy → audit (settings). Settings never acquire conversation locks.

Only canonical MASTER_ADMIN authority in an eligible internal mode may update the
setting. Session and user rows are locked and current roles are reread before the
singleton mutation. The same transaction records `CHAT_WRITE_POLICY_CHANGED`, actor,
database timestamp, and `{before, after}`. No-op changes do not add audit events.

Feed sources carry the current write decision, without a policy cache. Existing
focus/feed refresh updates text, camera, gallery and send controls without login.
Reply buttons use the current source decision rather than a message's old snapshot.
An already-open stale composer never overrides the next server-side authorization.

## Verification

- Unit: policy defaults, roles, stale authority, audit payload; bounded/CSRF-protected
  route; settings-component success/failure.
- Real PostgreSQL 16: Sunday with no work, first-open visibility, reply, immediate
  disable, private/historical/inactive denials, image API and attachment persistence,
  singleton/default, concurrent updates, audit failure rollback.
- Playwright: 390×844 and 412×915, real login/feed/settings/text APIs, no relogin
  transitions, camera/gallery filechooser identity, preview, controlled upload failure,
  backend rejection after disable, layout and non-admin UI/API denial.
- Media storage in PostgreSQL tests is an in-memory observable storage boundary;
  PostgreSQL itself and all production authorization/message handlers are real.
  Browser upload failure is intercepted to prevent any external storage call.
  These tests do not prove physical device-camera integration or a provider deployment.

This branch starts at target `91be72cbb0b18714607394d246a12b4ebc1dd314` and contains no
PR #82 harness code. No deployment or shared database changes are part of this PR.
The reported staging read-only screen was not inspected live: membership and
dual-role restrictions are established from the exact target code, not inferred
as a proven diagnosis of a particular staging session.
