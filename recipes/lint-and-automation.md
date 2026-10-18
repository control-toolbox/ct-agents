# Recipe: lint configuration and obsolete automation (themes 4 and 5)

Part of [`harmonize.md`](harmonize.md); read its ground rules and narration first.

Both themes are small. The reference is CTBase.jl; CTModels.jl was surveyed against it
(CTModels.jl#442) and differed only by a missing `.JuliaFormatter.toml`. One PR per
package covers both themes; if there is nothing to change, post a comment on the
"Harmonize" issue saying so, with the survey output, and open no PR.

## Inputs

`PKG`, `REPO`.

## Survey

From the package's worktree, with `$ROOT/CTBase` up to date on `main`:

```bash
REF=$ROOT/CTBase
for f in .JuliaFormatter.toml .markdownlint.json _typos.toml; do
  echo "## $f"; diff "$REF/$f" "$f" && echo identical
done
ls .github/workflows/ > /tmp/pkg.txt; ls "$REF/.github/workflows/" > /tmp/ref.txt
diff /tmp/ref.txt /tmp/pkg.txt   # < only in CTBase, > only in the package
grep -n "REPLACE_ME_DOCUMENTER_VITEPRESS_FAVICON\|logo: 'REPLACE_ME" docs/src/.vitepress/config.mts
ls docs/src/assets/ 2>/dev/null | grep -iE '^(logo|favicon)\.'
```

Paste the output in the first comment of the PR (or in the comment on the issue).

## Theme 4 — typo and lint configuration

| File | Expected | Action |
| --- | --- | --- |
| `.JuliaFormatter.toml` | `style = "blue"` | add it if missing; if it names another style, **stop and report** |
| `_typos.toml` | excludes `.conventions/` (theme 1) | nothing else: the word lists are package-specific |
| `.markdownlint.json` | exactly the content of [`GEN2`](https://github.com/control-toolbox/ct-conventions/blob/main/rules/general.md) (ct-conventions ≥ v0.4.0) | replace or create it; remove every `<!-- markdownlint-disable … -->` comment from the tracked `.md` files and list them |
| `formatter.lock` | — | nothing: the Formatter workflow writes it |
| `docs/src/.vitepress/config.mts` | no `logo` / favicon placeholder when the package ships no logo ([`site.md`](https://github.com/control-toolbox/ct-conventions/blob/main/rules/site.md), notes on the VitePress backend, "No logo / no favicon") | if `docs/src/assets/` has no `logo.{png,svg}` nor `favicon.ico`, delete the two lines `['link', { rel: 'icon', href: 'REPLACE_ME_DOCUMENTER_VITEPRESS_FAVICON' }],` and `logo: 'REPLACE_ME_DOCUMENTER_VITEPRESS',`; if the package ships a logo, leave them |

Do not fix what markdownlint reports with the new configuration: that is not part of
the theme. Count it in the survey comment (`npx markdownlint-cli2@0.18.1 $(git ls-files '*.md')`).

Why `.JuliaFormatter.toml`: the nightly Formatter workflow (CTActions `formatter.yml`) runs
`format(".", BlueStyle())`, with the style hard-coded. A local `format(".")` or the VS Code
formatter reads the file; without it they use the default style and disagree with CI.

Why the `config.mts` lines: DocumenterVitepress fills both placeholders from
`docs/src/assets/`, and when a file is missing it deletes the line from the built config but
warns on every build (`No logo.png file found`, `No favicon.ico file found`). The
control-toolbox packages ship no logo, and their favicon comes from the hosting site, so
deleting the lines gives the same site without the warnings. Check with a local docs build:
the two warnings are gone. Done in CTBase.jl#594 and CTFlows.jl#471.

## Theme 5 — obsolete automation

| Item | Action |
| --- | --- |
| `.github/workflows/AIHello.yml` | remove it (removed from CTBase in `5a82a55b`) |
| a workflow only in the package | **report**, do not remove: it may be package-specific (GPU runners, benchmarks) |
| `AddToProject.yml` missing | add it, copied from CTBase (it reads the organization secret `PROJECT_TOKEN`) |
| a CI job removed while its `run ci …` label is still used (the kkt GPU job, removed by mistake in CTSolvers and CTLie) | restore it from the commit before its removal, in a separate PR; say which commit removed it |
| another workflow only in CTBase | report |
| differences inside a shared workflow | ignore names, comments and whitespace; report any other difference, do not align it |

## PR

- Title: `chore: lint configuration and obsolete automation`, label `no changelog`
  (nothing changes for users of the package).
- Body: `Refs #N`, the survey, and one line per change.

## Stop and report when

- `.JuliaFormatter.toml` exists with another style;
- the survey finds a workflow you cannot classify.
