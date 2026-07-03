---
name: implement-task
description: Use to implement one or more implementation tasks — given a single ticket id, a comma-separated list of ticket ids, or a range like 10..15 or 10...15 (all ids from 10 to 15 inclusive) from a ticketing system, or a single, comma-separated, or ranged task number from an HTML implementation plan (which the user must provide). Works with a full requirement document and implementation plan when they exist, and falls back to just the ticket or the user's description for quick tasks and fixes that don't have one. Dispatches one agent per task, in order, each building the task with test-driven development, KISS/YAGNI, adversarial testing via the adversarial-tester skill, and end-to-end functional testing where possible, then verifying it with two independent reviewers (unrequested/dangerous behavior and side effects elsewhere in the product) and fixing any confirmed issues before moving on.
---

# implement-task — Build Implementation Tasks the Right Way

Implement one or more tasks produced by `define-implementation-plan`. Each task is
built by its own agent, one task at a time in order, using disciplined
engineering: test-first development, the simplest design that works, an
adversarial pass to find bugs, and end-to-end testing where it's feasible.

## Input

The task list arrives in one of these forms:

| Source | What you get | How to resolve it |
| --- | --- | --- |
| **One ticket** | A single ticket id (e.g. `PROJ-123`). | Fetch the ticket; its body is the task spec. |
| **Several tickets** | A comma-separated list of ticket ids, a range, or a mix of both. | Expand to the full ordered list (see *Ranges* below), fetch each; implement in the given order. |
| **HTML plan** | One task number, a comma-separated list, or a range (e.g. `1`, `1,2,3`, or `1..3`), **plus the path to the `implementation-plan.html`** the user must provide. | Expand ranges, then read the named numbered task(s) from that file. |

**Ranges.** A range with two or three dots — `10..15` or `10...15` — means **every
id from the first to the last, inclusive** (`10..15` → `10, 11, 12, 13, 14, 15`).
Both dot forms mean the same thing. Ranges may be combined with individual ids in a
comma-separated list (e.g. `10..12, 15, 18..20`); expand each range, keep the order
as written, and drop duplicates. Ranges apply to purely numeric ids (issue numbers
or HTML plan task numbers). For prefixed ticket ids that share one numeric prefix
(e.g. `PROJ-10..PROJ-15`, or `PROJ-10..15`), expand over the numeric part; if the
endpoints don't share a single obvious prefix, ask the user rather than guessing.

For tickets, fetch with the same tooling used elsewhere (`gh issue view <num>`,
the Jira MCP/CLI). For the HTML form, **the user must provide the path to the
implementation plan**; if they reference an HTML plan without a path, ask for it
before doing anything.

Every task spec normally links back to a requirement document
(`docs/<feature-slug>/requirement.html`). Resolve it — the agent needs the
functional and non-functional requirements, not just the task text.

If no requirement document (or full implementation plan) is available — for a
quick task or fix given straight as a ticket or a description in the prompt —
don't require one. Use whatever is in the ticket or the user's description as the
spec, and apply the same engineering discipline below. In that case, drive the
work from the ticket/description text instead of a linked requirement, and skip
the requirement-document steps in the Implementation Brief.

## Procedure

1. **Resolve the task list** into an ordered sequence of (task spec + linked
   requirement). First **expand any ranges** (`10..15` / `10...15` → every id from
   10 to 15 inclusive) and merge them with any individual ids, keeping the order as
   written and dropping duplicates. Then keep that order (ticket order, or ascending
   task number).
2. **Implement tasks one at a time, in order.** For each task, dispatch **one
   general-purpose agent** with the Implementation Brief below, substituting the
   task spec, the path to its requirement document, and the repo. **Wait for the
   agent to finish and report before starting the next task** — tasks build on
   each other and share the working tree, so they must not run in parallel.
3. **Verify the task** before moving on — run the two independent reviewers in
   *Verification* below, then have the build agent fix any confirmed issue. A task
   is not finished until its verdict is clean. Do this per task so problems are
   caught while the context is fresh and before the next task builds on top.
4. **Surface results.** After each task, report what was built, the tests added,
   the adversarial findings and what was fixed, the verification verdict, and the
   final test/build result. If a task's agent reports it could not finish (e.g. a
   blocking ambiguity or a failing build it can't resolve), or verification lands
   on REJECT, stop and bring it to the user before continuing to the next task.

## Implementation Brief (pass to each agent)

