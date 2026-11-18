# pr-review

A Claude Code plugin that runs headless code reviews with `claude -p`. It reviews one PR, your local branch before a push, or every open PR waiting on you or your teams. Each review is saved as structured markdown with a resumable Claude session. Nothing is ever posted to GitHub.

It has a `--local` mode, a config file, and swappable panels and lenses. It reads a repo's review rules from the base branch, and it ships with no repo or team scope of its own.

## Install

```bash
claude plugin marketplace add aasimsani/pr-review
claude plugin install pr-review@aasimsani-pr-review
```

## Use

```bash
pr-review.sh --repo owner/name --pr 123   # one PR
pr-review.sh --local                      # current branch vs origin/<default>
pr-review.sh                              # everything awaiting your review
```

Inside Claude Code, ask for a review and the `pr-review` skill runs the script. See [`SKILL.md`](plugins/pr-review/skills/pr-review/SKILL.md) for configuration, scheduling and resuming a review, and [`examples/team-setup.md`](plugins/pr-review/examples/team-setup.md) for a full team setup with a sample review.

## Extending: perspectives

Each review has one **panel** and any number of **lenses**, and both are plain markdown files.

- **The panel** says who reviews and what they focus on. The built-in panel is three senior engineers, then a tech lead who checks their findings against the wider codebase.
- **A lens** adds rules or a reviewer on top of the panel. Examples include a DATA category, an accessibility checklist, or "check every migration is reversible".

The severity scale, references, fingerprints and output format never change, so reviews stay comparable whichever perspective ran.

### Add a perspective

Write a lens as instructions to the reviewers. Use it for one run with `--lens`, or keep it in one of the places below.

```bash
pr-review.sh --local --lens ~/lenses/migrations.md --lens ~/lenses/a11y.md
```

| Keep it in | It applies to |
|---|---|
| `.pr-review/lenses/<name>.md` in the repo | Every review of that repo, versioned with the code |
| `REVIEW.md` at the repo root | Every review of that repo. Use it for the team's general rules. |
| `EXTRA_PROMPT_FILE` in `~/.config/pr-review/config.sh` | Every repo you review |

A lens that adds a category should say so and give its ID prefix and fingerprint prefix. Two ready-made lenses show the pattern:

| Lens | Adds |
|---|---|
| [`examples/data-integration-lens.md`](plugins/pr-review/examples/data-integration-lens.md) | An integration-platform reviewer and a DATA category |
| [`examples/ponytail-lens.md`](plugins/pr-review/examples/ponytail-lens.md) | An over-engineering reviewer and a SIMPLE category, based on [ponytail](https://github.com/DietrichGebert/ponytail) (MIT). It tags each cut `delete`, `reuse`, `stdlib`, `native`, `yagni` or `shrink`, and reports the net lines and dependencies it could remove. |

The ponytail lens works best while the code is still being written, so pass it to `--local` runs. Its findings are never High or Critical, so they don't block a merge on their own.

### Swap the panel

A panel file replaces the built-in panel completely. Name the reviewers, what each one looks for, and who checks their output.

| Set it with | It applies to |
|---|---|
| `--panel FILE` | This run only, overriding everything else |
| `.pr-review/panel.md` in the repo | Every review of that repo |
| `PANEL_FILE` in the config | Every repo that has no panel of its own |

```bash
pr-review.sh --repo acme/acme-api --pr 1234 \
  --panel plugins/pr-review/examples/panel-security-red-team.md
```

[`examples/panel-security-red-team.md`](plugins/pr-review/examples/panel-security-red-team.md) swaps in two attackers and a defender.

### Use different perspectives for different work

- **Per repo.** Commit `.pr-review/panel.md` and `.pr-review/lenses/` so everyone reviewing that repo gets the same perspective.
- **Per change.** Pass `--panel` or `--lens` for the change at hand. For example, use the red-team panel on an auth change, or a migrations lens on a schema change.
- **Per phase of work.** Use the ponytail lens on `--local` runs while you build, when cutting code is cheap. Leave it off the final PR review, so the human reviewer sees correctness findings without simplification nits.
- **Two passes.** Run the default panel, then run again with a specialist panel. The reports are separate files with comparable fingerprints.

The script reads repo files from the **base** branch (`origin/<base>`), never from the PR head. This means a PR that edits `REVIEW.md` or `.pr-review/` is still reviewed under the old rules, and the new rules apply after it merges.

## Test

```bash
bash tests/test_pr_review.sh
```

The test stubs `claude` and `gh`, so it costs nothing and never calls GitHub.

## License

MIT. See `LICENSE`.
