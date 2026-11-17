---
name: data-broker-opt-out
description: Guides opt-out requests to data brokers and verifies removals with fail-closed evidence rules, keeping personal data on the user's machine. Use when a user asks to remove their own (or an authorizing person's) listing from a data broker.
---

# Data broker opt-out and verification

Status: public release.

This pack holds per-broker opt-out routes, requirements, timing, and
verification rules. It does not tell you how to locate a person. The user
supplies the listing links or confirms the listing themselves.

## Files

Read them in this order:

1. `verification-rules.md` — when you may say something is removed, and the
   statuses to use.
2. `brokers/<broker_id>.json` — one broker each. Each field is described in
   `schema/broker.schema.json`.
3. `clusters.json` — parent companies and the sister sites they may cover.
4. `manifest.json` — counts, source version, and generation time.

## Reading a broker file

- `null` means "no data", never "no".
- **How far to trust it.** `knowledge_verification.status: "unverified"`
  means the entry is a lead to confirm on the broker's own pages.
  `"verified"` means the source project reviewed it on `last_verified`.
  `knowledge_block_status` is the stricter mark: only the value
  `"verified"` means a person watched the live opt-out form on that date.
  Any other value, or a date more than about 90 days old, means go by what
  the live page shows.
- **Live page check.** `opt_out_url_check` is the newest observation of the
  opt-out address. `"blocked"` means a bot check or an access-denied page
  stood in front of the form on that date, so expect one. `"loaded"` only
  means the address answered; a browser may still be stopped.
- **Missing steps.** This pack leaves out any step that looks a person up.
  `playbook.omitted_step_count` above zero means such a step was dropped,
  so the steps may start part-way through. Ask the user to do the lookup on
  the broker's own form and to pick their own record. A step that says to
  collect a listing link is also the user's job.
- **Step timing.** A step may name a different wait than `timing`. An
  early look is fine, but do not call a request failed until the longer of
  the two waits has passed.
- **Deletion follow-up.** `deletion_lane` describes a second request that
  asks the broker to delete stored data, not just hide the listing. It does
  not change which route you start with. When `prefer_deletion` is true,
  offer the follow-up to the user after the opt-out is sent.
- **Sister sites.** `coverage.covers` lists sister sites by name. A few
  have a broker file in `brokers/`, so look; most do not. Never report them as cleared. Tell the user each one
  needs a check of its own.

## Hard rules

1. Act only for the user, or for a person who has authorized the user. If
   authorization is unclear, stop and ask.
2. Send personal data only to the broker being asked, on that broker's own
   domain or its listed contact address, and only the fields that broker
   requires. Never send it anywhere else, including logs, other tools, or
   third-party services.
3. Never solve or bypass a CAPTCHA, bot check, or browser challenge. Some
   checks clear on their own, so wait about 20 seconds. If it is still there,
   ask the user to clear it in their own browser, and wait.
4. Never claim a removal without evidence that meets `verification-rules.md`.
5. Record every submission and its recheck dates locally, on the user's
   machine.
6. Do not create accounts, pay, upload identity documents, or accept terms
   without the user's explicit go-ahead for that step.
7. Use only routes listed in the broker file or found on the broker's own
   privacy page. If a route's domain does not match `site_domains`, confirm
   with the user first.

## California residents: state platform first

California runs a free deletion platform called DROP
(https://privacy.ca.gov/drop). One request there goes to every data broker
registered with the state. Registered brokers have had to act on those
requests since 2026-08-01, at least every 45 days.

- Ask whether the user lives in California. If so, tell them about DROP
  before any single-broker request. The user files it themselves, and the
  platform checks residency on its own.
- DROP does not replace this pack. Not every site here is a registered
  broker, and a DROP request proves nothing about one listing. Listings the
  user wants gone sooner still need their own request and their own check.
- A DROP request is never proof of removal. Apply `verification-rules.md`
  to each listing as usual.
- DROP is only for California residents. For everyone else, go straight to
  the Loop.

## Loop

1. **Confirm scope.** Whose listing, and is the user authorized? Which brokers?
   Does the user live in California (see above)?
2. **Load the broker file.** If `status.state` is not `active`, tell the user
   why (`status.reason`) and stop for that broker. If
   `coverage.parent_broker_id` is set, work on the parent first.
3. **Check requirements.** Tell the user up front what the broker needs
   (`requirements`): email confirmation, date of birth, a likely CAPTCHA.
   If `opt_out_url_check.result` is `blocked`, say a bot check is likely.
4. **Pick the route.** Prefer the first entry in `opt_out_routes`. A route
   with a null `url` means the pack has no entry link; ask the user to open the
   broker's privacy page.
5. **Submit with the user.** Follow `playbook.steps` when present; otherwise
   follow the broker's own form. Pause for any CAPTCHA or email confirmation.
   **If a block stops you before anything is sent**, record
   `needs_user_action` and tell the user that nothing was sent. Ask them to
   clear the check. An access-denied page with nothing to clear is a block
   too: reload once, then treat it the same way. A check that keeps coming back in an automated or
   built-in browser often does not show in the user's everyday browser, so
   ask them to open the address there. If it still cannot be cleared, offer
   the next entry in `opt_out_routes` (often an email address) and go on
   only with their go-ahead.
6. **Record it locally.** Broker, record link, route used, date, what the
   final page showed, where the evidence is saved (a dated screenshot or
   saved page in a folder the user picks), and the first recheck date
   (`timing.first_check_days`). If the broker gives records no link of
   their own, note the details the user used to pick the record, and ask the
   user to do the later checks.
7. **Recheck on schedule.** Apply `verification-rules.md`. Blocked, empty,
   off-domain, or partial pages are inconclusive: retry later.
8. **Report honestly.** Say "nothing sent yet, needs your action",
   "submitted", "waiting", "still listed", or "removal verified" exactly as
   the evidence supports. For parents, each site in `coverage.covers` needs
   its own check; never report one as cleared from the parent's result.
9. **Keep watching.** Repeat checks every `timing.ongoing_recheck_days` days.
