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

## Part I — done, in `control-toolbox/.github`

The organization's templates live in
[`control-toolbox/.github/.github/`](https://github.com/control-toolbox/.github/tree/main/.github)
(control-toolbox/.github#1, #2). They come from CTBase.jl, the reference chosen by the
maintainer, made generic: a `Package` dropdown instead of CTBase's areas, only the
organization's Discussions link in `config.yml`, "Prepare a release", no assignee. The organization also
holds `CODE_OF_CONDUCT.md` (Contributor Covenant 2.0, control-toolbox/.github#3), identical to
the copies some packages carry. A change to a template is made there, once, for every
package.

## Part II — per package, after Part I is merged and theme 2 is done

1. Check that the labels of the reference set exist (`labels.md`). If not, stop.
2. In the worktree, remove everything the organization now provides:

   ```bash
   git rm -r .github/ISSUE_TEMPLATE
   git rm -f .github/pull_request_template.md   # if it exists
   git rm -f CODE_OF_CONDUCT.md                 # if it exists
   ```

   A "blank issue" template (`blank-issue.md`, "Blank template for general issue") is
   removed too: blank issues are disabled organization-wide and `developer.md` covers
   general internal work. Any other template that is not one of the organization's (for
   instance a specific form) would keep the whole folder, which disables the defaults:
   stop and report, the human decides (the usual answer is to add it to
   `control-toolbox/.github` for every package).

   In the PR, list each removed file with the organization template that replaces it.
3. **Check after the merge**, not before (the defaults apply to the default branch).
   The PR template, from the API:

   ```bash
   gh api "repos/$REPO/community/profile" --jq .files.pull_request_template.html_url
   ```

   must point to `control-toolbox/.github`; the same holds for `.files.code_of_conduct_file`. The issue templates have no API and the
   page `https://github.com/$REPO/issues/new/choose` redirects to the login page without
   a session, so `curl` cannot check them: ask the human to open that page and confirm
   that the five templates (Bug report, Feature request, Developer, Documentation
   suggestion, Release) are offered, and record the answer in a comment on the issue.
4. `CHANGELOG.md`: no entry needed (no change for users of the package); the PR is
   labelled `no changelog`.

CTBase.jl follows Part II too: its templates become the organization's.

## Stop and report when

- a label of the reference set is missing in the package;
- the package has a template of its own that the organization does not provide.
