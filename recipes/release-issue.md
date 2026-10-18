# Templates: the texts of a release

Part of [`release.md`](release.md). The proposal shown to the human, the description of
the tracking issue, and the comments posted on it, one per phase. Replace the `<…>`
placeholders; keep one line per paragraph (GitHub renders line breaks).

**None of these texts mentions the registration bot** (`REL6`), except the registration
comment, which builds the mention when it is posted. A text that quotes a command must not
contain it either: the commands of [`release.md`](release.md) are written so that they
never do.

## The state marker

Every comment on the tracking issue ends with one line, invisible on GitHub:

```markdown
<!-- ct-release phase=<0-8> kind=<stable|beta> version=<X.Y.Z[-beta]> mode=<supervised|autonomous> pr=<N|-> merge=<sha|-> state=<done|stopped|waiting> -->
```

The marker is for the reader. A session that resumes a release does not trust it: it runs
`release.jl status`, which recomputes the phase from the facts.

## Proposal

Shown in the session in phase 1 (in remote mode, posted on the issue of the trigger), then
posted as the first comment of the tracking issue once validated.

````markdown
## Release proposal — <NAME> v<VERSION> (<stable|beta>)

**Version.** Last release `<LAST>`; last version in General `<LAST_STABLE>`. Since `<PREV>`: <breaking change(s): list | new features | fixes only>. By `REL1` (<0.Y.Z: the breaking component is Y | X ≥ 1: …>) and `REL2` (<the triple must be above every one released: …>), the version is **`<VERSION>`**.
**Jump in General (`REL3`).** <none | `<LAST_STABLE>` → `<VERSION>`: skipped `<…>`, all betas in ct-registry: `[merge approved]` needed on the General PR>.
**Resolves from General alone (`REL4`).** <✅ | not checked: beta>.

### What changes since `<PREV>`

- <one line per change, from the merged PRs and `[Unreleased]`, grouped as in `CHG2`>
- Breaking: <none | each one, with its `BREAKING.md` entry>
- Not in `[Unreleased]` yet, added by the release PR: <PRs, or none>
- Open PRs, not in this release: <#N title, or none>

### What the agent will do

1. Open the tracking issue `[RELEASE] v<VERSION>`, and comment on it at every phase.
2. Open the PR `release/v<VERSION>` (`Project.toml`, `CHANGELOG.md`, `BREAKING.md`) after the release gate passed locally; labels <run labels, GPU: kkt or occidata>, `documentation push preview`.
3. <autonomous: enable auto-merge: GitHub merges when the required checks pass (gate, breakage report, CI) | supervised: wait for you to merge it>.
4. Summarize the checks on the issue.
5. Wait for the `ReleaseTag` workflow: tag `v<VERSION>` and GitHub release, notes since `<PREV>`.
6. <stable: post the registration comment, once (<supervised: after your go>), and follow the General PR to its merge | beta: register in ct-registry with LocalRegistry>.
7. Close the issue with a final comment.

Every step starts from `release.jl status`: if anything interrupts the release, it is resumed from where the facts say it is.

### What you will do

- <supervised: merge the PR; give the go before the registration comment>
- <jump: comment `[merge approved]` on the General PR, or let the agent do it>
- <anything else: none>

**Questions.** Is `<VERSION>` right? Mode: `supervised` (the agent stops at each checkpoint) or `autonomous` (it goes on unless something fails; <available: the repository has auto-merge and the required checks | not available: <why>>)?
````

## Issue description

The organization's release template is not used: the agent writes this description.

````markdown
Release of <NAME> v<VERSION> (<stable|beta>), prepared by an agent following [ct-agents/recipes/release.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/release.md). Progress is in the comments below, one per phase.

## Checklist

- [ ] Version bumped in `Project.toml`
- [ ] `CHANGELOG.md` / `BREAKING.md` up to date
- [ ] CI, documentation and breakage checks are green
- [ ] <stable: Resolves from General alone (`REL4`)>
- [ ] Release PR merged
- [ ] GitHub release and tag created
- [ ] Registered (<General | ct-registry>)
````

The checklist is ticked in the final comment, not by editing the description: the
description is written once.

## Phase comments

One per phase, posted when the phase ends (or stops):

```markdown
**Phase <N> — <title>.**
Done: <what was done, in one or two sentences>.
Decisions: <each choice not dictated by the recipe, and why; or none>.
Check: `<command>` → <actual result>.
Next: <the next phase, and who acts: the agent, or the human (what is expected)>.
<!-- ct-release … -->
```

### Checks summary

Phase 4:

```markdown
**Phase 4 — Checks of #<PR>.**

| Check | Result |
| --- | --- |
| <name> | ✅ / ❌ / skipped |

Breakage: <link to the comment of the Breakage workflow, and its verdict>.
Documentation preview: <link, or not deployed>.
Next: <autonomous: merge | supervised: @<human>, the PR is ready for you to merge>.
<!-- ct-release … -->
```

## Registration comment

Phase 7, stable only, **posted once**. The mention is built by `printf`, so that it exists
nowhere before this command runs:

```bash
gh release view "v$VERSION" -R "$REPO" --json body --jq .body > whats-changed.md
{
  printf '%s%s register\n\nRelease notes:\n\n' '@' 'JuliaRegistrator'
  cat release-notes.md      # the CHANGELOG section of VERSION, links made absolute at the tag
  echo
  cat whats-changed.md      # the notes of the GitHub release, made by ReleaseTag
} > registration.md
gh issue comment "$ISSUE" -R "$REPO" --body-file registration.md
```

`release-notes.md` is the `## [<VERSION>]` section of `CHANGELOG.md` without its heading;
relative links (`BREAKING.md`) become `https://github.com/<REPO>/blob/v<VERSION>/BREAKING.md`.

## Final comment

```markdown
**Phase 8 — <NAME> v<VERSION> is released.**

- [x] Version bumped in `Project.toml` — #<PR>
- [x] `CHANGELOG.md` / `BREAKING.md` up to date — #<PR>
- [x] CI, documentation and breakage checks are green — <checks summary comment>
- [x] <stable: Resolves from General alone (`REL4`)>
- [x] Release PR merged — <MERGE_SHA>
- [x] GitHub release and tag created — <release URL>
- [x] Registered — <General PR, merged | ct-registry commit>

Left as is: <anything, with the reason; or nothing>.
Follow-ups: <downstream [compat] bumps (CompatHelper), issues to open; or none>.
<!-- ct-release phase=8 … state=done -->
```
