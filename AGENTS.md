# Fix Execution Engine — Build Rules

These rules apply to every agent working on this project.

## The job

Build Personal Space exactly as specified in [REQUIREMENTS.md](./REQUIREMENTS.md). That document
is the contract: its phases, success criteria and final criteria decide when work is done. When in
doubt, REQUIREMENTS.md wins.

## The team

- **orchestrator** (primary) — plans, delegates, reviews, gates phases. Does not write code.
- **frontend-dev** — all frontend code and frontend unit tests.
- **backend-dev** — all backend code, storage, seed data and backend unit tests.
- **qa** — end-to-end tests, test runs, screenshots, DEFECTS.md. Does not fix code.
<!-- - **adversary** — tries to break the running app; records findings in ADVERSARIAL_REVIEW.md. -->
<!-- TODO: Add in a security-expert role -->

Role boundaries are enforced by permissions and are absolute. Do not work around them with shell
commands: if the edit tool would deny a file, do not modify that file any other way.

## Repository conventions

- Products/venues are self-contained under `market-data/<product>/<venue>/` — code, e2e,
  screenshots, security notes and runtime data all live inside that directory (e.g. the ASX
  drop-copy stack is `market-data/bookings/asx/` with its own `exec-client`, `exec-server`,
  `e2e/`, `screenshots/`, `security/`, `data/`).
- End-to-end tests, and their configuration, live under each venue's `e2e/`. Only qa writes there.
- Screenshots live under each venue's `screenshots/`.
- No emojis in code, comments, print statements or logging. (Emoji page icons in the product's
  data and UI are a feature, not a violation.)
- Keep it simple: small modules, clear names, no defensive programming, no overengineering.
  Prefer popular, well-supported libraries over custom code.

## DEFECTS.md — the defect ledger

All defects live in `DEFECTS.md` at the repo root, one entry per defect, newest first.
Writers: **qa** (create, close, reopen) and **orchestrator** (record developer responses,
reject). Nobody else edits it, ever.

Format, exactly:

    ## DEF-001: Short title

    - Status: OPEN
    - Severity: HIGH | MEDIUM | LOW
    - Found by: qa | adversary (ADV-003)
    - Phase: 3

    Steps to reproduce:
    1. Numbered, specific, starting from app launch.

    Expected: What should happen.
    Actual: What happens instead.
    Screenshot: screenshots/def-001.png (optional)

    History:
    - qa: opened

Statuses and who may set them:

| Status | Meaning | Set by |
|---|---|---|
| OPEN | Filed, or reopened after a failed retest or a bounced dispute | qa |
| FIX-READY | A developer reports a fix is in | orchestrator, relaying the developer |
| DISPUTED | A developer reports CANNOT REPRODUCE or WORKING AS INTENDED, with a reason | orchestrator, relaying the developer verbatim |
| CLOSED | qa retested and confirmed the fix, or accepted the dispute | qa only |
| REJECTED | Will not fix, with a written reason | orchestrator only |

Every status change appends a History line saying who, what and why. A defect is never done
because a developer says so — it is done when qa closes it.

## SECURITY.md - the security tracking ledger

All security issue live in `SECURITY.md` at the repo root, one entry per issue, newest first.  
Writers **security-adviser** (create, close, reopen) and nd **orchestrator** (record developer responses,
reject). Nobody else edits it, ever.

Format, exactly:

    ## SEC-001: Short title

    - Status: OPEN
    - Severity: HIGH | MEDIUM | LOW
    - Found by: security-adviser (ADV-003)
    - Phase: 3

    Steps to attack:
    1. Numbered, specific, starting from app launch.

    Vulnerability: What is the vulnerability.
    Actual: What happens instead.
  
    History:
    - security-adviser: opened

## ADVERSARIAL_REVIEW.md — the adversary's findings

All adversary findings live in `ADVERSARIAL_REVIEW.md` at the repo root.
Writers: **adversary** (create entries) and **orchestrator** (fill Disposition). Nobody else.

Format, exactly:

    ## ADV-001: Short title

    - Session: phase-3 gate | final
    - Suggested severity: HIGH | MEDIUM | LOW

    What I did: ...
    Expected: ...
    Actual: ...
    Screenshot: screenshots/adv-001.png (optional)

    Disposition: PENDING

The orchestrator replaces PENDING with either `ACCEPTED -> DEF-NNN` or `REJECTED - reason`.
Accepted findings are reproduced and filed in DEFECTS.md by qa. No entry may remain PENDING when
the final phase completes.

<!-- graft:start -->
## Graft — repo context graph

This repo is indexed in `graft/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `graft map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `graft ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `graft grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `graft skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `graft callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `graft/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `graft ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `graft build` (deterministic,
no API key, $0).
<!-- graft:end -->
