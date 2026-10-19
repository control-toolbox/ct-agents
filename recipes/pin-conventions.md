# Recipe: pin ct-conventions in a package (theme 1)

Part of [`harmonize.md`](harmonize.md): read its ground rules, protocol (issues, worktree,
linking) and narration first. This file only holds what is specific to theme 1.

Align one control-toolbox package on [ct-conventions](https://github.com/control-toolbox/ct-conventions):
a tag pinned in `AGENTS.md` and read from a git-ignored clone in `.conventions/`, an
`AGENTS.md` that cites rule identifiers, a `CLAUDE.md` that only redirects. Mechanical
and checkable.

**Never a git submodule.** On Julia 1.10, `Pkg.add` fails on a package whose tree contains
a submodule (`GitError: cannot get submodules without a working tree`), and so does the
registry's AutoMerge: it blocked the registration of CTBase v0.30.9
(JuliaRegistries/General#171784, fixed in CTBase.jl#601).

Model to copy: [CTBase.jl#595](https://github.com/control-toolbox/CTBase.jl/pull/595),
then [CTBase.jl#601](https://github.com/control-toolbox/CTBase.jl/pull/601), which
replaced its submodule by the clone.

## Inputs

`PKG`, `REPO`, and `TAG` — the ct-conventions tag to pin (the latest release: `gh release view -R control-toolbox/ct-conventions --json tagName --jq .tagName`).
Branch and worktree: `chore/pin-ct-conventions`, `.worktrees/$PKG-pin-conventions`.

## Steps

1. **Local clone, git-ignored.** In the worktree (protocol step B):

   ```bash
   # only if the package has a submodule (.handbook or .conventions):
   for f in .handbook .conventions .gitmodules; do git rm -rfq --ignore-unmatch "$f"; done
   rm -rf .handbook .conventions
   printf '\n# ct-conventions, cloned at the tag pinned in AGENTS.md\n.conventions/\n' >> .gitignore
   git -c advice.detachedHead=false clone --quiet --depth 1 --branch "$TAG" \
     https://github.com/control-toolbox/ct-conventions.git .conventions
   ```

   `.gitmodules` must not exist afterwards, unless it lists another submodule: then stop
   and report.

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
   | `.handbook` bump instructions in `CONTRIBUTING.md` | the `Developer Resources` text of step 5 |

   For any other occurrence, find the matching rule with the correspondence table of
   [`BREAKING.md`](https://github.com/control-toolbox/ct-conventions/blob/main/BREAKING.md)
   in ct-conventions. If none fits, leave the line and report it.

   What may be edited: Markdown, comments and docstrings, `@testset` names, and strings
   printed by probe scripts (report the latter). Anything else is code: stop and report.

   **A reference that states a fact about the code is not evidence of that fact.** Some
   references are not citations but claims ("Docstrings deferred (Handbook convention)",
   "not yet implemented", "see the Handbook, this is a known deviation"). Check the claim
   against the code before copying it into a new citation, and say how you checked in the
   step 4 comment. Example: five files of CTModels carried "Docstrings deferred" while
   every symbol had a docstring (`Base.Docs.meta(CTModels.PlotCase)`; the comment was
   written in the same commit as the docstrings). If the claim is false, remove the
   comment; if it is true, keep it, cite the rule, and list it in the final comment as a
   standing deviation for the human to decide on; if you cannot tell, leave it and report
   it. Never invent an explanation.

5. **`CONTRIBUTING.md`.** The contributor guide must say how to get and bump the
   conventions. If it has no `## Developer Resources` section, add this one at the end,
   after a `---` separator (copy it from CTBase.jl); if it has one, make sure it contains
   this text and replace any older text about the Handbook:

   ```markdown
   ---

   ## Developer Resources

   The [ct-conventions](https://github.com/control-toolbox/ct-conventions) repository (coding rules, method, plan template) is pinned to a SemVer release tag in `AGENTS.md` ("Pinned version"). Read it from a local clone in `.conventions/`, which git ignores; `AGENTS.md` gives the clone command.

   It is not a git submodule: `Pkg.add` on Julia 1.10 fails on a package whose tree contains one (`cannot get submodules without a working tree`). To bump it, change the tag in `AGENTS.md` (the "Pinned version" line and the clone command), then re-clone.
   ```

   The `README.md` is generated from `CONTRIBUTING.md` by the UpdateReadme workflow: do
   not edit `README.md` by hand.

6. **`_typos.toml`.** The local clone must not be spell-checked. In the `extend-exclude`
   list of `[files]`, add `".conventions/",` (CTBase.jl has it). If the file or the list
   does not exist, create it with that entry only, and say so in the comment. Do not touch
   the other entries.

7. **`CHANGELOG.md`.** Under `[Unreleased]`, in `### 📚 Documentation`, add one entry:
   ct-conventions pinned at `$TAG` in `AGENTS.md`, `AGENTS.md` cites rules by
   identifier, references updated. Keep the `✅ Compatibility` footer ("No breaking
   changes").

8. **Check.** Paste each command's output as is:

   ```bash
   git ls-files -s | grep -c '^160000'                     # 0: no submodule in the tree
   git check-ignore -q .conventions && echo ignored         # ignored
   git -C .conventions describe --tags                     # $TAG
   grep -c 'ct-conventions' CONTRIBUTING.md                # at least 1
   grep -n 'conventions' _typos.toml                       # the .conventions/ exclusion
   grep -rnIi handbook . --exclude-dir=.git --exclude-dir=.conventions --exclude=CHANGELOG.md --exclude=BREAKING.md | wc -l   # 0
   git diff --stat origin/main -- src ext                  # only comment lines, if any
   git diff origin/main -- src ext | grep '^[+-][^+-]' | grep -v '^[+-][[:space:]]*#' | wc -l   # 0
   git -C "$ROOT/$PKG" status --short | diff - "/tmp/$PKG-status-before.txt"   # no output
   ```

   Then check that Julia 1.10 can install the pushed branch (the registry's lowest
   version), in an empty depot:

   ```bash
   d=$(mktemp -d); JULIA_DEPOT_PATH=$d julia +1.10 --project=$d -e \
     'using Pkg; Pkg.add(url="https://github.com/'"$REPO"'", rev="chore/pin-ct-conventions"); println("OK")'
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
.conventions/  # ct-conventions, cloned at the pinned tag (git-ignored)
.ct-agents/    # ct-agents, cloned at the pinned tag (git-ignored)
```

---

## Conventions

The coding conventions live in
[ct-conventions](https://github.com/control-toolbox/ct-conventions), pinned below to a
SemVer release tag (`vX.Y.Z`). Read them from a local clone in `.conventions/`
(git-ignored, not a submodule). If it is missing, or `git -C .conventions describe --tags`
is not the pinned tag, (re)create it:

```bash
rm -rf .conventions
git -c advice.detachedHead=false clone --quiet --depth 1 --branch <TAG> https://github.com/control-toolbox/ct-conventions.git .conventions
```

**Pinned version: ct-conventions <TAG>.** Cite rules by identifier (`MOD3`, `TST1`), and
state the version you applied in plans and reports.

Read in this order:

| File | Content |
| --- | --- |
| [`rules/general.md`](.conventions/rules/general.md) | Rules that hold everywhere — **always read** |
| [`README.md`](.conventions/README.md) | Index of the rule files, one per area (`MOD`, `TYP`, `DIM`, `EXC`, `PERF`, `DOC`, `TST`, `SITE`, `CHG`, `REL`) |
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
- **The human merges** — an agent never merges a pull request, except a release PR in `autonomous` mode, and then only with `release.jl merge` (see below).

---

## Recurring tasks

How a recurring task is done lives in [ct-agents](https://github.com/control-toolbox/ct-agents),
pinned below to a SemVer release tag (`vX.Y.Z`). Read it from a local clone in `.ct-agents/`
(git-ignored, not a submodule). If it is missing, or `git -C .ct-agents describe --tags` is
not the pinned tag, (re)create it:

```bash
rm -rf .ct-agents
git -c advice.detachedHead=false clone --quiet --depth 1 --branch <AGENTS-TAG> https://github.com/control-toolbox/ct-agents.git .ct-agents
```

**Pinned version: ct-agents <AGENTS-TAG>.** State it, with the ct-conventions version, in
plans and reports.

- **Release.** When asked, in any language, to prepare a release, follow
  [`recipes/release.md`](.ct-agents/recipes/release.md) (rules `REL1`–`REL7`). Stable or
  beta is the human's choice: if the request does not say, ask; never infer it. State in
  the proposal the versions you apply: ct-agents, ct-conventions, and the commit of the
  CTActions clone `release.jl` runs from.
````

## Stop and report when

- `AGENTS.md` has a package-specific section that contradicts a rule;
- a Handbook reference has no matching rule in the correspondence table;
- a step would change code, not comments;
- the checks of step 8 fail;
- and in the cases listed in `harmonize.md`.

## Bumping an existing pin

When a package already pins an older tag, the PR is small. Each pin has its own lines, so
that bumping one never touches the other:

```bash
# ct-conventions
sed -i '' -E "s/ct-conventions v[0-9]+\.[0-9]+\.[0-9]+\./ct-conventions $TAG./; s#--branch v[0-9]+\.[0-9]+\.[0-9]+ (https://github.com/control-toolbox/ct-conventions)#--branch $TAG \1#" AGENTS.md
# ct-agents (the clone command and the "Pinned version" line)
sed -i '' -E "s/ct-agents v[0-9]+\.[0-9]+\.[0-9]+\./ct-agents $AGENTS_TAG./; s#--branch v[0-9]+\.[0-9]+\.[0-9]+ (https://github.com/control-toolbox/ct-agents)#--branch $AGENTS_TAG \1#" AGENTS.md
rm -rf .ct-agents && git -c advice.detachedHead=false clone --quiet --depth 1 --branch "$AGENTS_TAG" \
  https://github.com/control-toolbox/ct-agents.git .ct-agents
rm -rf .conventions && git -c advice.detachedHead=false clone --quiet --depth 1 --branch "$TAG" \
  https://github.com/control-toolbox/ct-conventions.git .conventions
```

If the package still has a `.conventions` submodule, do step 1 first.

Check the package against the rules the new release adds or tightens (its `CHANGELOG.md`
entry), say the result in the PR body, and add a changelog line — or, if the previous pin
is still under `[Unreleased]`, change its version instead.

From ct-conventions v0.7.0 on, the bump also brings `AGENTS.md` to the template above: the
`REL` prefix in the index, the merge exception in `Key Conventions`, and the `Recurring
tasks` section; and it adds the release callers of
[CTActions](https://github.com/control-toolbox/CTActions/blob/main/docs/README.md),
`ReleaseGate.yml` and `ReleaseTag.yml`, and moves `Breakage.yml` to `breakage-suite.yml`,
copied from CTBase.jl.
