---
name: adversarial-tester
description: Use when you want to stress-test or break a specific function or module — to find edge-case inputs where it fails before trusting it, when existing tests feel too forgiving, or after implementing tricky logic. Produces failing test cases kept in the repo plus a self-contained HTML report of every case.
---

# adversarial-tester — Break a Function and Report Every Failure

Dispatch a single tester agent whose only goal is to **break** a named piece of
functionality: write a large batch of edge-case tests, keep all of them, and
emit a self-contained HTML report listing every case that passes or fails.

The agent tests against **intended** behavior, not merely current behavior — a
test that only asserts what the code already does can never fail.

## When to use

- You just implemented tricky logic and want an independent pass to find holes.
- Existing tests feel too easy / too few and you suspect untested edge cases.
- Before trusting a function with real data, you want to know where it breaks.

Not for: authoring a project's whole test suite, fixing the bugs found (this
skill only *discovers* — fix separately), or load/performance testing.

## Inputs to confirm first

Inspect the repo, then confirm with the user (default each to your best guess):

| Input | Meaning |
| --- | --- |
| **Target** | The exact function / module / feature to attack. |
| **Type of testing** | Which type of testing should the tester build: unit testing or functional testing, or both |
| **Test command** | How tests run here (`pytest -q`, `npm test`, `cargo test`, …). |
| **Test location** | Directory / convention where test files belong. |
| **Budget** | Time-box for the tester. Default **~3 minutes**. |

## Procedure

1. **Scope.** Resolve the five inputs above. If the target or test command is
   ambiguous, ask before dispatching.
2. **Dispatch one tester agent** (general-purpose) with the Adversarial Brief
   below, substituting the resolved inputs and the absolute paths to this
   skill's `report-template.html` and the output file
   `<cwd>/adversarial-test-report.html`.
3. **Surface results.** When the agent returns, report to the user: total /
   passed / failed counts, the most interesting failures, and the paths to the
   HTML report and the new test files. Do **not** auto-fix the bugs unless the
   user asks — this skill discovers, it does not repair.

## Adversarial Brief (pass to the tester agent)

> You are an adversarial tester. Your sole goal is to **break** `<TARGET>` and
> find as many *distinct* failing cases as possible within **<BUDGET>**.
>
> 1. Study `<TARGET>`'s intended behavior from its signature, docstrings, types,
>    naming, and any spec or docs. Tests must assert what the code *should* do,
>    not just what it currently does — otherwise nothing can fail.
> 2. Write a large batch of edge-case `<TEST_TYPE>` tests in the project's
>    framework, into `<TEST_LOCATION>`. If functional testing is in scope,
>    exercise `<TARGET>` through its real entry points / API, not just the pure
>    function. Systematically cover: boundaries & off-by-one; empty /
>    null / missing; very large inputs; unicode, emoji, encodings, whitespace;
>    type coercion & wrong types; negatives, zero, duplicates; ordering &
>    idempotency; malformed / injection-like / adversarial strings; locale &
>    timezone; concurrency & shared state; error handling & exceptions.
> 3. **Keep every test** — passing ones too. They become a regression suite.
> 4. Run the suite with `<TEST_COMMAND>` and record, per case: a short id,
>    category, one-line description, the input, expected, actual, and pass/fail.
> 5. Build the HTML report: copy `<TEMPLATE_PATH>`, replace ONLY the object
>    between the `BEGIN REPORT DATA` / `END REPORT DATA` markers with your real
>    `{ meta, results }` data, and write the result to `<REPORT_PATH>`. Change
>    nothing else in the template. Set `meta.generatedAt` to the current time
>    and `meta.testType` to `<TEST_TYPE>`.
> 6. Spend most of your budget finding *new* failure modes, not polishing.
>    Return a summary: total / passed / failed and the top failures.

## report-template.html

Ships next to this `SKILL.md`. Self-contained (no external dependencies): it
renders summary cards and a searchable, filterable table, and highlights
failures. The tester injects only the `{ meta, results }` object; nothing else.

### Data shape

```js
{
  meta: { target, testType, framework, command, testFiles: [], budget, generatedAt },
  results: [
    { id, category, description, input, expected, actual, status, notes }
    // status: "pass" | "fail"
  ]
}
```

## Common mistakes

- **Tests assert current behavior** → nothing ever fails. Anchor on *intended*
  behavior instead.
- **Throwing away passing tests** → keep them; they are the regression suite.
- **Hand-writing the HTML** → always start from the template; only the data
  changes.
- **Fixing bugs mid-run** → this skill only finds them; report and stop.
