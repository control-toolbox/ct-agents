# Recipe: issue and PR templates, defined once for the organization (theme 3)

Part of [`harmonize.md`](harmonize.md); read its ground rules and narration first.

The issue templates and the PR template are the same in every repository, so they are
defined **once**, in the organization's `control-toolbox/.github` repository, and the
packages stop carrying copies. GitHub uses the organization's files for every repository
that has none of its own.

Two facts from the
[GitHub documentation](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
shape the recipe:

- The `.github` repository must be **public**.
- If a repository has **anything** in its own `.github/ISSUE_TEMPLATE/` folder, *including
  a `config.yml`*, none of the organization's templates is used for that repository. So a
  package must delete the whole folder, not only its templates.

The consequence is that the contact links of `config.yml` are the organization's: a
package-specific Discussions link cannot stay.

Depends on [`labels.md`](labels.md): the templates apply the labels `bug`, `feature`,
`documentation`, `dev` and `release`, which must exist in every repository (GitHub
ignores a label that does not exist).

## Part I — once, in `control-toolbox/.github`

Not per package. Run it once, in a worktree of that repository, as theme 3.

1. **Source.** The templates of CTBase.jl on `main` (they come from commit `fe5de01b`): `bug.yml`,
   `feature.yml`, `developer.md`, `documentation.md`, `release.md` and
   `pull_request_template.md`.
2. **Copy** them to the `.github` repository:

   ```text
   .github/ISSUE_TEMPLATE/bug.yml
   .github/ISSUE_TEMPLATE/feature.yml
   .github/ISSUE_TEMPLATE/developer.md
   .github/ISSUE_TEMPLATE/documentation.md
   .github/ISSUE_TEMPLATE/release.md
   .github/ISSUE_TEMPLATE/config.yml
   .github/pull_request_template.md
   ```

3. **`config.yml`.** `blank_issues_enabled: false`, and the contact links that make sense
   for the whole organization (the organization's Discussions). Remove the link that
   names a package.
4. **Make the content generic.** It must read correctly for any package: no package name,
   no link to a package-specific file. Known case: `release.md` says "Prepare a CTBase
   release" in `about:`; write "Prepare a release". The release steps differ between the
   packages today (compare `.github/ISSUE_TEMPLATE/release.md` in each: CTBase,
   CTModels, CTSolvers, CTFlows, CTLie, OptimalControl have different versions, CTDirect
   and CTParser have none). **Stop and ask the human** which release checklist is the
   reference before choosing one, and say so in a comment on the PR.
5. Open the PR on `control-toolbox/.github`, narrate it, and stop. The human merges.
   Record the merge in the tracking comment of every "Harmonize" issue.

## Part II — per package, after Part I is merged and theme 2 is done

1. Check that the labels of the reference set exist (`labels.md`). If not, stop.
2. In the worktree, remove everything the organization now provides:

   ```bash
   git rm -r .github/ISSUE_TEMPLATE
   git rm -f .github/pull_request_template.md   # if it exists
   ```

   A package that has a template that is not one of the organization's (for instance a
   specific form) keeps it **and the whole folder**, which disables the defaults: stop and
   report, the human decides.
3. **Check after the merge**, not before (the defaults apply to the default branch):

   ```bash
   curl -s "https://github.com/$REPO/issues/new/choose" | grep -o 'Bug report\|Feature request\|Developer\|Documentation suggestion\|Release' | sort -u
   ```

   Paste the result in a comment on the issue: the five templates must be offered.
   `https://github.com/$REPO/compare` must show the PR template when opening a PR.
4. `CHANGELOG.md`: no entry needed (no change for users of the package); the PR is
   labelled `no changelog`.

CTBase.jl follows Part II too: its templates become the organization's.

## Stop and report when

- the `.github` repository is private, or Part I is not merged;
- a label of the reference set is missing in the package;
- the package has a template of its own that the organization does not provide.
