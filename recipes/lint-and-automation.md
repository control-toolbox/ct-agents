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
```

Paste the output in the first comment of the PR (or in the comment on the issue).

## Theme 4 — typo and lint configuration

| File | Expected | Action |
| --- | --- | --- |
| `.JuliaFormatter.toml` | `style = "blue"` | add it if missing; if it names another style, **stop and report** |
| `_typos.toml` | excludes `.conventions/` (theme 1) | nothing else: the word lists are package-specific |
| `.markdownlint.json` | same as CTBase | if it differs or is missing, report; do not change it |
| `formatter.lock` | — | nothing: the Formatter workflow writes it |

Why `.JuliaFormatter.toml`: the nightly Formatter workflow (CTActions `formatter.yml`) runs
`format(".", BlueStyle())`, with the style hard-coded. A local `format(".")` or the VS Code
formatter reads the file; without it they use the default style and disagree with CI.

## Theme 5 — obsolete automation

| Item | Action |
| --- | --- |
| `.github/workflows/AIHello.yml` | remove it (removed from CTBase in `5a82a55b`) |
| a workflow only in the package | **report**, do not remove: it may be package-specific (GPU runners, benchmarks) |
| a workflow only in CTBase | report |
| differences inside a shared workflow | ignore names, comments and whitespace; report any other difference, do not align it |

## PR

- Title: `chore: lint configuration and obsolete automation`, label `no changelog`
  (nothing changes for users of the package).
- Body: `Refs #N`, the survey, and one line per change.

## Stop and report when

- `.JuliaFormatter.toml` exists with another style;
- the survey finds a workflow you cannot classify.
