---
name: quick-implement-task
description: Use to one-shot implement a quick task or fix from a rough idea — given directly in the prompt or as a ticket reference (Jira, GitHub issue, or other tracker) — without first writing a detailed functional/non-functional specification. Builds it with the same engineering discipline as implement-task (test-driven development, KISS/YAGNI, adversarial testing via the adversarial-tester skill, end-to-end functional testing where possible). For small tasks and quick fixes that don't need a lot of thinking; for larger, ambiguous work define a specification first and use implement-task.
---

# quick-implement-task — One-Shot Small Tasks the Right Way

Implement a small task or quick fix directly from a rough idea, with no prior
specification step. This is the fast path: take the idea — from the prompt or a
ticket — and implement it in one shot, applying the **same engineering
standards** as `implement-task` (test-first development, the simplest design that
works, an adversarial pass to find bugs, and end-to-end testing where feasible).

The **only** difference from `implement-task` is the input: there is no
well-detailed specification of functional and non-functional requirements. You
work straight from the rough idea.

Use this for quick tasks and quick fixes that don't require a lot of thinking. If
the work is large, ambiguous, or has non-obvious requirements, prefer writing a
specification first (`define-functional-requirement` /
`define-implementation-plan`) and using `implement-task`.

## Input

The idea arrives in one of these forms:

| Source | What you get | How to resolve it |
| --- | --- | --- |
| **Prompt** | A rough description of the task or fix, written directly in chat. | Use it as the task description. |
| **Ticket** | A ticket id (e.g. `PROJ-123`). | Fetch the ticket; its body is the task description. |

For tickets, fetch with the same tooling used elsewhere (`gh issue view <num>`,
the Jira MCP/CLI).

There is **no requirement document** to resolve. If the idea is too ambiguous to
implement without guessing at the intended behavior, ask the user a focused
clarifying question or two before starting — don't write a full specification.

## Procedure

1. **Understand the task** from the prompt or ticket. If genuinely ambiguous,
   ask one or two focused questions; otherwise proceed.
2. **Implement it in one shot** using the Implementation Brief below. For a
   single rough idea, do the work directly. If the idea naturally splits into a
   few ordered pieces, implement them **one at a time, in order** — they share
   the working tree and build on each other, so don't run them in parallel.
3. **Surface results.** Report what was built, the tests added, the adversarial
   findings and what was fixed, and the final test/build result. If you hit a
   blocking ambiguity or a failing build you can't resolve, stop and bring it to
   the user.

## Implementation Brief

> You are implementing a quick task or fix from a rough idea. There is no
> specification document — the idea itself is the brief. Before writing anything:
> - Read the task description (from the prompt or ticket) and make sure you
>   understand the **intended behavior**. If it's ambiguous in a way that would
>   make you guess, ask a focused question rather than assuming.
> - Read the project's `CLAUDE.md` and its non-functional requirements (the
>   `./nfr/` documents) to learn **how this project expects tests to be written** —
>   the test framework, the commands to run them, where tests live, coverage
>   expectations, and any testing rules. Follow that guidance; do not invent your
>   own testing conventions. Conform to the project's non-functional requirements
>   even though no feature-specific NFRs were defined.
> - Read the relevant existing code and follow its patterns.
>
> Build the task with **test-driven development**, one unit of behavior at a time,
> using the project's test framework and conventions. Drive your test list from the
> behavior the idea describes. Each behavior goes through three steps:
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
> idea and nothing speculative.
>
> When the functionality is complete, run an **adversarial pass**: invoke the
> `trd-dev:adversarial-tester` skill against the functionality you just built to
> surface edge cases and bugs. Fix the cases that violate the intended behavior;
> note any that fall outside it (do not gold-plate). Keep its tests, and re-run the
> adversarial pass until the in-scope cases pass.
>
> Add **end-to-end functional testing** that exercises the task through its real
> entry point (API, CLI, or UI), not just the pure unit. This is feasible whenever
> such an entry point exists; if you skip it, state in one line why it isn't
> feasible.
>
> **Definition of done** — the task is done only when ALL of these hold:
> - the intended behavior of the idea has passing tests;
> - the adversarial findings are resolved (in-scope bugs fixed, out-of-scope ones
>   noted);
> - end-to-end functional testing is in place where feasible;
> - the full test suite, the build, and the project's lint / format / type checks
>   are all green.
>
> Report the actual command output for the suite, build, and checks — never claim
> success without it. If you hit a genuine blocker or an ambiguity you cannot
> resolve, stop and report it rather than guessing.

## Writing standards

Keep commit messages, test names, and any ticket comments concrete and plain: no
AI-slop phrasing, no buzzwords, and no vague nouns — name the project, the repo,
and file paths relative to the project root.

## Common mistakes

- **Reaching for this on big work** — quick-implement-task is for small tasks and
  quick fixes; if the work is large or ambiguous, write a specification first and
  use `implement-task`.
- **Guessing through ambiguity** — there's no spec to fall back on, so when the
  intended behavior is unclear, ask a focused question instead of assuming.
- **Inventing test conventions** — read `CLAUDE.md` and the `./nfr/` documents and
  follow the project's testing framework, commands, and rules; don't make up your
  own.
- **Code before a failing test** — write the test first and watch it fail; tests
  written after prove nothing.
- **Skipping the refactor step** — RED-GREEN-**REFACTOR**: with the test green,
  simplify the code and cut branches before moving on.
- **Gold-plating** — KISS/YAGNI; build what the idea asks, not what it might one
  day ask.
- **Skipping the adversarial pass** — run `adversarial-tester`, fix the in-scope
  bugs, and re-run it before calling the task done.
- **Calling it done early** — done means the intended behavior has passing tests,
  adversarial findings resolved, e2e where feasible, and suite/build/lint all
  green, with the command output to prove it.
