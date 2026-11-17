# Verification rules (fail closed)

These rules decide when an agent may say a listing is removed. They are derived
from the source app's verifier and status model. When in doubt, the answer is
"not verified yet", never "removed".

## 1. What counts as a removal

A listing is **verified removed** only when all of these hold:

1. You loaded a genuine page on the broker's own domain.
2. That page positively shows the specific record is gone (for example, the
   saved listing link now returns the broker's own "not found" page, or the
   broker's own results no longer contain that record).
3. You saved evidence of that page (a screenshot or saved copy, kept locally)
   with the date.

No evidence saved means no removal claim. Record the check as an error and retry.

## 2. Pages that are never a removal

When you are **checking whether a listing is gone**, treat each of these as
**inconclusive** (status `error_retry`, retry in about a day):

- A CAPTCHA, bot check, "verify you are human", or browser challenge page.
- A rate-limit, access-denied, login, paywall, or survey/upsell page.
- An HTTP error page, a blank page, or a page that did not finish loading.
- An empty result on a page you cannot confirm is the broker's real result page.
- Any page that is off the broker's domain (a redirect to another site, an ad
  landing page, a directory or home page instead of the record).
- A consent or cookie dialog covering the content.

An empty result is only trustworthy on a genuine, unblocked broker page.

The same pages can show up **before a request is sent**. Then nothing was
sent. Record `needs_user_action`, tell the user that nothing was sent, and
say what they need to do (see the Loop in `SKILL.md`).

## 3. A submitted request is not a removal

- A "request received" page, an acknowledgement email, a confirmation-link
  click, or a later "your opt-out is done" email proves **submission**, not
  removal.
- If the broker asks for an email confirmation and it has not been completed,
  the status is "waiting for email confirmation", not "submitted".
- Do not mark a request as submitted unless you saw the broker's real final
  confirmation or its accepted email-confirmation state.
- Suppression hides a listing; it does not prove stored data was deleted.

## 4. Record-boundary rule

One opt-out covers one record. When the broker gives each record its own
link, that link is the record's identity.

- Verify the same record you opted out, by its saved link.
- A different record with a similar name (a different link) is a **separate
  listing to review with the user**. It is not proof the opt-out failed, and
  its absence is not proof the opt-out worked.
- A parent-company request does not prove a sister site is clear. Check each
  covered site on its own domain before saying it is covered.

## 5. Under-match caution

Reading fewer records than the page visibly shows means your reading is
incomplete. If the page reports more results or more pages than you examined,
or the only record you read does not correspond to the saved record, the check
is **inconclusive**. Never turn a partial read into "removed".

If the record is plainly still present, the status is "still listed".

## 6. Recheck schedule

- First check: after the broker's processing window (`timing.first_check_days`
  in each broker file; default 14 days after the request).
- Then at day 30 and day 90 after the request
  (default ladder: 14, 30, 90).
- After that, repeat every `timing.ongoing_recheck_days` days. Listings can
  come back; a reappearance means a new request, not an old success.
- An inconclusive check is retried after about 24 hours and does not advance
  the ladder.
- Brokers with `verification.automated_recheck_unavailable: true` block
  evidence-bearing automated checks. Ask the user to look and record what they
  saw.

## 7. Statuses to record

- `needs_user_action` — nothing more can happen until the user acts
  (giving the details the form needs, a go-ahead for terms, a bot check, a
  lookup on the broker's form). Always say whether anything was sent yet.
- `waiting_for_email_confirmation` — the form was sent, and the broker's
  emailed link has not been clicked yet.
- `submitted` — the broker's real final confirmation was seen, or its
  emailed link was clicked.
- `verification_pending` — submitted, and the first check is not due yet.
- `still_listed` — a genuine check showed the record is still there.
- `removal_verified` — a genuine check met every point in section 1.
- `reappeared` — the record came back after `removal_verified`.
- `error_retry` — a check was inconclusive; retry in about a day.
- `declined` — the broker refused the request in writing. Tell the user
  what it said, and offer the next entry in `opt_out_routes` or, for
  California residents, the state platform. The listing is still there.

Only `removal_verified` may be reported as removed.
