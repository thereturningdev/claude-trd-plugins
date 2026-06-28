---
name: nfr-init
description: Use to INITIALIZE a project's non-functional requirements, or to UPDATE them when the project has changed significantly (e.g. it gained Python or a web component) or the bundled NFR set changed. Detects the project type (with the user's confirmation), selects only the applicable bundled NFR documents, drops them into ./nfr/, and wires them into CLAUDE.md as mandatory.
---

# nfr-init — Initialize or Update a Project's Non-Functional Requirements

This skill exists purely to **initialize** a project's non-functional
requirements — or to **update** them when the project has changed significantly.
It selects the bundled non-functional-requirement documents that apply to THIS
project, copies them into the project's `./nfr/` folder, and makes `CLAUDE.md`
treat them as mandatory. It is not a tool for authoring or editing requirement
content (that lives in the bundled `requirements/` library).

## When to use

- **Initialize** — bootstrapping a project that has no `./nfr/` set yet.
- **Update** — the project changed significantly since the NFRs were last applied
  (for example it gained a web frontend or started using Python, so a different
  set of documents now applies), or the bundled NFR library itself changed.

Re-running is safe: the selection is recomputed and `./nfr/` plus the `CLAUDE.md`
section are reconciled to it.

## The bundled library

The canonical NFR documents live in this skill's `requirements/` directory (next to
this `SKILL.md`). They are NOT all copied blindly — each applies only to certain
kinds of project. The skill copies the selected files verbatim; it does not invent,
edit, or summarize requirement content.

## Applicability rules

| Bundled file | Copy it when |
| --- | --- |
| `NFR-general.md` | **Always** — every project. |
| `NFR-development.md` | The project is a **software-development project** (it holds or will hold source code). |
| `NFR-python.md` | The project **uses Python**. |
| `NFR-swift.md` | The project **uses Swift**. |
| `NFR-web.md` | The project has a **web component** (a UI / frontend, or it serves HTTP). |

If `requirements/` contains a `.md` file not listed above, ask the user whether it
applies before copying it.

## Procedure

1. **Target.** The target is the current working directory (the project root).
   Confirm it with the user if ambiguous.

2. **Gather signals.** Inspect the project to infer its nature. Look for:
   - **Python:** `*.py`, `pyproject.toml`, `requirements.txt`, `setup.py`,
     `Pipfile`, `uv.lock`, `poetry.lock`.
   - **Swift:** `*.swift`, `Package.swift`, `Package.resolved`, `*.xcodeproj`,
     `*.xcworkspace`, a `Sources/` directory.
   - **Web:** `package.json` (especially with react / vue / svelte / angular /
     next / vite / express), `*.html`, `*.jsx` / `*.tsx`, a frontend `src/` or
     `public/` directory.
   - **Software development:** any source code, build/package manifests, or a
     populated git repo — i.e. this is a codebase, not just notes or docs.

   Keep the evidence; you will show it to the user.

3. **Ask the user.** State concisely what you detected, then ask the user to confirm
   which categories apply. The user's answer is authoritative and overrides
   detection. Confirm specifically: is this a software-development project? does it
   use Python? does it use Swift? does it have a web component? Default each answer
   to your detection.

4. **Select files.** Apply the applicability table to the confirmed answers:
   - always → `NFR-general.md`
   - software-development → `NFR-development.md`
   - Python → `NFR-python.md`
   - Swift → `NFR-swift.md`
   - web component → `NFR-web.md`

5. **Copy.** Ensure an `nfr/` subdirectory exists in the project root. Copy each
   SELECTED `.md` file into `./nfr/`, overwriting existing copies (they are
   canonical). Do NOT copy unselected files, and do NOT copy non-`.md` files. Then
   reconcile: remove from `./nfr/` any file whose name matches a bundled library
   file but is NOT in the current selection, so the folder reflects the current
   project type. Never delete files in `./nfr/` that did not come from the bundled
   library.

6. **Ensure CLAUDE.md exists.** If the project root has no `CLAUDE.md`, create an
   empty one.

7. **Wire CLAUDE.md (idempotent).** Maintain a single managed section delimited by
   these exact markers:

   ```
   <!-- nfr:start -->
   ## Non-Functional Requirements (MANDATORY)

   These requirements are mandatory for all work in this project. Read each file
   and follow it:

   - [<file name without .md>](nfr/<file>)
   ... one bullet per file now present in ./nfr/ ...
   <!-- nfr:end -->
   ```

   - If the markers already exist in `CLAUDE.md`, replace everything between them
     with the freshly generated list, so the references always match the selected
     files.
   - If the markers do not exist, append the whole block (preceded by a blank line)
     to the end of `CLAUDE.md`.
   - Use the file name without its `.md` extension as the link text.

8. **Report.** Tell the user which files were copied into `./nfr/`, which bundled
   files were skipped and why, and confirm the `CLAUDE.md` section was created or
   refreshed.

## Notes

- Safe to re-run: the selection is recomputed, `./nfr/` is reconciled to it, and the
  `CLAUDE.md` section is replaced in place via the markers — it never duplicates.
- Requirement content is owned by the bundled `requirements/` library, not by this
  procedure. To change what projects receive, edit the files in `requirements/`; to
  add a new category, add the file AND a row to the applicability table above.
