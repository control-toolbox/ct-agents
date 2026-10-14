# Recipe: harmonize the labels of a package (theme 2)

Part of [`harmonize.md`](harmonize.md); read its ground rules and narration first.

Labels are repository **settings**, not files: there is no PR to review, and deleting a
label removes it from every issue that carries it. So this recipe produces a **plan**,
the human validates it, and only then is it applied.

The goal is a clean, consistent set, **not** the same set everywhere: a package keeps the
labels that make sense for it. CTBase has no GPU code, so it has no GPU run labels, and
that is right.

## Inputs

`PKG`, `REPO`.

## The reference set

The labels of CTBase.jl, which the issue and PR templates rely on
([`issue-templates.md`](issue-templates.md)). Every package must have these:

`automated pr`, `bug`, `dependencies`, `dev`, `documentation`, `duplicate`, `feature`,
`formatting`, `good first issue`, `help wanted`, `no changelog`, `question`, `release`,
`wontfix`, and the run labels its workflows use (`run breakage`, `run documentation`,
`run ci …`, see `.github/workflows/`).

CTBase's own run label is `run ci github-runner`. Other packages have run labels for the
runners they use (`run ci kkt-runner`, `run ci occidata-runner`, `run probe`, …): they
are wired in the workflows (`WORKFLOWS.md` of CTActions), so keep them as they are.

## Steps

1. **Survey.**

   ```bash
   gh label list -R "$REPO" --limit 200 --json name,description,color
   # number of issues and PRs per label
   gh issue list -R "$REPO" --state all --limit 1000 --json labels \
     --jq '[.[].labels[].name]|group_by(.)|map({label:.[0],n:length})|sort_by(-.n)[]|"\(.n)\t\(.label)"'
   ```

2. **Find what depends on each label** before proposing to rename or delete it:

   ```bash
   grep -rn "<label name>" .github/ docs/ CONTRIBUTING.md README.md 2>/dev/null
   ```

   A label named in a workflow (`if: contains(... 'run ci …')`), in `dependabot.yml`
   (`github_actions`, `dependencies`) or in an issue template must not be renamed or
   deleted without changing that file: report it instead.

3. **Write the plan** as a table, one row per label to touch, nothing else:

   | Action | Label | Target | Issues affected | Reason |
   | --- | --- | --- | --- | --- |
   | create | `feature` | — | 0 | in the reference set, used by the templates |
   | rename | `internal dev` | `dev` | 14 | reference name; the issues keep it |
   | merge | `refactor` into `refactoring` | `refactoring` | 3 | duplicate |
   | keep | `run ci occidata-runner` | — | — | used by `CI.yml` |
   | propose delete | `march2023` | — | 2 | obsolete; the human decides |

   Rules for the plan:
   - **Rename** rather than create-and-delete: a rename keeps the label on its issues.
   - **Merge** duplicates (`enhancement` / `feature`, `refactor` / `refactoring`,
     `tests` / `unit tests` / `testing`) by relabelling their issues, then deleting the
     emptied label. Prefer the reference name when there is one.
   - **Never delete** a label that is still on issues without the human's approval,
     even an apparently useless one (`invalid`, `help sed`, `march2023`): propose it.
   - Do not touch a package-specific label (`gpu`, `roadmap`, `application`,
     `breaking-change`, `migration`, …) except to list it as "keep".
   - Keep names, descriptions and colors consistent with CTBase for the reference set.

4. **Post the plan** as a comment on the "Harmonize" issue (the narration of this theme,
   since there is no PR), then **stop and wait for the human's answer**. Do not apply
   anything before it.

5. **Apply** what the human validated, and only that:

   ```bash
   gh label create "feature" -R "$REPO" --color <hex> --description "<text>"
   gh label edit "internal dev" -R "$REPO" --name "dev"
   # merge: relabel, then delete the emptied label
   for n in $(gh issue list -R "$REPO" --label "refactor" --state all --limit 1000 --json number --jq '.[].number'); do
     gh issue edit "$n" -R "$REPO" --add-label "refactoring" --remove-label "refactor"
   done
   gh label delete "refactor" -R "$REPO" --yes
   ```

6. **Check.** Compare the final list with the validated plan, and with the reference set;
   paste both in a comment on the issue.

## Stop and report when

- a label to rename or delete is named in a workflow, `dependabot.yml` or a template;
- the plan touches a label the human did not validate;
- the survey cannot count the issues (rate limit, permissions): report instead of
  guessing.
