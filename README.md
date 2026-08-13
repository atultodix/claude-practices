# Claude Practices

Working practices for building with Claude, distilled from a live multi-project build. **Three files, three readers, three moments:**

| File | Reader | Loaded when | Ceiling |
|---|---|---|---|
| **`rules.md`** | **Claude Code** — whoever holds the keyboard | Every session, automatically, via `~/.claude/CLAUDE.md` | **200 lines** |
| **`coordinator.md`** | **Claude Chat** acting as coordinator | Pasted at project-chat start **and at every handoff** | ~200 lines |
| **`practices.md`** | **A human, or a bootstrapping session** | Once per project — the method and how the pieces fit | none |

**The two audiences are genuinely different.** `rules.md` answers *"how do I build here?"*. `coordinator.md` answers *"how do I plan, review, dispatch and hand off?"* — including the review standard and the handoff protocol itself. A handful of rules appear in both; that is correct, not duplication, because neither reader should have to read the other's file to be complete.

---

## Setup (per machine, once)

```bash
git clone git@github.com:<org>/claude-practices.git ~/claude-practices
mkdir -p ~/.claude && ln -sf ~/claude-practices/rules.md ~/.claude/CLAUDE.md
```

**Symlink, never copy.** A copy drifts the moment either side changes, and drift is the exact problem this repo exists to solve. To update everything on that machine:

```bash
cd ~/claude-practices && git pull
```

`~/.claude/CLAUDE.md` is user-level memory: Claude Code loads it in every session regardless of which project folder you're in. A project's own `CLAUDE.md` stacks on top with project-specific facts.

**For Claude Chat project sessions**, paste `coordinator.md` at the start of the chat and again at every handoff — there is no auto-load equivalent, so this stays manual, but it comes from a versioned file rather than a copy on someone's desktop. `practices.md` goes into the project's own instructions when the project is set up.

**Handoff prompt** (paste into the fresh chat, with `coordinator.md`):

```
You're taking over as engineering coordinator for <project>. The attached
coordinator.md is how we work — treat it as standing instructions.

Start by reading, in this order: the latest journal entry, the decision log's
"For Next Session" block, and the state-of-build doc. Then verify what they
claim against the code and the database before acting on any of it.

Give me a state report before we do anything: what's merged, what's in flight
and on which branch, what's blocked, and what's waiting on me.
```

---

## Governance — what earns a place here

Any project can propose a rule. **Most proposals should be rejected.** Every line added to `rules.md` dilutes the rest: past roughly 200 lines, instruction-following measurably degrades. This is a **zero-sum file** and should be treated as one.

### The five tests — a rule enters `rules.md` only if it passes all of them

1. **General.** Would a *different* project, with a different codebase and stack, have been better off knowing this? If it names a specific vendor, repo, tenant, person or product, it fails — that belongs in the project's own `CLAUDE.md`.
2. **Paid for in blood.** Did *not* knowing this cost something real — a broken build, lost work, a wrong decision shipped? Rules born from incidents survive contact. Rules born from "seems sensible" are bloat.
3. **Actionable at a moment.** It must say what to do, when. *"Be careful with migrations"* is a sentiment. *"Create migrations with `--create-only`; never run them manually against prod"* is a rule.
4. **Non-derivable.** Not something a competent assistant would do anyway, and not deducible from reading the code.
5. **Not already covered.** If it sharpens an existing rule, **edit that rule** — never add a near-duplicate. Two rules saying almost the same thing is worse than one saying it well.

Fails any test → it goes to `practices.md`, or to the project's own instructions, or nowhere.

### The budget makes the tradeoff explicit

`rules.md` has a **200-line ceiling**; `coordinator.md` holds to roughly the same. At the ceiling, adding means **removing** — which forces the question *"is this more valuable than what it displaces?"* every time. A file that only grows is a file nobody reads carefully.

**Which file?** Ask who needs it *at the moment it applies*. The builder mid-flight → `rules.md`. The coordinator planning, reviewing or handing off → `coordinator.md`. Both → both; the small overlap is deliberate. Neither, but a human setting up a project → `practices.md`.

### Provenance — one line, always

Every rule carries the incident that produced it, compressed:

> *(three blocked flights in one week came from assuming an MCP-read file was in the repository)*

This does two jobs: it makes the rule defensible when someone asks "why do we do this," and it makes the rule **prunable** — when the incident can no longer recur, the rule can retire.

### The prune

At the ceiling, or quarterly, whichever comes first: read every rule and ask *has this earned its place?* A rule nobody has needed and whose incident is now structurally impossible gets **demoted to `practices.md`** — kept, not deleted, because the reasoning may matter later.

### Flow, in both directions

- **Project → here.** When a practice proves itself in a project, apply the five tests. If it passes, PR it here and note "promoted to shared practices" in that project's journal.
- **Here → projects.** A rule learned here should be restated in active projects' own instructions in their own terms, not left waiting to be read at some future session start.

### Who decides

Proposals come by PR — from a person or from Claude. **The owner rules.** Claude's job when proposing is to *apply the five tests out loud and say which one the rule is weakest on* — not to advocate for its own suggestion.
