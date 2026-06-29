---
name: verify-implementation
description: Use to check that the work on the current branch actually matches the requirement that drove it — given the branch's linked ticket or its requirement document under docs/. Launches a dynamic workflow that, in parallel, verifies the implementation meets the requirement, hunts for unrequested or dangerous behavior it slipped in, predicts where the change could ripple into the rest of the product, and then confirms whether those ripples actually broke or contradicted anything. Produces a single verdict with evidence.
---

# verify-implementation — Check the Branch Against Its Requirement

Take the work sitting on the current git branch and the requirement that asked for
it, and decide — with evidence — whether the implementation is correct, complete,
and safe. This is a **verification** skill: it reads and reasons, it does not
change code. The output is a verdict, not a fix.

It runs a **dynamic workflow** that fans four analyses out concurrently and then
synthesizes them into one report.

## Inputs

Two things must be resolved before the workflow starts: **the implementation** and
**the requirement** it was meant to satisfy.

### The implementation — the current branch

The implementation under review is the diff of the current branch against the base
branch (usually `main`). Resolve it inline:

- Base branch: the repo's default branch (`main` or `master`); if the branch was
  cut from something else, ask the user.
- The change set is `git diff <base>...HEAD` plus `git diff --stat <base>...HEAD`
  for the file list. Capture both.
- If the branch is the base branch itself (nothing to compare), stop and tell the
  user there is no branch work to verify.

### The requirement — ticket or docs

The requirement arrives in one of two forms:

| Source | What you get | How to resolve it |
| --- | --- | --- |
| **Ticket** | The ticket id that set the requirement for this branch. | Fetch it (`gh issue view <num>`, the Jira MCP/CLI). Branch names usually carry the id — derive it from the branch name, else ask the user. Use the **full** ticket body, including any linked requirement document. |
| **Docs spec** | A requirement document under `docs/<feature-slug>/requirement.html`. | Derive the feature slug from the branch name; if several docs could match, ask the user which one. Read the whole document — functional **and** non-functional sections. |

If neither a ticket nor a `docs/<slug>/requirement.html` can be located, **stop and
ask the user** to point you at the requirement. Never verify against a guessed
requirement.

## Procedure

1. **Resolve inputs.** Get the diff and stat for the current branch against its
   base, and the full requirement text (ticket body or requirement document). Note
   the repo root and base branch.
2. **Launch the workflow** (below), passing the requirement text, the changed-file
   list, the base branch, and the repo root as `args`. The four analyses run as
   described.
3. **Report the verdict** to the user from the workflow's synthesized result — see
   *Output*.

## The workflow

Call the `Workflow` tool with the script below. It carries four analyses:

- **A — Conformance.** Does the implementation actually satisfy the requirement?
  Walk every acceptance criterion / stated behavior and mark it met, partially met,
  or missing, citing the code (`file:line`) that satisfies it. Flag requirements
  with no corresponding code.
- **B — Unrequested behavior.** Does the diff introduce behavior the requirement
  never asked for — and is any of it dangerous or undesirable? Look for scope creep,
  silent side effects, weakened validation/permissions, data loss, broadened
  surface area, debug/backdoor code, and changed defaults. Rate each by risk.
- **C — Side-effect prediction.** Step back and analyze the **whole product and the
  whole requirement set**: given what this change does, which *other* parts of the
  codebase and product could it affect? Produce concrete, checkable hypotheses
  (named modules, callers, shared state, contracts, UX flows), each with why it
  could be impacted and how to confirm it.
- **D — Side-effect verification.** Take C's hypotheses and actually check them:
  read the named code and confirm whether the change breaks, contradicts, or
  introduces behavior **inconsistent** with those other parts. C feeds D — D runs
  per hypothesis as soon as C produces them (a pipeline, not a barrier), so D is not
  blind guessing. A and B run alongside.

A final **synthesis** agent merges A, B, and D into one verdict.

```js
export const meta = {
  name: 'verify-implementation',
  description: 'Verify a branch against its requirement: conformance, unrequested behavior, and side effects',
  phases: [
    { title: 'Analyze' },
    { title: 'Verify side effects' },
    { title: 'Synthesize' },
  ],
}

// args: { requirementText, changedFiles: string[], baseBranch, repoRoot }
const ctx = `Repo root: ${args.repoRoot}
Base branch: ${args.baseBranch}
Changed files:
${args.changedFiles.join('\n')}

You can run \`git diff ${args.baseBranch}...HEAD\` (optionally scoped to a path) to read the actual change, and read any file in the repo. Only inspect — never modify code.

THE REQUIREMENT:
${args.requirementText}`

const FINDINGS = {
  type: 'object',
  additionalProperties: false,
  required: ['summary', 'items'],
  properties: {
    summary: { type: 'string' },
    items: {
      type: 'array',
      items: {
        type: 'object',
        additionalProperties: false,
        required: ['title', 'status', 'severity', 'evidence'],
        properties: {
          title: { type: 'string' },
          status: { type: 'string', description: 'met | partial | missing | risk | confirmed | refuted' },
          severity: { type: 'string', description: 'none | low | medium | high | critical' },
          evidence: { type: 'string', description: 'file:line citations and a one-line justification' },
        },
      },
    },
  },
}

