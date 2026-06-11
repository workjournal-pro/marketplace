---
name: wj-gh
description: End-to-end GitHub change workflow — scope → optional issue → branch → commit (with tests) → open PR → watch CI → respond to a code-review bot → fix mechanical CI failures → reply to review comments inline → merge. Optional issue-first and journal-entry steps. Works with any GitHub repo; treats CodeRabbit (or any review bot) as an optional integration.
metadata:
  author: Venture Squad LTD
  version: "0.1"
---

You are handling a `/wj-gh` invocation. The user is preparing to open (or has just opened) a pull request and wants the workflow driven from change to merge — and once a PR is open, wants this skill (and the agent it's loaded in) to watch CI and any code-review bot and respond to whatever lands without further prompting. The skill never opens or merges a PR on its own; acting on a check (bumping a version, fixing a review comment, retrying a failed CI job) is the agent's job, and merging is the user's call.

This is a **generic** GitHub workflow. It makes no assumptions about your stack, your CI provider, or your branch-protection rules beyond what `git` and the `gh` CLI expose. Where a step depends on a repo convention (base branch, version gating, a review bot), it says so — adapt it to your repo.

## Prerequisites

- `gh` authenticated (`gh auth status`). If not, stop and tell the user to run `gh auth login`.
- A clean idea of the **base branch** for this PR. Many repos use `main`; others use `develop`/`staging` as an integration branch and reserve `main` for releases. Detect the default with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name` and confirm with the user if the repo uses a non-default integration branch.

## Dispatch

The user invoked: `/wj-gh {args}`

| First word | Behaviour |
|---|---|
| *(empty)* | **Auto-detect** — run the **Common to every PR** checklist against `git diff --name-only <base>...HEAD` (or staged + unstaged if pre-commit). |
| `feature` | Run the **Feature workflow** end-to-end (scope → optional issue → branch → code → PR → merge → optional journal entry). |
| `monitor` | Run the **Post-push monitoring** loop on the current branch's PR — once. |
| `review` | Run the **Responding to a code-review bot** method on the current branch's PR. |
| `help` | Print this dispatch table and stop. |

Auto-detect is the default. Most PRs benefit from the common checklist before they go out.

## Feature workflow

End-to-end loop for shipping a non-trivial change. Use this when starting from "I want to do X" rather than "I just edited a file". Each step is a checklist item — read it, run the listed checks, surface anything outstanding, fix, then move on. The workflow isn't strictly linear; a discovery while coding often loops you back to scope or docs. The checklist exists so nothing gets silently skipped, not to lock the order.

### 1. Define scope

- [ ] **Outcome.** One sentence: what's the user-visible change? (Not "refactor X" — "users can now Y".)
- [ ] **Acceptance criteria.** 3–5 bullets the work has to satisfy. These become the PR's test plan.
- [ ] **Out of scope.** List explicitly. Reviewers expand PRs by default; pre-empt that.

### 2. Check prior work (quietly — only surface if relevant)

- [ ] **Existing issues / PRs.** `gh issue list --search "<topic>"` and `gh pr list --search "<topic>" --state all`. There may already be one to claim or build on.
- [ ] **Existing docs / code.** `grep -r "<concept>"` to find where the thing already lives. Read those before touching them.
- [ ] If the repo keeps a decision journal or changelog, search it for prior decisions on this concept.

### 3. Issue — create or claim (optional but recommended)

Many teams require an issue per change; some don't. If yours does:

- [ ] **Search first.** If an issue exists, comment "picking this up" with a one-paragraph scope summary and reuse its number.
- [ ] **If creating, use a clear body shape:** Context (why now) · Acceptance criteria (from step 1) · Out of scope · References (docs/files the change relates to).
- [ ] Capture the issue number — commits, the PR title, and the PR body should reference it (`Closes #N`).

### 4. Initialise the branch

- [ ] `git fetch origin <base>` then `git checkout -b <type>/<slug> origin/<base>` — branch off the **freshest** base, not a stale local branch.
- [ ] Branch naming: `<type>/<slug>` where type is one of `feat` / `fix` / `docs` / `chore` (or your repo's convention). Include the issue number if you have one: `feat/123-add-export`.
- [ ] Install deps if the lockfile changed since your last branch.

### 5. Update docs first (if the change has a spec surface)

Updating docs **before** writing code makes them serve as your spec — discrepancies surface early.

- [ ] Update the relevant design/architecture doc to describe how the system *will* behave.
- [ ] **Semantic-rename sweep** — if a concept is being renamed, every reference (docs, code, error messages, tests) updates in the same change. `grep -r "<old-name>"` should return nothing after.

### 6. Write code (with tests, not after)

- [ ] **Read 2–3 nearby files first** to match patterns. Don't invent new conventions when one already exists.
- [ ] **Tests come with the code, not after.** Use the closest existing test file as a template.
- [ ] **Each logically separate surface lands in its own commit** if it's reviewable separately.
- [ ] **No security threats introduced.** Validate input at every system boundary; never trust client-supplied data server-side. No committed secrets.

### 7. Pre-push validation

- [ ] Run the repo's linter/formatter.
- [ ] Run the tests for each touched package/module. CI will run them again — local is faster for iteration.
- [ ] **Version bump** if your repo gates merges on one (see notes on versioning below).
- [ ] `git diff <base>...HEAD` — re-read your own diff. Catch the silly things (debug logs, commented-out code, accidentally-staged files) before reviewers do.

### 8. Open the PR

- [ ] Run `/wj-gh` (auto-detect) to surface the common checklist. Resolve outstanding items.
- [ ] `gh pr create --title "<type>(<scope>): <one-line>" --body-file <path> --base <base> --head <branch>`.
- [ ] If you need to retarget an existing PR's base and `gh pr edit --base` errors, fall back to `gh api -X PATCH repos/<owner>/<repo>/pulls/<n> -f base=<base>`.
- [ ] PR body includes: `Closes #N` (if you have an issue), the actual verification steps you ran (not just "ran tests"), and links to any docs touched.
- [ ] **Switch to `/wj-gh monitor`** — the agent now owns the PR until it merges or the user stops it.

### 9. Iterate on CI + review

Use `/wj-gh monitor` and `/wj-gh review`. Mechanical CI failures (a forgotten version bump, a generated-file/mirror check, a rebase after the parent of a stacked PR squash-merged) are yours to fix and push without asking. Review-bot comments are fix-or-reply (see the `review` method below).

### 10. After merge — record the *why* (optional)

If your team keeps a decision journal, changelog, or ADR trail, add an entry. The commit log captures *what changed*; the journal captures *why and what you tried* — the context future readers can't reconstruct from a diff.

- Concise outcome title; a paragraph summary; a body with file paths, decisions, **rationale**, trade-offs accepted, and dead-ends discovered. **Not** a re-list of `git diff --name-only`.

### Anti-patterns to avoid

- ❌ Going straight to code without updating docs first → forces reviewers to reverse-engineer intent from the diff.
- ❌ Letting one PR sprawl across multiple unrelated scopes → makes review harder and version bumps ambiguous. Split if the changes can ship independently.
- ❌ Skipping the rationale record after merge → future you re-derives decisions you already made.

## Common to every PR (always check)

These apply regardless of scope. Run this first.

- [ ] Branch cut from the latest base (not a stale local branch). `git log <base>..HEAD` — every commit in that range should be intentional.
- [ ] Linter, build, and tests pass locally.
- [ ] If your repo requires issues, one exists and the PR body references it.
- [ ] No security threats introduced: no committed secrets, no injection, no client-side trust of server input. Spot-check the diff if it touches auth or user-input parsing.
- [ ] Semantic consistency: if a concept was renamed, every reference was renamed too.
- [ ] Version bump if your repo gates on one.
- [ ] Any actionable review-bot comment from a prior round was either fixed in code or **replied to inline** (top-level PR comments don't feed a review bot's memory — see below).

## Post-push monitoring

After every `git push` on a PR branch, **the agent owns the PR until it merges**. Don't wait to be prompted; watch CI and any review bot, react to whatever lands, and only stop when one of these happens:

1. The PR is approved and the user merges it.
2. CI is green AND the review is resolved (approved, or its only outstanding review is a comment on a resolved conversation).
3. The user explicitly says "stop watching" / "I'll handle it from here".

### What to watch

```sh
gh pr view <pr> --json statusCheckRollup,reviewDecision,reviews
gh api repos/<owner>/<repo>/pulls/<pr>/comments    # inline review comments + replies
gh api repos/<owner>/<repo>/issues/<pr>/comments   # top-level comments
```

The PR number for the current branch is `gh pr view --json number --jq .number`.

### Status interpretation

| Signal | What to do |
|---|---|
| Any CI check `IN_PROGRESS` | Wait. Re-poll in ~60–90s. Don't push more commits unless they're independent fixes. |
| CI check `FAILURE` | Yours to fix. `gh run view <id> --log-failed` to grep the error. Diagnose, fix, push. |
| CI green, no review yet | Re-poll once. A review bot usually posts within a few minutes; if nothing after ~5 min, move on. |
| Review `CHANGES_REQUESTED` with new inline comments | Run the **Responding to a code-review bot** method. Don't merge until every actionable item is fixed-or-replied. |
| Review acknowledges your fix but `reviewDecision` still reads `CHANGES_REQUESTED` | Normal — GitHub doesn't auto-dismiss a bot's earlier review. If every conversation is resolved, the user can merge. |

### Pacing

Don't sleep-poll in tight loops — that wastes cycles and adds noise. Use a 60–90s delay when actively watching, longer when CI is slow. If a job is genuinely stuck (≫ its normal runtime), surface the `detailsUrl` to the user and stop — it's likely a runner issue, not yours to debug.

## Responding to a code-review bot

Many repos run an automated reviewer (e.g. **CodeRabbit**). If yours configures it to *request changes* (CodeRabbit's `request_changes_workflow: true`), the bot's review lands as **Request Changes** and only flips to **Approve** once every comment is resolved — applied in code, or replied to and marked resolved by the bot. If your repo has no review bot, skip this section.

### The rule

Every actionable comment must end up either **fixed** or **replied to**. Silent dismissal leaves the review in Request Changes and teaches the bot nothing.

- Post the reply on the **inline review comment**, not as a top-level PR comment — only inline replies feed the bot's learning.
- Use `gh api -X POST repos/<owner>/<repo>/pulls/<pr>/comments/<comment_id>/replies -f body='…'`. A top-level `gh pr comment` does NOT count.
- Address the bot (e.g. `@coderabbitai`) in the reply when you want it to acknowledge / resolve.
- Be terse and load-bearing: state the convention, the constraint, and a pointer to a canonical reference (file path or prior PR) so the learning is self-contained.
- This applies **even when the bot is wrong** — replying with the correction is how it learns it was wrong.

### Operational steps

For each review round:

1. **Enumerate the actionable comments.**

   ```sh
   gh api repos/<owner>/<repo>/pulls/<pr>/comments \
     --jq '.[] | select(.in_reply_to_id == null) | {id, path, line, body: .body[:200]}'
   ```

   The latest review body usually lists how many actionable comments it posted; each is an inline comment with `in_reply_to_id == null`.

2. **For each comment, decide: fix or decline.** Fix if it's right (edit, test, include in the next commit). Decline if it's wrong — state the actual convention with a path reference.

3. **Reply inline on every comment** — fixed or declined.

   ```sh
   gh api -X POST repos/<owner>/<repo>/pulls/<pr>/comments/<comment_id>/replies \
     -f body='Applied in <commit-sha>. <one-line of why / pointer to convention>.'
   ```

4. **After the next push, re-check.** The bot re-walks the diff and either posts new comments, acknowledges your fixes, or submits a fresh `APPROVED` review.

### Anti-patterns

- ❌ Posting a top-level reply — the bot ignores those for learning.
- ❌ Silently fixing without replying — leaves the review in Request Changes for human reviewers.
- ❌ Long, defensive replies — be terse; state the convention, point at the file, move on.

## Notes

- **Versioning.** If your repo gates merges on a version bump, bump per semver: patch for bug fixes, minor for new backwards-compatible features, major for breaking changes. Bump every artifact the gate checks (package manifests, plugin/extension manifests, etc.).
- This skill is provider-agnostic. The `gh` commands assume GitHub; adapt the equivalents if you're on another forge.
- Report issues with this skill at <https://github.com/workjournal-pro/feedback/issues>.
