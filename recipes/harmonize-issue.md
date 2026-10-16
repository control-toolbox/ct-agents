# Template: the description of a "Harmonize" issue

Part of [`harmonize.md`](harmonize.md). Every package has one "Harmonize" issue; its
description is this text, with `<PKG>` and `$REPO` replaced. Title: `[Chore] Harmonize
repository metadata and ct-conventions integration with CTBase`.

The descriptions written before the recipes asked for a git submodule of the
Handbook, which is wrong today (a submodule breaks `Pkg.add` on Julia 1.10). Step A of
`harmonize.md` replaces such a description by this one.

````markdown
## Goal

Harmonize <PKG> with the conventions of the control-toolbox ecosystem, following the [ct-agents harmonization recipes](https://github.com/control-toolbox/ct-agents/blob/main/recipes/harmonize.md). CTBase.jl is the reference.

## Scope — one PR (or settings change) per theme

| # | Theme | Recipe | What changes |
| --- | --- | --- | --- |
| 1 | Pin ct-conventions | [pin-conventions.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/pin-conventions.md) | the latest [ct-conventions](https://github.com/control-toolbox/ct-conventions) release pinned in `AGENTS.md` and read from a git-ignored clone in `.conventions/` (**not a git submodule**: it breaks `Pkg.add` on Julia 1.10); `AGENTS.md` cites rules by identifier; `CLAUDE.md` redirects to it; `CONTRIBUTING.md` and `_typos.toml` updated; every reference to the former Handbook replaced |
| 2 | Labels | [labels.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/labels.md) | the reference label set (`feature`, `dev`, `release`, …); package-specific and workflow labels kept; plan validated by the maintainer before it is applied |
| 3 | Issue and PR templates, code of conduct | [issue-templates.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/issue-templates.md) | the package's own `.github/ISSUE_TEMPLATE/`, PR template and `CODE_OF_CONDUCT.md` removed: the organization's, in [control-toolbox/.github](https://github.com/control-toolbox/.github), apply instead |
| 4 | Typo and lint configuration | [lint-and-automation.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/lint-and-automation.md) | `.JuliaFormatter.toml` (BlueStyle); the shared `.markdownlint.json` (`GEN2`), without `markdownlint-disable` comments |
| 5 | Obsolete automation | [lint-and-automation.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/lint-and-automation.md) | obsolete workflows removed (`AIHello.yml`); any other difference with CTBase's workflows reported, not aligned |

Progress is tracked in a comment below, ticked when each PR is merged.

## Acceptance criteria

- `AGENTS.md` pins a ct-conventions release; no git submodule in the tree; Julia 1.10 installs the package.
- No reference to the former Handbook outside `CHANGELOG.md` and `BREAKING.md`.
- `https://github.com/$REPO/issues/new/choose` offers the organization's five templates.
- The labels the templates and the workflows use exist.
- Package-specific workflows and documentation still work.
````
