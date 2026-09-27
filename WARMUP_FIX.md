# Verified quota warmup

This fork fixes warmup success being reported before a quota countdown starts.
The upstream implementation accepted HTTP 200, read one stream chunk, ignored
stream errors, and closed the response before completion.

Warmup now waits for `response.completed` with completed status. Failed,
incomplete, malformed, or prematurely closed streams return an error. This also
applies to authentication/model retries and additional quota pools.

After completion, warmup fetches fresh usage twice at least six seconds apart.
The reset deadline must remain fixed within one second and remain in the future.
It checks the five-hour window when present, otherwise the weekly window, and
applicable additional model quota pools. Zero percent usage is allowed because
a small request can round down to zero while still starting a countdown.

Verification allows three intervals for usage propagation. If no countdown can
be confirmed, the command reports that uncertainty instead of claiming success.
Warmup does not replenish quota or reset an existing countdown. Expired or
revoked login sessions still require `codex-switch login <alias>`.

Validation: 42 warmup-related tests passed, including HTTP 200 error streams,
premature stream closure, moving deadlines, zero-rounded usage, weekly-only
accounts, and additional quota pools. Live tests on two previously inactive
profiles verified fixed reset deadlines and decreasing time remaining.
