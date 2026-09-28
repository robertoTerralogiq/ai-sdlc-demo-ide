# Journey: LOAN-12 in IDE mode

This is a live run on 2026-09-28 (UTC) of one ticket, from issue to merged PR, with
`SDLC_MODE=ide`. In this mode the AI review runs from the developer's IDE rather than in CI.

Two identities appear on GitHub:
- **Developer**: [@robertoTerralogiq](https://github.com/robertoTerralogiq), a person. Files the
  ticket, makes the judgment calls, and owns the merge.
- **Developer assistant**: [@developmentAssistant](https://github.com/developmentAssistant), the
  automated actor. It writes the commits, opens the PR, reviews it and pushes the fixes. It has
  write access, but no admin rights and no `workflow` scope, so it cannot change the gate that
  checks its own work.

What made the merge wait (branch protection on `main`):
- the `stage-3: merge-gate` check;
- the `stage-2: ai-review (ide)` commit status, which the IDE review posts on the exact commit it
  reviewed;
- every conversation resolved.

## Stages

| # | Stage | Who | What happened | Evidence |
| --- | --- | --- | --- | --- |
| 0 | Baseline | Developer | Repo created, and protection and mode set | [`5983904`](../../commit/5983904), [run](../../actions/runs/36403069183) |
| 0.1 | Gate fix | Developer | Race found: the gate only waited for tests, so auto-merge could have merged before any review. Fixed by requiring the review status on each commit | [`7e5b675`](../../commit/7e5b675) |
| 1 | Ticket | Developer | LOAN-12 filed with six acceptance criteria | [#1](../../issues/1) |
| 2 | Implement | Assistant | First draft committed on `feat/LOAN-12-early-settlement` | [`a56012f`](../../pull/2/commits/a56012f) |
| 3 | Open PR | Assistant | PR opened with auto-merge (squash, delete branch) | [#2](../../pull/2) |
| 4 | AI review, round 1 | Assistant (IDE) | **6 findings: 3 blockers** (SQL injection, hardcoded key, key logged) and 3 majors (float money, no timeout, swallowed exception). Status set to `failure` ⇒ **merge blocked** | [summary](../../pull/2#issuecomment-5867140473), status on `a56012f` |
| 5 | Fix, round 1 | Assistant | Bound SQL, `LookupError`, Decimal half-up, key required at startup and never logged, timeout, tests for criteria 1–5 | [`58546eb`](../../pull/2/commits/58546eb), [run](../../actions/runs/36403512523) |
| 6 | AI review, round 2 | Assistant (IDE) | Resolved all **6** earlier threads. Found 3 new ones: 1 major (unreachable status check) and 2 minors (no `send_quote` test, sqlite connection not closed). Status `success`, but the open conversations still blocked the merge | status on `58546eb` |
| 5b | Fix, round 2 | Assistant | Developer chose to fix all three, not waive them. Dead check removed, `send_quote` payload/timeout/rejection tested, fixture closes the connection | [`a1ed444`](../../pull/2/commits/a1ed444), [run](../../actions/runs/36403810478) |
| 6b | AI review, round 3 | Assistant (IDE) | Resolved the **3** previous threads. 1 new minor: core-banking URL is hardcoded. Status `success` | status on `a1ed444` |
| 7 | Decision | Developer | Accepted the minor as out of scope, filed follow-up [#3](../../issues/3), replied on the thread and resolved it | [#3](../../issues/3) |
| 8 | Merge | GitHub auto-merge | All requirements met ⇒ squash-merged, branch deleted, #1 closed. GitHub credits the merge to the assistant because it enabled auto-merge | [`b8f68b0`](../../commit/b8f68b0), [run on main](../../actions/runs/36404061284) |

Totals:
- 3 AI review rounds and 2 fix rounds.
- 10 review threads, all resolved: 9 by the assistant's re-reviews and 1 by the developer.
- Tests went from 5 to 12.
- About 9 minutes from the PR opening to the merge.

## What was live and what was a stand-in

- **Live:** every GitHub action and identity above, every CI run, and every Gemini review
  (`gemini-2.5-pro`, run with the reviewer from `antigravity_reviewer/`, as the `review-pr`
  skill does).
- **Stand-ins:** Antigravity itself was not installed on the machine that ran this. The code
  the `implement-ticket` and `address-review` skills would write came from committed files:
  - stage 2 used `demo/mr-fixture/`, written in advance;
  - stage 5 used `demo/fix-round-1/`, written in advance;
  - stage 5b used `demo/fix-round-2/`, written live by Claude Code acting as the assistant.
- **Reviewer resolving its own threads:** a re-review resolves a thread when the problem is no
  longer reported. Its reply says "not reported", not "fixed", because it cannot tell a fix
  from a dropped false positive.
- **Round 3 also reviewed `demo/fix-round-2/`,** because that stand-in was committed in the PR
  and the local run lacked CI's `demo/**` exclude. The `review-pr` skill command now sets
  `REVIEW_EXCLUDE_GLOBS="demo/**"`.

## Re-run it

The steps are in `DEMO_RUNBOOK.md`, Act 3.
