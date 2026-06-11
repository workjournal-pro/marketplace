---
name: prime
description: >
  Prime the session: list repository files, surface relevant journal entries,
  and show current branch status (open PR, merged, or in-progress).
allowed-tools: Bash(git:*), Bash(gh:*), Skill
metadata:
  author: Venture Squad LTD
  version: "0.1"
---

Prime the session by running the steps below. **Each Bash call must be a single command** — never combine commands with `&&` or `;`. Present all output to the user.

## Step 1: Repository Files

Run these as two separate Bash tool calls, then present the combined results under a `## Repository files` heading:

1. `git ls-files` — show under a `### Tracked` subheading.
2. `git status -s` — show under a `### Status` subheading.

## Step 2: Journal

Surface the last 3 entries from the project's journal. Pick which journal system based on availability:

1. Check whether this project uses Workjournal: run `ls .workjournal 2>/dev/null` as a single Bash call.
2. If stdout shows `.workjournal` AND the `workjournal:workjournal` skill is in the available-skills list, invoke `Skill(skill: "workjournal:workjournal", args: "last 3")`.
3. Otherwise, invoke `Skill(skill: "devjournal:devjournal", args: "last 3")`.
4. If the chosen skill fails, skip silently.

## Step 3: Branch Status

Determine branch status by running single-command Bash calls one at a time. Stop as soon as you have enough information.

1. Run `git branch --show-current` to get the current branch name.
2. Run `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'` to get the default branch. If it fails, assume `main`.
3. If the current branch **is** the default branch, print `## Branch: <name>` and `On default branch.` — done.
4. Otherwise, run `gh pr view --json state,title,url,isDraft` to check for a PR.
   - If a PR exists, print `## Branch: <name>` with the PR state (Draft/Open/Merged/Closed), title, and URL.
   - If no PR exists, run `git rev-list --left-right --count origin/<default>...HEAD` and print `## Branch: <name>` with `No PR. X commit(s) ahead, Y behind <default>.` If that also fails, print `No PR. Local branch (no remote tracking).`
