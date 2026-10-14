# Recipe: pin ct-conventions in a package (theme 1)

Part of [`harmonize.md`](harmonize.md): read its ground rules, protocol (issues, worktree,
linking) and narration first. This file only holds what is specific to theme 1.

Align one control-toolbox package on [ct-conventions](https://github.com/control-toolbox/ct-conventions):
a pinned `.conventions/` submodule, an `AGENTS.md` that cites rule identifiers, a
`CLAUDE.md` that only redirects. Mechanical and checkable.

Model to copy: [CTBase.jl#595](https://github.com/control-toolbox/CTBase.jl/pull/595).

## Inputs

`PKG`, `REPO`, and `TAG` — the ct-conventions tag to pin (e.g. `v0.2.0`).
Branch and worktree: `chore/pin-ct-conventions`, `.worktrees/$PKG-pin-conventions`.

## Steps

1. **Submodule.** In the worktree (protocol step B):

   ```bash
   # only if the package has a .handbook submodule:
   git submodule deinit -f .handbook && git rm -f .handbook && rm -rf .git/modules/.handbook
   git submodule add https://github.com/control-toolbox/ct-conventions.git .conventions
   git -C .conventions checkout "$TAG"
   ```

2. **`AGENTS.md`.** Rewrite it from the template below. Keep any section that is specific
   to this package (its own layout, commands, pitfalls) after `Key Conventions`. If a
   specific section contradicts a rule of ct-conventions, keep it and report it; do not
   decide.

3. **`CLAUDE.md`.** Replace its whole content with:

   ```markdown
   # <PKG> — Claude Code

   All agent instructions live in `AGENTS.md`; this file only redirects to it.

   @AGENTS.md
   ```

4. **References to the Handbook.** Count them **before** changing anything, on the
   unmodified code, and list them:

   ```bash
   git grep -nIi handbook origin/main -- . ':!CHANGELOG.md' ':!BREAKING.md' | wc -l   # BEFORE
   git grep -nIi handbook origin/main -- . ':!CHANGELOG.md' ':!BREAKING.md'           # the list
   ```

   The count is above zero for every package (the `.handbook` and `Handbook` spellings
   both match). If it is zero, the command is wrong or the package is already done: stop
   and report. After the replacements, the same search on the working tree must find
   nothing:

   ```bash
   grep -rnIi handbook . --exclude-dir=.git --exclude-dir=.conventions --exclude=CHANGELOG.md --exclude=BREAKING.md | wc -l   # AFTER
   ```

   Report both counts in the step 4 comment, and the list as `old → new (rule)`.

   and replace each by its new form. Links to the old `philosophy/<file>.md` pages become
   links to `rules/<new file>.md` of ct-conventions, with the identifier when the sentence
   is about one rule:

   | Old page | New page | Prefix |
   | --- | --- | --- |
   | `philosophy/modules.md` | `rules/modules.md` | `MOD` |
   | `philosophy/types-traits-interfaces.md` | `rules/types.md` | `TYP` |
   | `philosophy/dimension-and-shape.md` | `rules/dimension.md` | `DIM` |
   | `philosophy/exceptions.md` | `rules/exceptions.md` | `EXC` |
   | `philosophy/performance.md` | `rules/performance.md` | `PERF` |
   | `philosophy/docstrings.md` | `rules/docstrings.md` | `DOC` |
   | `philosophy/testing.md` | `rules/testing.md` | `TST` |
   | `philosophy/documentation.md` | `rules/site.md` | `SITE` |
   | `philosophy/changelog.md` | `rules/changelog.md` | `CHG` |

   Frequent phrasings:

   | Found | Replace by |
   | --- | --- |
   | `.handbook/` in `_typos.toml` or similar lists | `.conventions/` |
   | link to the Handbook repository | link to `https://github.com/control-toolbox/ct-conventions` |
   | `WORKFLOWS.md` of the Handbook | `https://github.com/control-toolbox/CTActions/blob/main/docs/WORKFLOWS.md` |
   | `VITEPRESS-DOC.md`, "Plot image format" | `SITE12` |
   | `documentation.md`, self-reference | `SITE5` |
   | "Handbook tenet 2", "using, never import", `modules.md` rule | `MOD3` |
   | "fake types never inside functions", "recording fakes at module top level" | `TST3` |
   | "capability-gated test" contract or policy | `TST6` |
   | "alias/where bounds", `types-traits-interfaces.md#aliases-and-where` | `TYP3` |
   | "1-D = scalar", `dimension-and-shape.md` | `DIM1` |
   | "in-place buffer stays a vector" | `DIM2` |
   | "safe common denominator" | the migration note of `rules/dimension.md` (no identifier) |
   | "hot path vs. setup path", `performance.md` workflow | `PERF1` |
   | "Docstrings deferred (Handbook convention)" | "docstrings last" (`plan-template.md`) |
   | `.handbook` bump instructions in `CONTRIBUTING.md` | `git -C .conventions fetch --tags && git -C .conventions checkout <tag>` |

   For any other occurrence, find the matching rule with the correspondence table of
   [`BREAKING.md`](https://github.com/control-toolbox/ct-conventions/blob/main/BREAKING.md)
   in ct-conventions. If none fits, leave the line and report it.

   What may be edited: Markdown, comments and docstrings, `@testset` names, and strings
   printed by probe scripts (report the latter). Anything else is code: stop and report.

5. **`CHANGELOG.md`.** Under `[Unreleased]`, in `### 📚 Documentation`, add one entry:
   ct-conventions pinned in `.conventions/` at `$TAG`, `AGENTS.md` cites rules by
   identifier, references updated. Keep the `✅ Compatibility` footer ("No breaking
   changes").

6. **Check.** Paste each command's output as is:

   ```bash
   git submodule status                                    # .conventions at $TAG
   grep -rnIi handbook . --exclude-dir=.git --exclude-dir=.conventions --exclude=CHANGELOG.md --exclude=BREAKING.md | wc -l   # 0
   git diff --stat origin/main -- src ext                  # only comment lines, if any
   git diff origin/main -- src ext | grep '^[+-][^+-]' | grep -v '^[+-][[:space:]]*#' | wc -l   # 0
   git -C "$ROOT/$PKG" status --short | diff - "/tmp/$PKG-status-before.txt"   # no output
   ```

   Then run the package's `test/suite/meta` if it exists (see `method.md` §1 of
   ct-conventions), and the docs build in draft mode if `docs/make.jl` or a guide was
   touched.

## `AGENTS.md` template

````markdown
# <PKG> — Agent Navigation Guide

Quick-reference for any agent working on this repository.

---

## Repository Layout

<keep the package's own layout; add the `.conventions/` line>

```text
.conventions/  # ct-conventions (git submodule, pinned to a tag)
```

---

## Conventions

The coding conventions live in
[ct-conventions](https://github.com/control-toolbox/ct-conventions), vendored as a git
submodule in `.conventions/` and pinned to a SemVer release tag (`vX.Y.Z`). Read the
local files; if `.conventions/` is empty, run `git submodule update --init`.

**Pinned version: ct-conventions <TAG>.** Cite rules by identifier (`MOD3`, `TST1`), and
state the version you applied in plans and reports.

Read in this order:

| File | Content |
| --- | --- |
| [`rules/general.md`](.conventions/rules/general.md) | Rules that hold everywhere — **always read** |
| [`README.md`](.conventions/README.md) | Index of the rule files, one per area (`MOD`, `TYP`, `DIM`, `EXC`, `PERF`, `DOC`, `TST`, `SITE`, `CHG`) |
| [`method.md`](.conventions/method.md) | How to work: tests, coverage, docs build, output capture |
| [`plan-template.md`](.conventions/plan-template.md) | Template for a substantial piece of work |

How agents behave on these repositories is described in
[ct-agents](https://github.com/control-toolbox/ct-agents).

---

## Key Conventions

- **English everywhere** — code, docs, commits, issues, PRs (`GEN1`).
- **No top-level exports** — use `Package.Submodule.symbol` everywhere (`MOD8`).
- **Qualified imports** — `using Pkg: Pkg`, never bare `using Pkg`; `import` is never used (`MOD3`, `MOD4`).
- **Fake types at module top-level** — never inside test functions (`TST3`).
- **Structured errors** — seven typed exceptions; pick by the choice rule (`EXC1`).
- **Type stability enforced** — hot paths `@inferred`-clean, verified with JET (`TYP8`, `PERF1`–`PERF5`).
- **1-D is a scalar** — a one-dimensional state/control/variable is a `Number` (`DIM1`).
- **Plan before code** — a local agent proposes a plan ([`plan-template.md`](.conventions/plan-template.md)) and waits for confirmation before working on code. The process for remote agents is still to be defined.
- **Docstrings last** — written once the implementation is stable (`DOC1`–`DOC8`).
- **The human merges** — an agent never merges a pull request.
````

## Stop and report when

- `AGENTS.md` has a package-specific section that contradicts a rule;
- a Handbook reference has no matching rule in the correspondence table;
- a step would change code, not comments;
- the checks of step 6 fail;
- and in the cases listed in `harmonize.md`.
