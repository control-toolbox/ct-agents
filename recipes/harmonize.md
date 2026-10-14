# Recipe: harmonize the repositories of the ecosystem

Each package has an open issue "[Chore] Harmonize repository metadata and Handbook
integration with CTBase". Its scope is wide, so the work is split in **themes**, one
recipe and one pull request per theme and per package. This file holds what every theme
shares: the issues, the worktree, the narration on the PR, how to link and close, the
cleanup. Each theme recipe says only what is specific to it.

Meant to be run by a simple agent, one package and one theme at a time, in local or
remote mode.

## Themes

| # | Theme | Recipe | Shape | Depends on |
| --- | --- | --- | --- | --- |
| 1 | Pin ct-conventions, `AGENTS.md`, `CLAUDE.md` | [`pin-conventions.md`](pin-conventions.md) | one PR per package | — |
| 2 | Labels | [`labels.md`](labels.md) | a plan validated by the human, then repository settings (no PR) | — |
| 3 | Issue and PR templates, code of conduct | [`issue-templates.md`](issue-templates.md) | one PR on `control-toolbox/.github`, then one PR per package | 2, and the first PR merged |
| 4 | Typo and lint configuration | [`lint-and-automation.md`](lint-and-automation.md) | one PR per package with theme 5, or a comment if nothing to do | — |
| 5 | Obsolete repository automation | [`lint-and-automation.md`](lint-and-automation.md) | same PR as theme 4 | — |

Model for a package: [CTBase.jl#595](https://github.com/control-toolbox/CTBase.jl/pull/595)
(theme 1).

## Inputs

- `ROOT` — the directory that contains all the packages and `.worktrees/` (e.g.
  `/Users/ocots/Research/logiciels/dev/control-toolbox`).
- `PKG` — the package directory name under `ROOT` (e.g. `CTModels.jl`).
- `REPO` — its GitHub repository (e.g. `control-toolbox/CTModels.jl`).
- `THEME` — the theme recipe to run, and its parameters (e.g. `TAG` for theme 1).

## Ground rules

- Work in a fresh worktree created from `origin/main` (step B), never in the package
  directory itself: it may be on someone else's feature branch, with uncommitted changes.
  The worktree path is **`$ROOT/.worktrees/$PKG-<theme>`**, never inside `$ROOT/$PKG/`
  (a `.worktrees/` folder there shows up as untracked in the package).
- One branch and one PR per theme, named `chore/<theme>`. Commits, PR title, PR body and
  comments in English, `docs:` or `chore:` prefix.
- Open the PR and stop. **Never merge**: the human merges.
- Do not edit history: leave past entries of `CHANGELOG.md` and `BREAKING.md` as they
  are.
- No source change. If a step would touch code, stop and report.
- **One commit per step** of the theme recipe, pushed right after the step, so that the
  history follows the narration.
- Link every PR to the existing issues, comment on them, and narrate the work on the PR.
- **A check that finds nothing proves nothing until it is shown to find something.**
  Before the work, run each search on the unmodified code (`origin/main`) and report the
  count, which must be above zero when the thing searched for exists; after the work,
  report the count again. Never write commands with backslash-escaped quotes in a
  comment: they are copied and run as is.

## Protocol

### A. Find the issues about this work

Known on 2026-10-14 (search again, the list may have changed):

| Package | "Harmonize" issue |
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
  --search "Handbook OR AGENTS.md OR conventions OR submodule OR labels OR templates" \
  --json number,title,assignees --jq '.[]|"#\(.number) \(.title) [\([.assignees[].login]|join(","))]"'
```

Read each one. Do not start if the package already has an open PR for the same theme
(`gh pr list -R "$REPO" --search "<theme keywords>"`): report it instead.

### B. Worktree

Worktrees live in `$ROOT/.worktrees/`, named `<PKG>-<theme>`.

```bash
git -C "$ROOT/$PKG" status --short > "/tmp/$PKG-status-before.txt"   # snapshot, to compare at the end
git -C "$ROOT/$PKG" fetch origin
git -C "$ROOT/$PKG" worktree add "$ROOT/.worktrees/$PKG-<theme>" -b chore/<theme> origin/main
cd "$ROOT/.worktrees/$PKG-<theme>"
```

Do not commit or edit anything in `$ROOT/$PKG` itself. At the end, `git -C "$ROOT/$PKG"
status --short` must be identical to the snapshot; paste both in the final comment.

### C. Run the theme recipe

Follow its steps, and narrate them (below).

### D. Link the PR to the issues

The "Harmonize" issue covers all the themes, so a PR only **`Refs #N`**. Never write
`Closes` for it, and never close an issue you did not fully resolve. The human closes the
"Harmonize" issue once every theme is done.

Keep one **tracking comment** on the "Harmonize" issue, created by the first theme run
and edited by the following ones:

```markdown
Harmonization progress (recipes: https://github.com/control-toolbox/ct-agents/tree/main/recipes)
- [x] 1. Pin ct-conventions — <PR URL>
- [ ] 2. Labels
- [ ] 3. Issue and PR templates
- [ ] 4. Typo and lint configuration — not specified yet
- [ ] 5. Obsolete automation — not specified yet
```

Tick a theme when its PR is **merged**, not when it is opened. Also comment on each
related issue touched by a PR, with the PR link.

### E. Mark the PR ready

After the final comment (below), mark the PR ready for review, and report to the user:
files changed, anything left as is and why, check results, PR URL, issues commented.

### F. Clean up

Once the human has merged (not before):

```bash
cd "$ROOT/$PKG"
git worktree remove --force "$ROOT/.worktrees/$PKG-<theme>"   # --force: it may contain a submodule
git worktree prune
git branch -D chore/<theme>
```

## Narrating the work

The work is reviewed afterwards by reading the PR comments and then checking them against
the diff. So the agent explains what it does as it goes, in English, as comments on the
PR (not in the description, which stays short; one line per paragraph, GitHub renders
line breaks; it ends with the generation note).

1. **Open the PR as a Draft right after the first commit**, and push after each following
   step.
2. **Post one comment per step** of the theme recipe, right after pushing that step's
   commit. Comments posted all at once at the end are not narration:

   ```markdown
   **Step <N> — <title>.**
   Done: <what was changed, in one or two sentences>.
   Files: <list of files touched>.
   Decisions: <each choice that was not dictated by the recipe, and why>.
   Left as is: <anything found and not changed, with file:line and the reason>.
   Check: `<command>` → <actual result>.
   ```

3. **When a step handles a list** (references to replace, labels to merge), list every
   item as `old → new (reason)`, and every item left as is with the reason. The reviewer
   must be able to tell that none was missed.
4. **Final comment, before marking the PR ready:** a summary with the checklist of the
   recipe steps (done / not applicable / stopped), the output of the checks pasted as
   is, the issues linked and commented, and the open questions for the human.
5. Never describe a check that was not run. If a check could not run (no `meta` suite, no
   Julia available), say so.
6. **A claim in a comment that the diff does not show is a defect.**

## Stop and report when

- a PR for the same theme is already open on the package;
- the theme recipe says to stop, or a step would change code, not comments;
- a check fails;
- the theme is not specified.

Whenever you stop, comment on the "Harmonize" issue with the reason and the state of the
worktree, so that the stop is visible there.
