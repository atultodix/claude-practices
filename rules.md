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

**Migrations:** create with `--create-only`. **Prod migrations are applied by a guarded, plan-first operator script** (prints the DB/project ref, aborts unless it matches the expected ref; PLAN by default / `APPLY=1`; idempotent; reads back), run **only on explicit human OK** — never ad-hoc inline in a feature lane. **Do not assume a deploy auto-applies migrations** unless you have verified a migration runner is actually wired (a `next build`-only deploy does NOT). **Regenerate the generated client after a schema change lands** — a stale client is the most common silent local breakage.
*(2026-09-19: the prior "apply via the deploy pipeline on push" wording was false for the BRRD repo — no runner existed; migrations 0005/0006 had to be applied by the guarded script.)*

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

**A gate is a COMMAND AND AN EXIT CODE, never an adjective.** Report `npx tsc --noEmit → exit 0` and `npm test → 1504/1504, exit 0`, not "tsc clean" / "suite green" — an adjective is a claim about a run nobody can re-run. Assert **exit 0**; never interpret a non-zero value (tsc returns 1 or 2 for the same failing code). Four ways the number lies: a **pipe reports the pipe's** status (`tsc --noEmit | tail` exits 0 on a failing typecheck — use `set -o pipefail`); **two commands that both look like "the typecheck" disagree** (`next build` discards every diagnostic in a `*.test.*` / `*.spec.*` file); an **incremental cache replays a stale verdict in both directions** (pass `--incremental false` rather than remembering to clear it); and an **already-red gate hides the next red** — and everything behind the first failure is unmeasured, so say so.
*(Five mis-measuring gates in one week; a typecheck red on main for nine consecutive merges while every arc reported "tsc clean".)*

**An unread gate is an unbuilt gate — a red main is fixed BEFORE the next arc starts.** CI announcing a failure into a void is the same outcome as no CI, with a receipt. So the alarm goes where the work is *dispatched from* — the issue tracker — never to a channel built to be deleted; and the reading is a command with an exit code (`npm run ci:status`), never "check CI". **Three verdicts, not two:** a *cancelled* run measured NOTHING and must never read as green, and a gate that aborts mid-step leaves everything behind the first failure unmeasured — run the later checks anyway and publish both outcomes. Don't let the ref-cancelling concurrency rule cancel **main**: that discards the verdict on a commit nobody will check again.
*(Eleven merges landed on a red main over ten days. GitHub's failure email existed the whole time and went to the founder's own account; four of those merges also had an UNKNOWN unit result behind an aborting typecheck, and two main commits were never checked at all.)*

**Say when you verified less than you claim.** "Reviewed from the report only, artifact unread." "Not walked — browser unavailable." Silence implies the full check happened.

## 4 · Terminal, git, reports

**Every command block starts with an explicit `cd`** into the correct repo or worktree, plus `pwd` (and a branch check for git operations). Commands are code blocks, never prose.

**Long output goes to disk, not chat.** End-of-phase and completion reports are written to a file in the worktree root (`PHASE_REPORT.md` / `BUILD_REPORT.md`); the chat reply is a pointer. Chat pastes can silently arrive empty — **if a message references content whose body is missing, say so in that turn**; never quietly compensate.

**Evidence cited is evidence committed.** A report may only reference screenshots or artifacts that are themselves committed; scratchpad paths rot with the session, and a report citing them loses its proof.

**Multi-account machines:** prefix every `gh` or git-over-https block with an explicit account switch — other projects' terminals silently flip the active account. **Never put a token inside a remote URL** — it bypasses every credential helper and its expiry breaks all lanes at once.

**Branch hygiene:** prune merged branches when they exceed ~20, during the board pass. Never prune a branch absent from `--merged origin/main`; git refuses branches a worktree holds, and that refusal is the safety net.

**Scheduled workflows: at most 2x/day, once daily preferred — anything more needs the founder's explicit approval.** State the cadence's reason in the workflow file.
*(Two hourly smokes silently burned the entire monthly Actions quota; the failure email arrived before anyone knew the meter was running.)*

**CI triggers: `pull_request` + `push` to main only — never bare `push`; `concurrency` with `cancel-in-progress` always; heavy suites (browser e2e) on PR/main only, fast checks may run wider.** The same SHA must never bill twice.
*(A high-tempo week fired a five-job Playwright pipeline twice per commit — ~2,400 minutes in five days, a second dead quota three days after the first.)*

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

**CI triggers (org ruling, 2026-08-22, quota incident ×2):** workflows
trigger on `pull_request` + `push: branches: [main]` only — never bare
push on feature branches (double-fires bill every SHA twice). Concurrency
per ref with cancel-in-progress. Heavy jobs (browser suites) gated to
PRs + main. A branch that needs checks opens a DRAFT PR. Proven in
whr-web#24 (one run per SHA, before/after run IDs in the PR).
**Amended 2026-09-13 (geni #120), founder to ratify or revert:**
cancel-in-progress **except on `main`** — `cancel-in-progress: ${{ github.ref
!= 'refs/heads/main' }}`. Cancelling on a shared ref discards the verdict on
the commit underneath, and on `main` that commit is a merge nobody checks
again: measured twice in geni-frontend (d5ab4746, 1ee3b3e4 — `unit` killed
inside `npm ci`, never re-run, and `cancelled` reads as neither pass nor
fail). `main` takes ~1 push per arc, so the quota cost is ~one 4-minute run
that nobody was going to supersede; every other ref still cancels.