const HYPOTHESES = {
  type: 'object',
  additionalProperties: false,
  required: ['hypotheses'],
  properties: {
    hypotheses: {
      type: 'array',
      items: {
        type: 'object',
        additionalProperties: false,
        required: ['area', 'why', 'howToCheck'],
        properties: {
          area: { type: 'string', description: 'named module / caller / contract / flow that could be affected' },
          why: { type: 'string' },
          howToCheck: { type: 'string' },
        },
      },
    },
  },
}

phase('Analyze')

// A and B run alongside the C->D pipeline.
const [conformance, unrequested, sideEffects] = await parallel([
  () => agent(
    `${ctx}\n\nTASK — CONFORMANCE. Walk every acceptance criterion and stated behavior in the requirement. For each, decide whether the branch's implementation meets it (met / partial / missing) and cite the code (file:line) that satisfies it. List any requirement with no corresponding code as missing. Honor non-functional requirements too.`,
    { label: 'A:conformance', phase: 'Analyze', schema: FINDINGS },
  ),
  () => agent(
    `${ctx}\n\nTASK — UNREQUESTED BEHAVIOR. Read the diff and find behavior the requirement never asked for. For each, judge whether it is dangerous or undesirable: scope creep, silent side effects, weakened validation or permissions, data loss, broadened attack surface, debug/backdoor code, changed defaults, swallowed errors. status=risk; set severity by how dangerous it is. Ignore behavior the requirement explicitly authorizes.`,
    { label: 'B:unrequested', phase: 'Analyze', schema: FINDINGS },
  ),
  () => agent(
    `${ctx}\n\nTASK — SIDE-EFFECT PREDICTION. Analyze the whole product and the whole requirement set. Given what this change does, which OTHER parts of the codebase and product could it affect? Produce concrete, checkable hypotheses — name the module, caller, shared state, contract, or UX flow; say why it could be impacted and exactly how to confirm it. Prefer a focused set of high-value hypotheses over a long speculative list.`,
    { label: 'C:predict', phase: 'Analyze', schema: HYPOTHESES },
  ).then(r => parallel((r?.hypotheses ?? []).map(h => () =>
    agent(
      `${ctx}\n\nTASK — SIDE-EFFECT VERIFICATION. A prior analysis predicted this area could be affected by the branch:\nArea: ${h.area}\nWhy: ${h.why}\nHow to check: ${h.howToCheck}\n\nActually check it. Read the named code and decide whether the branch breaks it, contradicts it, or introduces behavior INCONSISTENT with it. status=confirmed if a real problem exists, refuted if not. Cite file:line. Be adversarial but honest — do not invent problems.`,
      { label: `D:verify:${h.area}`.slice(0, 60), phase: 'Verify side effects', schema: FINDINGS },
    ),
  ))),
])

const verifiedSideEffects = (sideEffects ?? []).filter(Boolean)

phase('Synthesize')

const verdict = await agent(
  `${ctx}\n\nTASK — SYNTHESIZE THE VERDICT. You are given the results of three analyses of the branch.\n\nCONFORMANCE (does it meet the requirement):\n${JSON.stringify(conformance, null, 2)}\n\nUNREQUESTED / DANGEROUS BEHAVIOR:\n${JSON.stringify(unrequested, null, 2)}\n\nVERIFIED SIDE EFFECTS (inconsistencies with the rest of the product, already checked against the code):\n${JSON.stringify(verifiedSideEffects, null, 2)}\n\nProduce one verdict for the user. Lead with an overall call: SHIP / FIX-FIRST / REJECT. Then: unmet or partial requirements; dangerous unrequested behavior; confirmed side effects / inconsistencies. Keep only real, evidence-backed findings (drop refuted ones). Cite file:line. Be concrete and blunt — no filler.`,
  { label: 'synthesize', phase: 'Synthesize' },
)

return { verdict, conformance, unrequested, sideEffects: verifiedSideEffects }
```

## Output

Report to the user, in this order:

1. **Verdict** — SHIP / FIX-FIRST / REJECT, one line of why.
2. **Requirement gaps** — anything unmet or only partially met, with the citation.
3. **Unrequested / dangerous behavior** — what the diff added beyond the
   requirement and why it matters.
4. **Confirmed side effects** — places elsewhere in the product the change breaks
   or contradicts, each with the code that proves it.

Cite `file:line` for every finding. This skill does not edit code — if the user
wants the issues fixed, point them at `implement-task` / `quick-implement-task`.

## Writing standards

Same standards as the other `trd-dev` skills: no AI-slop phrasing, no buzzwords,
no vague nouns — name the repo, the file, the line. Every finding must be
checkable and backed by a citation; if something is uncertain, say so rather than
asserting it.

## Common mistakes

- **No requirement located** — hard-stop and ask the user for the ticket or the
  `docs/<slug>/requirement.html`; never verify against a guessed requirement.
- **Verifying the wrong diff** — compare the branch against its real base; confirm
  the base with the user if the branch wasn't cut from `main`.
- **Editing code** — this skill verifies and reports only; it does not fix.
- **Side-effect verification without prediction** — D must consume C's hypotheses;
  don't let it guess blind.
- **Reporting refuted findings as real** — the synthesis drops anything the
  verification step refuted; only ship evidence-backed findings.
- **No citations** — every gap, risk, and side effect names the file and line.
