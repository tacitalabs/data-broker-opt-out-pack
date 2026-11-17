# Infobreaker agent pack

**Status: public release.** Free for non-commercial use under
**CC BY-NC-SA 4.0** (see `LICENSE` and `ATTRIBUTION.md`).

Machine-readable knowledge about how to ask data brokers to remove a listing
and how to verify the result, for general AI agents (skills, MCP clients,
local-model agents). Personal data stays on the user's machine; this pack
contains none.

## Contents

- `SKILL.md` — agent instructions and hard rules.
- `brokers/` — 67 broker files.
- `clusters.json` — ownership clusters (parent company and covered sites).
- `verification-rules.md` — fail-closed rules for removal claims.
- `schema/broker.schema.json` — JSON Schema for a broker file.
- `manifest.json` — counts and provenance.
- `LICENSE`, `ATTRIBUTION.md` — licence text and credits.

## Deliberately left out

- **How to locate a person.** No lookup steps, query formats, or result-page
  parsing. Publishing those could help someone track down another person. The
  user brings their own listing links.
- **Page selectors and automation code.** They change often and are not needed
  to follow a broker's own form.
- **Any personal data, evidence, or run history.**

Text from the source project that read as lookup guidance was dropped at build
time (6 strings in this build).

## Accuracy

Each broker file carries `knowledge_verification`. A broker is marked verified
only when its live opt-out form was observed. Most are still unverified.

Steps, requirements, and confirmation signals for 43 brokers were written
from the source project's own automation and run notes. Each file lists where
its facts came from in `provenance`: `module_code` and `repo_doc` mean
"this is what worked in the source project", not "checked on the live site
today". `opt_out_url_check` records whether the opt-out page loaded when it
was last looked at.

`null` means the project has no data for that field. Nothing here is legal
advice.

## How it is generated

This pack is generated from the Infobreaker desktop app's broker data, which
is not public. A safety gate blocks any build that contains off-domain email
addresses, phone-like strings, street addresses, page selectors, or lookup
instructions. Do not edit the files here by hand; they are replaced on each
release.

## Corrections

Broker sites change often. If a route, step, or confirmation here is wrong,
open an issue that says what the broker's own page showed and on what date.
Do not include anyone's personal details in an issue.
