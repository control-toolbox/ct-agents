# Recipe: pin ct-conventions in a package

Align one control-toolbox package on [ct-conventions](https://github.com/control-toolbox/ct-conventions):
a pinned `.conventions/` submodule, an `AGENTS.md` that cites rule identifiers, a
`CLAUDE.md` that only redirects. Mechanical and checkable: meant to be run by a simple
agent, one package at a time, in local or remote mode.

Model to copy: [CTBase.jl#595](https://github.com/control-toolbox/CTBase.jl/pull/595).

## Inputs

- `PKG` — the package directory (e.g. `CTModels.jl`).
- `TAG` — the ct-conventions tag to pin (e.g. `v0.2.0`).

## Ground rules

- Work in a fresh worktree or clone from `main`; never on a feature branch of someone
  else, never with uncommitted changes around.
- Branch `chore/pin-ct-conventions`. Commits and PR title in English, `docs:` prefix.
- Open a PR and stop. **Never merge**: the human merges.
- Do not edit history: leave past entries of `CHANGELOG.md` and `BREAKING.md` as they
  are, even where they say "Handbook".
- No source change. If a step would touch `src/` or `ext/` code (not comments), stop and
  report.

## Steps

1. **Submodule.**

   ```bash
   git checkout main && git pull
   git checkout -b chore/pin-ct-conventions
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

4. **References to the Handbook.** Find them:

   ```bash
   grep -rnI "Handbook\|\.handbook" . --exclude-dir=.git --exclude-dir=.conventions \
     | grep -v "^./CHANGELOG.md\|^./BREAKING.md"
   ```

   and replace each by its new form:

   | Found | Replace by |
   | --- | --- |
   | `.handbook/` in `_typos.toml` or `.gitignore`-like lists | `.conventions/` |
   | link to the Handbook repository | link to `https://github.com/control-toolbox/ct-conventions` |
   | `WORKFLOWS.md` of the Handbook | `https://github.com/control-toolbox/CTActions/blob/main/docs/WORKFLOWS.md` |
   | `VITEPRESS-DOC.md`, "Plot image format" | `ct-conventions SITE12` |
   | `documentation.md`, self-reference | `ct-conventions SITE5` |
   | "Handbook tenet 2" / "using, never import" | `MOD3` |
   | "Handbook: fake types never inside functions" | `TST3` |
   | `philosophy/types-traits-interfaces.md#aliases-and-where` | `TYP3` |
   | `.handbook` bump instructions in `CONTRIBUTING.md` | `git -C .conventions fetch --tags && git -C .conventions checkout <tag>` |

   For any other occurrence, find the matching rule with the correspondence table of
   [`BREAKING.md`](https://github.com/control-toolbox/ct-conventions/blob/main/BREAKING.md)
   in ct-conventions. If none fits, leave the line and report it.

5. **`CHANGELOG.md`.** Under `[Unreleased]`, in `### 📚 Documentation`, add one entry:
   ct-conventions pinned in `.conventions/` at `$TAG`, `AGENTS.md` cites rules by
   identifier, references updated. Keep the `✅ Compatibility` footer ("No breaking
   changes").

6. **Check.**

   ```bash
   git submodule status                      # .conventions at $TAG
   grep -rnI "\.handbook" . --exclude-dir=.git --exclude-dir=.conventions \
     | grep -v "^./CHANGELOG.md\|^./BREAKING.md"     # nothing
   git diff --stat main -- src ext            # no code change (comments only, if any)
   ```

   Then run the package's `test/suite/meta` if it exists (see `method.md` §1 of
   ct-conventions), and the docs build in draft mode if `docs/make.jl` was touched.

7. **PR.** Commit, push, open the PR against `main`, one line per paragraph in the body
   (GitHub renders line breaks), ending with the generation note. Report: the files
   changed, anything left as is and why, and the check results. Do not merge.

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
- **Docstrings last** — written once the implementation is stable (`DOC1`–`DOC8`).
- **The human merges** — an agent never merges a pull request.
````

## Stop and report when

- the package is not on `main`, or has uncommitted changes;
- `AGENTS.md` has a package-specific section that contradicts a rule;
- a Handbook reference has no matching rule in the correspondence table;
- a step would change code, not comments;
- the checks of step 6 fail.
