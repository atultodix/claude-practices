# Working Rules

Always-on operating rules. Imported into every Claude Code session by a POINTER in `~/.claude/CLAUDE.md` → this checkout (`~/projects/claude-practices`). Refresh: `git -C ~/projects/claude-practices pull`. The pointer file carries no rules of its own.
**Ceiling: 200 lines.** To add, remove. Full method and reasoning: `practices.md`.
Each rule carries the incident that produced it — that makes it defensible, and prunable.

---

## 1 · Before building

**Read the code, not the notes.** Planning docs go stale. Before scoping anything, read the actual file, the actual issue, the actual database state. Journals summarise; code is truth.
*(Repeated scoping errors from trusting a document over the repo.)*

**STOP on a shaky assumption.** When a spec rests on a mechanism that might not hold, verify it and stop with findings before building past it. The verification regularly overturns the assumption — and a ruling beats reworking a wrong build.

**If you can't open the artifact, stop and say so.** A dispatch naming a prototype, spreadsheet or migration you cannot read is blocked, not a licence to invent. Report what's missing.
*(A design handoff sat uncommitted on the operator's machine; the flight would have invented nine surfaces.)*

**Reuse; don't reinvent.** A module consumes the existing graph — existing goals, existing work taxonomy, existing org. A module that maintains its own version of what already exists is a spreadsheet with better styling.

## 2 · Databases and destructive operations

**Guarded operator scripts.** Anything touching a real database:
- prints its DB host/target as the **first line** and **aborts unless it matches the expected project ref** — check the whole connection string; under poolers the ref may sit in the username, not the host;
- names its target workspace/tenant explicitly;
- is **idempotent** — pre-cleans its own sentinel artifacts.

**Plan by default; write only on an explicit flag.** Scripts that touch real tenants run read-only and write only with `APPLY=1` (or equivalent). The human reviews the plan output first. **The plan output is a deliverable**: totals, refusals with reasons, conflicts, per-field counts.

**Never overwrite silently.** Filling an absent value and correcting a present one are **different verbs with different bars**. Correction requires a reason, keeps the old value visible, and is audited. Never relax the fill verb to allow edits.

**Destructive actions outside scripts too — list, identify, then act.** Deleting branches, revoking tokens, removing files, dropping access: **print what will be destroyed and confirm the target account/tenant/repo before executing.** The listing is the plan step; run it as its own command and read it.
*(Every token on the wrong account was revoked before anyone identified which account owned the leaked one.)*

**Refuse loudly; never guess.** Ambiguous input (a two-digit year, an unparseable date, a duplicate key) is refused and listed with its row and reason. A guessed value is worse than an absent one.

**Migrations:** create with `--create-only`; prod migrations apply via the deploy pipeline on push — never manually. **Regenerate the generated client after a schema change lands** — a stale client is the most common silent local breakage.

**Environment:** `set -a; source <env file>; set +a`, then echo the resolved target before running anything.

**Smoke scripts clean up after themselves** using sentinel labels that target only their own artifacts. Leave audit history intact — never delete audit rows to be tidy.

## 3 · Verifying

**Built ≠ verified.** Nothing is done until something exercised it end to end in the real environment — real client, real database, real routes. Unit tests on fakes prove logic, not integration.

**Cheapest first, in gated stages:** static checks → unit suites → real-environment roundtrip → the surface in a browser. Each stage gates the next.

**Only loading the page proves the page.** Walk any new or changed user-facing surface in a real browser against the deployed build: load it, do the thing it exists for, reload to confirm persistence, look at what rendered.
*(Every route of a module 500'd on production for a day with 234 unit suites green — client-module exports called from server pages. A table's first row was invisible under a sticky header whose offset came from a prototype's shell.)*

**Clear the dev build cache after a branch switch, before any dev-server walk.** A stale `.next` (or equivalent) can serve **bodyless 404s for an entire route subtree** — handlers never run, writes vanish silently, and it reads exactly like a code defect. Production builds fresh per deploy and cannot exhibit it, so the bug is un-reproducible where it matters. **The tell:** a 404 with **no error body** means the framework never reached your handler — your own error paths always return one. Routing failure, not code failure.
*(Twice: a route that "didn't exist", and a ritual whose decisions silently failed to persist — the second nearly blamed on a merge that was fine.)*

**Wait for the deploy before walking.** ~5 minutes from push to live. A walk against a stale build produces a false finding. If a walk contradicts a report, establish which build is live before concluding the fix failed.

**Read a CI failure before believing it.** *"Canceling since a higher priority waiting request exists"* = superseded by a later push. *"Exceeded the maximum execution time"* = the suite outgrew the limit. A run that dies in under ~90s failed at setup, not in the tests.
*(A real 3-assertion failure hid behind a timeout for hours because everyone read the red as "the timeout again.")*

**Enumerate; never count.** When a spec defines N surfaces or N criteria, list each by name against its artifact — *"S1 → /admin/exit-cases ✓ · Overview → not built ✗"*. "All green" hides gaps that an enumerated list cannot.

**Say when you verified less than you claim.** "Reviewed from the report only, artifact unread." "Not walked — browser unavailable." Silence implies the full check happened.

## 4 · Terminal, git, reports

**Every command block starts with an explicit `cd`** into the correct repo or worktree, plus `pwd` (and a branch check for git operations). Commands are code blocks, never prose.

**Long output goes to disk, not chat.** End-of-phase and completion reports are written to a file in the worktree root (`PHASE_REPORT.md` / `BUILD_REPORT.md`); the chat reply is a pointer. Chat pastes can silently arrive empty — **if a message references content whose body is missing, say so in that turn**; never quietly compensate.

**Evidence cited is evidence committed.** A report may only reference screenshots or artifacts that are themselves committed; scratchpad paths rot with the session, and a report citing them loses its proof.

**Multi-account machines:** prefix every `gh` or git-over-https block with an explicit account switch — other projects' terminals silently flip the active account. **Never put a token inside a remote URL** — it bypasses every credential helper and its expiry breaks all lanes at once.

**Branch hygiene:** prune merged branches when they exceed ~20, during the board pass. Never prune a branch absent from `--merged origin/main`; git refuses branches a worktree holds, and that refusal is the safety net.

**Local merge by default; a draft PR when the change is infrastructural** — CI, auth, migrations, the workflow itself — so checks run *before* the merge rather than after.
*(A CI change merged straight to main would have broken the gate with no warning.)*

## 5 · What the product must never do

**No fake zeros.** Absent data is "pending" or blank — never a fabricated number. A computed metric with no inputs says so.

**Absence is omission.** A sparse record says *less*; it never shows an em-dash, an "N/A", or an internal explanation. **Internal phrasing must never reach a user-facing document** — pin it with a test that fails if it does.

**Issued artifacts are immutable; corrections are new versions.** Anything a user received is never rewritten. A correction supersedes: new version active, old preserved and marked, chain linked, single-active enforced at the database. Consent state starts clean on the new version.

**Declared is never dressed as observed.** Output over declared inputs says so on its face, and ranks below observed evidence in any scoring. One declaration is never counted as two sources.

**Separate math from display.** Computations use the entity's own identity; display labels are a separate contract.

**Propose, then confirm.** Extraction, matching and inference **propose**; a human confirms. Nothing auto-creates a record, and a human's dismissal is authoritative — recompute may refresh evidence, never a status a person set.

**Audience separation is sacred.** Where data must not cross an audience boundary (compensation into employee views, one tenant into another), verify with deep key-scans, not spot checks. Scope writes by tenant; fail closed when the scope is absent.

## 6 · Working with the human

**Plan → show → approve → then act.** No dispatch, no repo write, no tenant write before the human sees the plan.

**Multiple questions → one interactive prompt**, not a wall of prose. If a widget can't carry it, number the items and give a one-line answer key. *In-chat and answerable beats thorough and unanswered.*

**Honest findings over comfort.** Name the gap, the developer-grade surface, the thing that isn't verified. When asked "any contradictions?", lead with the contradiction, not reassurance.

**Never approve from a summary when the artifact is available.** A report written by the thing that produced the work is a claim, not evidence.

**End neutrally:** what's done, what's next. Never suggest when to stop — the human says when.

**Scope hygiene:** separate projects are separate scopes. Never cross-reference their activity, decisions or architecture unless the human invokes the connection.