> You are implementing a single task. Prior tasks in the sequence are already
> implemented in the working tree. You have this task's spec, and — when one
> exists — the path to its requirement document
> (`docs/<feature-slug>/requirement.html`). Before writing anything:
> - Read the task spec and, if a requirement document was provided, the
>   requirement — honor the **functional and non-functional requirements**, not
>   just the task text. If there is no requirement document, the task spec (the
>   ticket or the user's description) is the whole spec; work from it directly.
> - Read the project's `CLAUDE.md` and its non-functional requirements (the
>   `./nfr/` documents) to learn **how this project expects tests to be written** —
>   the test framework, the commands to run them, where tests live, coverage
>   expectations, and any testing rules. Follow that guidance; do not invent your
>   own testing conventions.
> - Read the relevant existing code and follow its patterns.
>
> Build the task with **test-driven development**, one unit of behavior at a time,
> using the project's test framework and conventions. Drive your test list from the
> task's **acceptance criteria**. Each behavior goes through three steps:
> 1. **RED** — Write a unit test that asserts the *intended* behavior of the next
>    small piece of functionality. Run it and **confirm it fails** for the right
>    reason (the behavior is missing, not a typo in the test).
> 2. **GREEN** — Implement the **minimal** code to make the test pass. Run it and
>    **confirm it passes**.
> 3. **REFACTOR** — With the test green, **simplify the code as much as possible**:
>    remove duplication and reduce the number of branches/conditionals to the
>    minimum the behavior needs. Re-run the test and confirm it is **still green**.
>
> Repeat the three steps for the next behavior until the task is complete.
>
> Follow **KISS and YAGNI** throughout: build the simplest thing that satisfies the
> requirement and nothing speculative. Honor the task's interface contract exactly
> (especially any shared interface), and its independently-releasable constraint —
> if it must ship disabled, put it behind the named flag with the stated default.
> **At all times conform to the project's and the task's non-functional
> requirements.**
>
> When the task's functionality is complete, run an **adversarial pass**: invoke
> the `trd-dev:adversarial-tester` skill against the functionality you just
> built to surface edge cases and bugs. Fix the cases that violate the requirement;
> note any that fall outside it (do not gold-plate). Keep its tests, and re-run the
> adversarial pass until the in-scope cases pass.
>
> Add **end-to-end functional testing** that exercises the task through its real
> entry point (API, CLI, or UI), not just the pure unit. This is feasible whenever
> such an entry point exists; if you skip it, state in one line why it isn't
> feasible.
>
> **Definition of done** — the task is done only when ALL of these hold:
> - every acceptance criterion in the task spec has a passing test;
> - the adversarial findings are resolved (in-scope bugs fixed, out-of-scope ones
>   noted);
> - end-to-end functional testing is in place where feasible;
> - the full test suite, the build, and the project's lint / format / type checks
>   are all green.
>
> Report the actual command output for the suite, build, and checks — never claim
> success without it. If you hit a genuine blocker or an ambiguity the spec and the
> requirement cannot resolve, stop and report it rather than guessing.

## Verification

Once the build agent reports a task done, **verify it before moving on**. The
build agent just wrote this code, so it is the wrong reviewer for its own work —
it is anchored on its own mental model and blind to its own scope creep. Bring in
fresh, adversarial eyes.

The passing tests and the adversarial pass already prove the task *does what its
acceptance criteria ask*, so verification does **not** re-walk conformance. It
covers the two things tests don't: behavior that was never asked for, and damage
elsewhere in the product. Dispatch **two independent agents in parallel**, each
reading only (they inspect and report — they never edit code):

- **Unrequested / dangerous behavior.** Read the task's diff and find behavior the
  requirement never asked for; judge whether any of it is dangerous — scope creep,
  silent side effects, weakened validation or permissions, data loss, broadened
  attack surface, debug/backdoor code, changed defaults, swallowed errors. Rate
  each by severity. Ignore behavior the requirement explicitly authorizes.
- **Side effects.** Given what this change does, predict which *other* parts of the
  codebase and product it could affect (named modules, callers, shared state,
  contracts, UX flows), then **actually check each one**: read the named code and
  decide whether the change breaks it, contradicts it, or introduces behavior
  inconsistent with it. Confirm or refute each with a `file:line` citation. Be
  adversarial but honest — do not invent problems.

Every finding must cite `file:line`. Drop anything the check itself refuted; keep
only real, evidence-backed problems.

**Act on the verdict:**

- **Clean** (no confirmed problems) — the task is done; move to the next task.
- **Fixable problems** — hand them back to the build agent to fix (following the
  same TDD discipline), then re-run verification until it is clean.
- **Fundamental problems** (the approach is wrong, or a confirmed break the fix
  can't contain) — stop and bring it to the user before continuing.

When there is no formal requirement document — a quick task run from just a ticket
or description — still run both checks; they judge the diff against the ticket's
intent and the rest of the codebase, neither of which needs a formal spec.

## Writing standards

Keep commit messages, test names, and any ticket comments concrete and plain: no
AI-slop phrasing, no buzzwords, and no vague nouns — name the project, the repo,
and file paths relative to the project root.

## Common mistakes

- **Running tasks in parallel** — they share the working tree and build on each
  other; implement them one at a time, in order.
- **Inventing test conventions** — read `CLAUDE.md` and the `./nfr/` documents and
  follow the project's testing framework, commands, and rules; don't make up your
  own.
- **Code before a failing test** — write the test first and watch it fail; tests
  written after prove nothing.
- **Skipping the refactor step** — RED-GREEN-**REFACTOR**: with the test green,
  simplify the code and cut branches before moving on.
- **Gold-plating** — KISS/YAGNI; build what the requirement asks, not what it
  might one day ask.
- **Skipping the adversarial pass** — run `adversarial-tester`, fix the in-scope
  bugs, and re-run it before calling the task done.
- **Calling it done early** — done means every acceptance criterion has a passing
  test, adversarial findings resolved, e2e where feasible, suite/build/lint all
  green with the command output to prove it, **and** a clean verification verdict.
- **Skipping verification** — after the build agent is done, run the two
  independent reviewers (unrequested behavior + side effects) and fix confirmed
  problems before the next task builds on top.
- **Letting the builder verify itself** — verification needs fresh, adversarial
  eyes; dispatch separate agents, don't ask the build agent to review its own diff.
- **Ignoring the requirements** — the task spec is not the whole story; conform to
  the project's and the task's functional *and* non-functional requirements at all
  times.
