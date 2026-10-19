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
**Versions applied.** ct-agents `<TAG>` (this recipe), ct-conventions `<TAG>`, CTActions `<commit>` (the release tool).

### What changes since `<PREV>`

- <one line per change, from the merged PRs and `[Unreleased]`, grouped as in `CHG2`>
- Breaking: <none | each one, with its `BREAKING.md` entry>
- Not in `[Unreleased]` yet, added by the release PR: <PRs, or none>
- Open PRs, not in this release: <#N title, or none>

### What the agent will do

1. Open the tracking issue `[RELEASE] v<VERSION>`, and comment on it at every phase.
2. Open the PR `release/v<VERSION>` (`Project.toml`, `CHANGELOG.md`, `BREAKING.md`) after the release gate passed locally; labels <run labels, GPU: kkt or occidata>, `documentation push preview`.
3. <autonomous: merge with `release.jl merge` once `release.jl ready` says ready (what actually ran on the head commit: gate, breakage report, every job the release needs) | supervised: wait for you to merge it, once `release.jl ready` says ready>.
4. Summarize the checks on the issue.
5. Wait for the `ReleaseTag` workflow: tag `v<VERSION>` and GitHub release, notes since `<PREV>`.
6. <stable: post the registration comment, once (<supervised: after your go>), and follow the General PR to its merge | beta: register in ct-registry with LocalRegistry>.
7. Close the issue with a final comment.

Every step starts from `release.jl status`: if anything interrupts the release, it is resumed from where the facts say it is.

### What you will do

- <supervised: merge the PR; give the go before the registration comment>
- <jump: comment `[merge approved]` on the General PR, or let the agent do it>
- <anything else: none>

**Questions.** Is `<VERSION>` right? Mode: `supervised` (the agent stops at each checkpoint) or `autonomous` (it goes on unless something fails; <available: the package has `ReleaseGate.yml`, and the agent merges only with `release.jl merge` | not available: <why>>)?
````

## Issue description

The organization's release template is not used: the agent writes this description.

````markdown
Release of <NAME> v<VERSION> (<stable|beta>), prepared by an agent following [ct-agents/recipes/release.md](https://github.com/control-toolbox/ct-agents/blob/main/recipes/release.md). Progress is in the comments below, one per phase; each one repeats the steps, marked as far as they went.

## Steps

<the progress list, no step marked>
````

The description is written once, never edited.

## The progress list

The steps of the release, one line each. Every phase comment repeats the whole list: ✅ on
a step done, ❌ on the step that stopped the release (with the reason, the release does
not go on), nothing on a step not reached. Reading the comments in order shows how the
release went. No check boxes: nobody ticks them.

```markdown
- Proposal validated: <stable|beta> `<VERSION>`, <supervised|autonomous>
- <stable: Resolves from General alone (`REL4`)>
- Release PR opened, the gate passed locally — `Project.toml`, `CHANGELOG.md`, `BREAKING.md`
- Checks green (`release.jl ready`): CI, documentation, breakage, the gate
- Release PR merged
- Tag and GitHub release; documentation of the tag
- Registered in <General | ct-registry>
- Issue closed
```

E.g. after phase 4 of a stable, its checks failing:

```markdown
- ✅ Proposal validated: stable `0.30.12`, autonomous
- ✅ Resolves from General alone (`REL4`)
- ✅ Release PR opened, the gate passed locally — #625
- ❌ Checks green (`release.jl ready`): `CI.yml`, run 392…: `test (1.12)` did not pass
- Release PR merged
- Tag and GitHub release; documentation of the tag
- Registered in General
- Issue closed
```

## Phase comments

One per phase, posted when the phase ends (or stops):

```markdown
**Phase <N> — <title>.**
Done: <what was done, in one or two sentences>.
Decisions: <each choice not dictated by the recipe, and why; or none>.
Check: `<command>` → <actual result>.
Next: <the next phase, and who acts: the agent, or the human (what is expected)>.

<the progress list, marked>
<!-- ct-release … -->
```

### Checks summary

Phase 4:

```markdown
**Phase 4 — Checks of #<PR>.**

<the output of `release.jl ready`, as is: one line per workflow, the run that counts and its verdict>

Breakage: <link to the comment of the Breakage workflow, and its release check>.
Documentation preview: <link, or not deployed>.
Next: <autonomous: `release.jl merge` | supervised: @<human>, `release.jl ready` says ready: the PR is yours to merge>.

<the progress list, marked>
<!-- ct-release … -->
```

## Registration comment

Phase 7, stable only, **posted once**, by the release tool — never by hand:

```bash
$TOOL register --repo "$REPO" --version "$VERSION"          # dry run, the mention masked
$TOOL register --repo "$REPO" --version "$VERSION" --post   # refused unless status says phase 7, todo
```

It is the bot's mention and command, `Release notes:`, the `## [<VERSION>]` entry of
`CHANGELOG.md` (relative links made absolute at the tag), then the notes of the GitHub
release. The mention is built by the tool, so it exists in no file and in no command.

## Final comment

```markdown
**Phase 8 — <NAME> v<VERSION> is released.**

- ✅ Proposal validated: <stable|beta> `<VERSION>`, <supervised|autonomous>
- ✅ <stable: Resolves from General alone (`REL4`)>
- ✅ Release PR opened, the gate passed locally — #<PR>
- ✅ Checks green (`release.jl ready`) — <checks summary comment>
- ✅ Release PR merged — <MERGE_SHA>
- ✅ Tag and GitHub release; documentation of the tag — <release URL>
- ✅ Registered in <General | ct-registry> — <General PR, merged | ct-registry commit>
- ✅ Issue closed

Left as is: <anything, with the reason; or nothing>.
Follow-ups: <downstream [compat] bumps (CompatHelper), issues to open; or none>.
<!-- ct-release phase=8 … state=done -->
```
