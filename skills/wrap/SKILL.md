# description: Wrap up a work session — update CHANGELOG, BACKLOG, DEPLOYMENTS, and decisions log from what happened this session

Copy into `.claude/skills/wrap/SKILL.md`. Run this at the **end of a session**. It captures what
was built, fixed, and deployed into the repo's change-management files so nothing is lost between
sessions — this is the core habit that makes a project's history queryable by Claude (and by you)
without re-reading the whole codebase or git log every time.

Pairs with the four templates in `../../change-management/`: copy those to your project root (and
`docs/decisions.md`) once, then let `/wrap` maintain them going forward.

## What this session changed (gather evidence first)

```bash
git status --short
git diff --stat
git log --oneline -10
```

Also review the conversation for: bugs diagnosed (and their **root cause**), features added,
decisions made, and anything deployed or restarted.

## Update the files

Edit these directly (don't just describe — actually write the entries):

1. **`CHANGELOG.md`** — add/extend today's dated section (`## YYYY-MM-DD`, newest first) with:
   - `### Added` — new features/capabilities.
   - `### Fixed` — bugs, each with a **one-line root cause** (this is the debugging knowledge base — be specific: what was wrong, why, how fixed).
   - `### Changed` — behaviour/rename/refactor changes.
   - `### Deployed` — which environment(s) received it; explicitly note "local only, not pushed" when true.

2. **`BACKLOG.md`** — remove items finished this session; add newly-discovered TODOs/follow-ons. Keep it pending-only.

3. **`DEPLOYMENTS.md`** — add a row **only if something was deployed/restarted** (date · env · commit/ref · what · verified?).

4. **`docs/decisions.md`** — add an entry **only if a non-obvious architectural/process decision was made** (context → decision → why).

5. **Claude's persistent memory** (if your setup has one) — if a durable cross-session fact, convention, or pitfall emerged, save/update it there too.

## Confirm

```bash
git status --short   # show the doc files now have changes
```

Report a 3-line summary: what was logged to CHANGELOG, what moved in BACKLOG, whether a deployment
row was added. Ask whether to commit the doc updates (and, if the user wants, the session's code
changes) — don't push unless asked.

## Notes
- This is **bookkeeping, not deploying.** Never push to production from `/wrap`.
- Keep entries terse and factual. Root causes > narration.
- If nothing shipped and nothing broke, it's fine to only touch BACKLOG.

## Why this pattern is worth adopting
Four files, one convention each, checked into the repo (not a wiki, not tribal knowledge):
- **CHANGELOG.md** = what shipped + why bugs happened (a searchable debugging knowledge base over time).
- **BACKLOG.md** = the pending queue only — never grows unbounded because finished items are removed, not just checked off.
- **DEPLOYMENTS.md** = what's running where, since when — answers "is prod already this fix?" in one glance.
- **docs/decisions.md** = lightweight ADRs — the *why* behind non-obvious calls, so nobody re-litigates them a month later.
Point your project's CLAUDE.md (or equivalent) at these files so every session starts by reading BACKLOG + the latest CHANGELOG entry, and ends by running `/wrap`.
