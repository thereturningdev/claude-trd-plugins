---
name: implement-task
description: Use to implement one or more already-defined implementation tasks — given a single ticket id or a comma-separated list of ticket ids from a ticketing system, or a single or comma-separated task number from an HTML implementation plan (which the user must provide). Dispatches one agent per task, in order, each building the task with test-driven development, KISS/YAGNI, adversarial testing via the adversarial-tester skill, and end-to-end functional testing where possible.
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
| **Several tickets** | A comma-separated list of ticket ids. | Fetch each; implement in the given order. |
| **HTML plan** | One task number, or a comma-separated list (e.g. `1` or `1,2,3`), **plus the path to the `implementation-plan.html`** the user must provide. | Read the named numbered task(s) from that file. |

For tickets, fetch with the same tooling used elsewhere (`gh issue view <num>`,
the Jira MCP/CLI). For the HTML form, **the user must provide the path to the
implementation plan**; if they reference an HTML plan without a path, ask for it
before doing anything.

Every task spec links back to a requirement document
(`docs/<feature-slug>/requirement.html`). Resolve it — the agent needs the
functional and non-functional requirements, not just the task text.

## Procedure

1. **Resolve the task list** into an ordered sequence of (task spec + linked
   requirement). Keep the order given (ticket order, or ascending task number).
2. **Implement tasks one at a time, in order.** For each task, dispatch **one
   general-purpose agent** with the Implementation Brief below, substituting the
   task spec, the path to its requirement document, and the repo. **Wait for the
   agent to finish and report before starting the next task** — tasks build on
   each other and share the working tree, so they must not run in parallel.
3. **Surface results.** After each task, report what was built, the tests added,
   the adversarial findings and what was fixed, and the final test/build result.
   If a task's agent reports it could not finish (e.g. a blocking ambiguity or a
   failing build it can't resolve), stop and bring it to the user before
   continuing to the next task.

## Implementation Brief (pass to each agent)

> You are implementing a single task. Prior tasks in the sequence are already
> implemented in the working tree. You have this task's spec and the path to its
> requirement document (`docs/<feature-slug>/requirement.html`). Before writing
> anything:
> - Read the task spec and the requirement — honor the **functional and
>   non-functional requirements**, not just the task text.
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
  test, adversarial findings resolved, e2e where feasible, and suite/build/lint all
  green, with the command output to prove it.
- **Ignoring the requirements** — the task spec is not the whole story; conform to
  the project's and the task's functional *and* non-functional requirements at all
  times.
