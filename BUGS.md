# Deliberate bugs — answer key (not for the AT suite to read)

This mock is intentionally built with two real bugs in the *implementation
logic* (not in USER_STORIES.md, not in the test cases). The finalized test
cases were written as if the site were correct; these bugs are what the
Playwright AT suite is supposed to catch and the QA notice reporter is
supposed to flag.

Keep this file out of the selector-discovery pass and out of the AT
suite's own references — treat it the way a real reviewer would never get
to see the dev's private notes before testing.

## Bug 1 — lockout never expires

`isLocked()` in `server.js` only checks whether `session.lockedUntil` has
ever been set; it never compares it against `Date.now()`. So once a
session hits the 3-try lockout, it is locked forever, not for 15 seconds
as DECISIONS.md #8 specifies.

- **Caught by:** TC-018 (lockout expires after timeout window) — will
  fail; the retry with the correct password after waiting 15s still gets
  bounced to `/login-failed`.
- **Not caught by:** TC-017 (lockout persists across reload) — that one
  still passes, since the bug doesn't affect persistence, only expiry.

## Bug 2 — Confirm is never actually disabled, and no-selection isn't guarded

DECISIONS.md #10 says Confirm should be disabled/non-interactive until a
slot is selected. `#confirm-btn` in `renderBooking()` has no `disabled`
attribute logic at all — it's always clickable. The `/confirm` handler
also has no explicit "nothing selected" guard; it relies on
`SLOTS.find(...)` returning `undefined` and silently re-renders the page
with no message.

- **Caught by:** TC-015 (confirm with nothing selected) — the expected
  result says Confirm should be disabled/non-interactive; a selector check
  for a `disabled` attribute on `#confirm-btn` will fail.
- **Side effect:** clicking Confirm with nothing selected is at least
  harmless (no booking is created, no crash) — so if TC-015's assertion
  is written loosely (just "no booking created"), it could pass by
  accident and miss the real bug. Worth checking the assertion actually
  targets the disabled state, not just the absence of a booking.
