# Claude Code adapter

What is specific to Claude Code when applying [`base.md`](../base.md) and the
procedures of
[`method.md`](https://github.com/control-toolbox/ct-conventions/blob/main/method.md).

## Editing files

Prefer the dedicated tools (Read/Edit/Write) over `cat`/`sed`/`awk`/`echo` in the shell
for reading or modifying files.

## Running a slow suite in the background with `Monitor`

For a slow suite (collocation/Ipopt-heavy, full regression run), the `tee`'d command
does not have to block the session:

1. Launch it via `Bash` with `run_in_background: true`.
2. Arm `Monitor` on the log file, filtering for `"Test Failed"|"Error During Test"`
   (Julia `Test.jl`'s exact failure/error markers) — each match streams as a
   notification while the suite keeps running.
3. Keep working (read code, draft the next change); the background task itself
   reports completion separately, then run `generate_report` on the full log as usual.

Not a default — for a short suite (well under a minute), just running and waiting is
simpler and the extra setup isn't worth it. Reach for it when a run is expected to
take minutes, so a failure doesn't sit unnoticed and idle time isn't wasted.

## Interactive flags

Interactive git flags (`-i`, e.g. `git rebase -i`) are not supported.
