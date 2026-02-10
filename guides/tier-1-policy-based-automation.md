# Tier 1: Policy-Based Automation (No AI)

> Set-and-forget workflows that eliminate manual toil using deterministic rules, schedules, and branch protection policies. This is the foundation every repo should have before layering on AI.

## Why Start Here

Policy-based automation is battle-tested, predictable, and free (or nearly free). These tools have been running in production across millions of repositories for years. They don't hallucinate, they don't need API keys to LLM providers, and they don't surprise you. Start here, measure the time saved, and only move to Tier 2/3 when you hit the limits of what rules can express.

## The Core Stack

### 1. Dependency Updates

**Pick one of:**

| Tool | Best For | Setup |
|------|----------|-------|
| [Dependabot](https://github.com/dependabot) | GitHub-only repos, simplicity | Drop a `.github/dependabot.yml` in your repo |
| [Renovate](https://github.com/renovatebot/renovate) | Multi-platform, monorepos, maximum control | Install the GitHub App or self-host |

**Recommended Renovate config for a typical repo:**

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "automerge": true,
  "automergeType": "pr",
  "automergeStrategy": "squash",
  "packageRules": [
    {
      "matchUpdateTypes": ["patch", "pin", "digest"],
      "automerge": true,
      "automergeType": "branch"
    },
    {
      "matchUpdateTypes": ["minor"],
      "automerge": true,
      "groupName": "minor updates"
    },
    {
      "matchUpdateTypes": ["major"],
      "automerge": false,
      "labels": ["major-update", "needs-review"]
    },
    {
      "matchDepTypes": ["devDependencies"],
      "automerge": true,
      "groupName": "dev dependencies"
    }
  ],
  "schedule": ["after 9am and before 5pm every weekday"],
  "timezone": "America/New_York"
}
```

**Key decisions:**
- **Automerge patches**: Almost always safe. If your CI is solid, let these flow.
- **Group minor updates**: Reduces PR noise. One PR for all minor bumps.
- **Major updates need humans**: These can break APIs. Always review.
- **Schedule during work hours**: So failures get noticed, not buried over the weekend.

**Equivalent Dependabot config:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/New_York"
    groups:
      minor-and-patch:
        update-types: ["minor", "patch"]
    reviewers:
      - "your-team"
    labels:
      - "dependencies"
    open-pull-requests-limit: 10
```

### 2. Merge Automation

**The problem:** PRs sit waiting for someone to click "merge" even after CI passes and reviews are approved.

**Option A: GitHub native auto-merge**

Enable auto-merge on individual PRs via the GitHub UI or API. Requires branch protection with required status checks.

**Option B: Mergify (recommended for teams > 5)**

```yaml
# .mergify.yml
pull_request_rules:
  # Auto-merge dependency bot PRs that pass CI
  - name: Auto-merge dependency updates
    conditions:
      - author~=^(dependabot\[bot\]|renovate\[bot\])$
      - check-success=ci
      - "#approved-reviews-by>=1"
    actions:
      merge:
        method: squash

  # Auto-merge docs-only changes
  - name: Auto-merge documentation
    conditions:
      - files~=^(docs/|README|\.md$)
      - -files~=^(?!docs/|README|.*\.md$)
      - check-success=ci
    actions:
      merge:
        method: squash

  # Auto-label by file path
  - name: Label frontend changes
    conditions:
      - files~=^src/(components|pages|styles)/
    actions:
      label:
        add: ["frontend"]

  - name: Label backend changes
    conditions:
      - files~=^src/(api|services|models)/
    actions:
      label:
        add: ["backend"]

  # Request reviews based on area
  - name: Request backend review
    conditions:
      - files~=^src/(api|services|models)/
      - -author~=^(dependabot\[bot\]|renovate\[bot\])$
    actions:
      request_reviews:
        teams:
          - backend-team
```

**Option C: Kodiak (lightweight, open source)**

```toml
# .kodiak.toml
version = 1

[merge]
automerge_label = "automerge"
method = "squash"
delete_branch_on_merge = true

[merge.message]
title = "pull_request_title"
body = "pull_request_body"

[update]
always = true  # Keep PRs up to date with base branch
```

### 3. Release Automation

**Pick one of:**

| Tool | Philosophy | Best For |
|------|-----------|----------|
| [semantic-release](https://github.com/semantic-release/semantic-release) | Fully automated, commit-driven | Single-package repos with strict Conventional Commits |
| [Release Please](https://github.com/googleapis/release-please) | Human-approved, PR-based | Teams that want a review step before releasing |
| [Changesets](https://github.com/changesets/changesets) | Developer-declared intent | Monorepos, teams that don't want to enforce commit conventions |

**semantic-release GitHub Actions workflow:**

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**Release Please workflow:**

```yaml
# .github/workflows/release-please.yml
name: Release Please
on:
  push:
    branches: [main]
permissions:
  contents: write
  pull-requests: write
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
```

Release Please maintains a "Release PR" that accumulates changes. When you merge it, a release is cut. This gives you a human checkpoint without manual changelog writing.

### 4. Issue and PR Hygiene

```yaml
# .github/workflows/stale.yml
name: Stale Issues
on:
  schedule:
    - cron: "30 9 * * 1"  # Every Monday at 9:30 AM
jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v9
        with:
          stale-issue-message: >
            This issue has been inactive for 60 days.
            It will be closed in 14 days if there is no further activity.
          stale-pr-message: >
            This PR has been inactive for 30 days.
            It will be closed in 7 days if there is no further activity.
          days-before-issue-stale: 60
          days-before-pr-stale: 30
          days-before-issue-close: 14
          days-before-pr-close: 7
          exempt-issue-labels: "pinned,security,blocked"
          exempt-pr-labels: "pinned,work-in-progress"
```

```yaml
# .github/workflows/labeler.yml
name: Labeler
on:
  pull_request_target:
    types: [opened, synchronize]
permissions:
  contents: read
  pull-requests: write
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
```

### 5. Branch Protection

This isn't a tool you install — it's GitHub settings that underpin everything else. Without branch protection, auto-merge and merge queues are meaningless.

**Recommended settings for `main`:**

- Require pull request reviews (at least 1)
- Require status checks to pass (your CI job names)
- Require branches to be up to date before merging
- Require signed commits (optional but recommended)
- Do not allow bypassing the above settings
- Enable merge queue (if on GitHub Enterprise Cloud or public org repos)

### 6. Conventional Commits Enforcement

```yaml
# .github/workflows/commitlint.yml
name: Commit Lint
on: [pull_request]
jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v6
```

Pair with local enforcement via Husky + commitlint:

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional husky
npx husky init
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

```js
// commitlint.config.js
export default { extends: ['@commitlint/config-conventional'] };
```

## The Complete Tier 1 Workflow

```
Developer pushes → commitlint validates message
                 → CI runs tests, linting, security scans
                 → Labeler tags PR by file paths
                 → Mergify/Kodiak auto-merges if policy met
                 → semantic-release/Release Please cuts a release
                 → Stale bot cleans up abandoned PRs/issues

Renovate/Dependabot → Opens dependency PRs on schedule
                    → Automerge patches that pass CI
                    → Label major updates for human review
```

## What This Gets You

- **Zero manual dependency management** for patch/minor updates
- **Automatic releases** tied to commit conventions
- **Self-cleaning issue tracker** with stale bot
- **Consistent PR labeling and review routing**
- **Protected main branch** with enforced quality gates

## When to Move to Tier 2

You've outgrown Tier 1 when:
- You want feedback on code *quality* and *design*, not just "does it pass CI?"
- Your team spends significant time writing PR descriptions or summaries
- You want to catch bugs that tests don't cover (logic errors, edge cases)
- You need to enforce style guides that go beyond linting rules
- Review turnaround is a bottleneck despite auto-merge being in place

See [Tier 2: AI-Assisted Review and Triage](tier-2-ai-assisted-review.md).
