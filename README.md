> A scratch repo for real runs of Exhibit's release workflow (ticket 193), used for nothing else.

# Exhibit

Every pull request brings its own evidence. Exhibit decides what evidence a PR needs, checks that the evidence is strong, and learns from each bug that gets through.

**Status:** in development. A person makes a release by pushing a version tag such as `v0.1.0`, and the release workflow does the rest (docs/spec.md section 10, "Releases"). The project setup exists: `npm ci`, then `npm test` and `npm run lint`. CI runs both, and a secret scan, on every pull request and on every push to `main`. A red run on `main` means two merges clashed: each passed on its own, but not together. The fix is a PR that makes `main` green again.

What works today:

- `exhibit history --repo owner/name` exports the merged PRs, with their files, reviews and CI results, to `.exhibit/history.jsonl`. Inside a clone of the repo it also runs SZZ, which traces each fix back to the PR that added the bug (docs/spec.md section 9.2). It writes a report to `.exhibit/report.md` and `.exhibit/report.html` (section 8.5). A second run reads only what is new, and `--full` reads every PR again. It needs `GITHUB_TOKEN`: section 8.3 lists the permissions the token needs.
- `exhibit label` lets a person label each escaped defect, with optional Claude drafts.
- `exhibit check` checks one PR against the policy, and `exhibit publish` stores the result and posts a comment on the PR. The GitHub Action runs both.
- Exhibit checks its own pull requests in shadow mode, with its own `evidence.yml`.

- [Explainer with diagrams](docs/index.html): start here.
- [Spec](docs/spec.md): commands, file formats, how each part works, and the build order.

## The idea

When agents open more pull requests than a person can read, the question changes. It is no longer "is this code good?" It becomes "is this evidence strong enough that nobody needs to read the code?"

Exhibit answers that question in four steps:

1. **Learn from history.** It traces each fix back to the PR that added the bug, and turns what got through into rules in `evidence.yml`.
2. **Check each PR.** It runs the tests on the base and the head, looks for a test that failed before and passes now, measures how many planted bugs the tests catch on the changed lines, and runs any probes the policy names.
3. **Give a verdict.** The verdict is "meets policy" or "needs a person", and it always lists the gaps.
4. **Measure itself.** A backtest shows what the policy would have caught on past PRs. For each kind of change, Exhibit reports misses and false demands, so a person can decide when to stop reading the code for that kind.

## Add Exhibit to a repo

Exhibit starts in shadow mode: it shows a verdict on each PR, and nothing waits on it.

1. Add `evidence.yml` at the top of the repo. It says what evidence each kind of change needs (docs/spec.md section 8.1). Exhibit's own [`evidence.yml`](evidence.yml) is a small example.
2. Set `settings.test_command` so that it installs what the tests need and writes JUnit XML to `settings.test_results`. Exhibit runs it in clean worktrees of the base and the head.
3. Copy [`examples/pull-request.yml`](examples/pull-request.yml) to `.github/workflows/exhibit.yml`. Keep the same Exhibit commit SHA in both jobs. If your tests need a certain Node, add `actions/setup-node` before the check step.
4. Open a PR. The check job writes the verdict to the job summary. The publish job stores the exhibit on the `exhibit-data` branch and posts one comment on the PR.
5. When shadow mode has run long enough to trust it, a person sets `enforce: true` in both jobs.

The two field-test repos have setup notes of their own, with a workflow that pins a release and a first policy: [Building58](examples/building58/README.md) and [edd-platform](examples/edd-platform/README.md).

## Commands

These commands work today:

```
exhibit history     # read past PRs and find the bugs that got through
exhibit label       # error analysis, with optional Claude drafts
exhibit check       # check one PR against the policy
exhibit publish     # store the exhibit and post a comment on the PR
```

These are still planned:

```
exhibit backtest    # test the policy on past PRs
exhibit trace       # after a merge, find policy misses
```

Exhibit is a CLI and a GitHub Action. It needs no server.

## How it is built

An office room of Claude Code sessions in [Building58](https://github.com/bfalkowski/Building58) builds Exhibit. Two other rooms use it on their own repos and file issues here. See section 12 of the spec.

## License

MIT. Anyone may use, change and share Exhibit. See [LICENSE](LICENSE).
