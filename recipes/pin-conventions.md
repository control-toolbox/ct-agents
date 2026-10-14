# Recipe: pin ct-conventions in a package

Align one control-toolbox package on [ct-conventions](https://github.com/control-toolbox/ct-conventions):
a pinned `.conventions/` submodule, an `AGENTS.md` that cites rule identifiers, a
`CLAUDE.md` that only redirects. Mechanical and checkable: meant to be run by a simple
agent, one package at a time, in local or remote mode.

Model to copy: [CTBase.jl#595](https://github.com/control-toolbox/CTBase.jl/pull/595).

## Inputs

- `PKG` — the package directory name under `control-toolbox/` (e.g. `CTModels.jl`).
- `REPO` — its GitHub repository (e.g. `control-toolbox/CTModels.jl`).
- `TAG` — the ct-conventions tag to pin (e.g. `v0.2.0`).

## Ground rules

- Work in a fresh worktree created from `origin/main` (step 1), never in the package
  directory itself: it may be on someone else's feature branch, with uncommitted changes.
- Branch `chore/pin-ct-conventions`. Commits, PR title, PR body and issue comments in
  English, `docs:` prefix.
- Open a PR and stop. **Never merge**: the human merges.
- Do not edit history: leave past entries of `CHANGELOG.md` and `BREAKING.md` as they
  are, even where they say "Handbook".
- No source change. If a step would touch code (see the end of step 4), stop and report.
- Always link the PR to the existing issues and comment on them (step 0 and step 7).

## Steps

0. **Find the issues about this work.** Each package has an open issue
   "[Chore] Harmonize repository metadata and Handbook integration with CTBase". It asks
   for much more than this recipe covers: labels, issue and PR templates, the Handbook
   submodule, the guidance files (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`), typo/lint
   configuration, obsolete automation. This recipe covers only the submodule, the guidance
   files and the references.

   Known on 2026-10-14 (search again, the list may have changed):

   | Package | Issue |
   | --- | --- |
   | CTModels.jl | control-toolbox/CTModels.jl#442 |
   | CTSolvers | control-toolbox/CTSolvers.jl#246 |
   | CTFlows.jl | control-toolbox/CTFlows.jl#460 |
   | CTDirect.jl | control-toolbox/CTDirect.jl#642 |
   | CTParser.jl | control-toolbox/CTParser.jl#358 |
   | CTLie | control-toolbox/CTLie#48 |
   | OptimalControl | control-toolbox/OptimalControl.jl#982 |

   Also list the related open issues, without closing them: an "Enforce Handbook tenet 2:
   `using`, never `import`" issue is open in CTParser.jl (#303) and OptimalControl.jl
   (#831); `[Roadmap]` issues may mention the Handbook too.

   ```bash
   gh issue list -R "$REPO" --state open --limit 100 \
     --search "Handbook OR AGENTS.md OR conventions OR submodule" \
     --json number,title,assignees --jq '.[]|"#\(.number) \(.title) [\([.assignees[].login]|join(","))]"'
   ```

   Read each one. Note which of its scope items this PR will cover. Do not start if the
   package already has an open PR for the same work (`gh pr list -R "$REPO" --search
   "conventions OR Handbook"`): report it instead.

1. **Worktree and submodule.** Worktrees live in `control-toolbox/.worktrees/`, named
   `<PKG>-<topic>`.

   ```bash
   cd "<control-toolbox>/$PKG"
   git fetch origin
   git worktree add "../.worktrees/$PKG-pin-conventions" -b chore/pin-ct-conventions origin/main
   cd "../.worktrees/$PKG-pin-conventions"
   # only if the package has a .handbook submodule:
   git submodule deinit -f .handbook && git rm -f .handbook && rm -rf .git/modules/.handbook
   git submodule add https://github.com/control-toolbox/ct-conventions.git .conventions
   git -C .conventions checkout "$TAG"
   ```

   Do not commit or edit anything in `<control-toolbox>/$PKG` itself.

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

6. **Check.**

   ```bash
   git submodule status                      # .conventions at $TAG
   grep -rnI "\.handbook" . --exclude-dir=.git --exclude-dir=.conventions \
     | grep -v "^./CHANGELOG.md\|^./BREAKING.md"     # nothing
   git diff --stat origin/main -- src ext     # no code change (comments only, if any)
   ```

   Then run the package's `test/suite/meta` if it exists (see `method.md` §1 of
   ct-conventions), and the docs build in draft mode if `docs/make.jl` was touched.

7. **PR and issues.** Commit in small commits, push, and open the PR against `main`. One
   line per paragraph in the body (GitHub renders line breaks), ending with the generation
   note. In the body, link the issue of step 0:

   - `Refs #N` for the "Harmonize" issue — **not** `Closes`, unless this PR covers every
     item of its scope (labels, templates, PR template, submodule, guidance files, typo
     configuration, obsolete automation). In practice it does not: leave the issue open.
   - `Refs #M` for each related issue this PR touches. Never close an issue you did not
     fully resolve.

   Then comment on each linked issue, in English:

   ```markdown
   Opened <PR URL>: pins ct-conventions <TAG> in `.conventions/`, rewrites `AGENTS.md` to
   cite rule identifiers, makes `CLAUDE.md` a redirect and updates the references to the
   Handbook.
   Covered from the scope: submodule, guidance files, typo configuration.
   Not covered here: <the remaining items, e.g. labels, issue templates, PR template>.
   ```

   Also add a comment on the PR itself listing the issues it relates to, so that the link
   is visible from both sides.

   Report to the user: the files changed, anything left as is and why, the check results,
   the PR URL and the issues commented. Do not merge.

8. **Clean up.** Once the human has merged (not before):

   ```bash
   cd "<control-toolbox>/$PKG"
   git worktree remove --force "../.worktrees/$PKG-pin-conventions"   # --force: it contains a submodule
   git worktree prune
   git branch -D chore/pin-ct-conventions
   ```

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

- a PR for the same work is already open on the package;
- `AGENTS.md` has a package-specific section that contradicts a rule;
- a Handbook reference has no matching rule in the correspondence table;
- a step would change code, not comments;
- the checks of step 6 fail.

Whenever you stop, comment on the "Harmonize" issue with the reason (and the state of the
worktree), so that the stop is visible there.
