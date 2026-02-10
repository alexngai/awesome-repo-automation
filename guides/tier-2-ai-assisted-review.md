# Tier 2: AI-Assisted Review and Triage

> AI agents that comment, suggest, and classify — but a human still makes the final call. This tier adds intelligence to your PR workflow without giving up control.

## Prerequisites

You should have [Tier 1](tier-1-policy-based-automation.md) in place first. AI review is most valuable when it builds on top of solid CI, branch protection, and automated merging. Without those, AI suggestions just become more noise in an already messy workflow.

## The Core Stack

### 1. AI Code Review

This is the highest-impact addition at this tier. Pick one:

| Tool | Strengths | Pricing | Self-Hosted |
|------|-----------|---------|-------------|
| [CodeRabbit](https://www.coderabbit.ai/) | Best overall accuracy, codebase graph, agentic pre-merge checks | Free tier, $24/seat/mo Pro | No |
| [Qodo Merge / PR-Agent](https://github.com/Codium-ai/pr-agent) | Open-source, self-hostable, slash commands | Free (self-hosted), paid hosted | Yes |
| [Greptile](https://www.greptile.com/) | Deep codebase context, cross-file analysis | $20/user/mo | Yes |
| [Ellipsis](https://www.ellipsis.dev/) | Auto-applies fixes, style guide enforcement | Commercial | No |
| [Sourcery](https://www.sourcery.ai/) | Best for Python-heavy teams | Free for OSS | No |
| [Bito AI](https://bito.ai/) | Combines LLM + static analysis (fbinfer, Snyk) | Free tier, $15/seat/mo | Yes (Enterprise) |

**Recommended: CodeRabbit for commercial teams, PR-Agent for self-hosted/OSS**

**PR-Agent setup (self-hosted via GitHub Actions):**

```yaml
# .github/workflows/pr-agent.yml
name: PR Agent
on:
  pull_request:
    types: [opened, reopened, ready_for_review]
  issue_comment:
    types: [created]

permissions:
  issues: write
  pull-requests: write
  contents: read

jobs:
  pr-agent:
    if: ${{ github.event.sender.type != 'Bot' }}
    runs-on: ubuntu-latest
    steps:
      - uses: Codium-ai/pr-agent@main
        env:
          OPENAI_KEY: ${{ secrets.OPENAI_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          command: auto_review
```

This gives you:
- Automatic review on every PR with categorized feedback
- Slash commands in PR comments (`/review`, `/describe`, `/improve`, `/ask`)
- Self-hosted so your code never leaves your infrastructure

**CodeRabbit setup (hosted):**

Install the [CodeRabbit GitHub App](https://github.com/apps/coderabbitai), then configure via `.coderabbit.yaml`:

```yaml
# .coderabbit.yaml
language: en
reviews:
  profile: chill  # Options: chill, assertive, followup
  request_changes_workflow: false
  high_level_summary: true
  poem: false
  review_status: true
  auto_review:
    enabled: true
    drafts: false
chat:
  auto_reply: true
```

The `chill` profile reduces noise — it only flags things that matter. The `assertive` profile catches more but is noisier. Start with `chill` and escalate.

### 2. Programmable PR Rules with Danger.js

AI review catches *qualitative* issues. Danger.js catches *structural* ones with deterministic rules. They complement each other perfectly.

```javascript
// dangerfile.ts
import { danger, warn, fail, message } from 'danger';

// Warn if PR is too large
const bigPRThreshold = 500;
const linesChanged = danger.github.pr.additions + danger.github.pr.deletions;
if (linesChanged > bigPRThreshold) {
  warn(`This PR changes ${linesChanged} lines. Consider breaking it up.`);
}

// Require changelog entry for non-trivial changes
const hasChangelog = danger.git.modified_files.includes('CHANGELOG.md')
  || danger.git.created_files.includes('CHANGELOG.md');
const isTrivial = (danger.github.pr.title + danger.github.pr.body).includes('#trivial');
if (!hasChangelog && !isTrivial && linesChanged > 20) {
  warn('Please add a CHANGELOG entry for this change.');
}

// Fail if tests are missing for new source files
const newSourceFiles = danger.git.created_files.filter(
  f => f.startsWith('src/') && !f.includes('test') && !f.includes('spec')
);
const newTestFiles = danger.git.created_files.filter(
  f => f.includes('test') || f.includes('spec')
);
if (newSourceFiles.length > 0 && newTestFiles.length === 0) {
  fail('New source files were added without corresponding tests.');
}

// Warn if package.json changed but lockfile didn't
const packageChanged = danger.git.modified_files.includes('package.json');
const lockfileChanged = danger.git.modified_files.includes('package-lock.json')
  || danger.git.modified_files.includes('yarn.lock')
  || danger.git.modified_files.includes('pnpm-lock.yaml');
if (packageChanged && !lockfileChanged) {
  warn('package.json changed but no lockfile was updated.');
}

// Celebrate first-time contributors
const isFirstPR = await danger.github.api.pulls.list({
  owner: danger.github.thisPR.owner,
  repo: danger.github.thisPR.repo,
  state: 'closed',
  creator: danger.github.pr.user.login,
}).then(r => r.data.length === 0);
if (isFirstPR) {
  message('Welcome! Thanks for your first contribution to this repo.');
}
```

```yaml
# .github/workflows/danger.yml
name: Danger
on: [pull_request]
permissions:
  pull-requests: write
  contents: read
jobs:
  danger:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx danger ci
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3. PR Classification and Routing with GitStream

GitStream classifies PRs by type (feature, fix, docs, refactor, dependency) and applies different policies:

```yaml
# .cm/gitstream.cm
automations:
  # Estimated review time label
  estimated_review_time:
    if:
      - true
    run:
      - action: add-label@v1
        args:
          label: "{{ calc.etr }} min review"

  # Auto-approve safe changes
  auto_approve_docs:
    if:
      - {{ files | allDocs }}
    run:
      - action: approve@v1
      - action: add-label@v1
        args:
          label: "docs-only"

  auto_approve_tests:
    if:
      - {{ files | allTests }}
    run:
      - action: approve@v1
      - action: add-label@v1
        args:
          label: "tests-only"

  # Flag risky changes
  flag_sensitive_files:
    if:
      - {{ files | match(list=['.*Dockerfile.*', '.*\.env.*', '.*secrets.*', '.*auth.*']) | some }}
    run:
      - action: add-label@v1
        args:
          label: "security-review-needed"
      - action: request-review@v1
        args:
          reviewers: [security-team]

  # Large PR warning
  large_pr:
    if:
      - {{ branch.diff.size > 500 }}
    run:
      - action: add-comment@v1
        args:
          comment: |
            This PR is large ({{ branch.diff.size }} lines changed).
            Consider splitting it for easier review.
      - action: add-label@v1
        args:
          label: "large-pr"

calc:
  etr: {{ branch.diff.size | estimatedReviewTime }}
```

### 4. AI-Powered PR Descriptions

Stop writing PR descriptions manually. These tools generate them from the diff:

**What The Diff** — Install the [GitHub App](https://whatthediff.ai/) and it auto-generates a description when a PR is opened.

**PR-Agent `/describe` command** — If you already have PR-Agent installed:

```yaml
# Add to your pr-agent workflow
- uses: Codium-ai/pr-agent@main
  env:
    OPENAI_KEY: ${{ secrets.OPENAI_API_KEY }}
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    command: describe
```

This generates:
- A structured summary of what changed and why
- A walkthrough of each changed file
- Labels based on the type of change

### 5. Security Scanning with AI Triage

Layer AI triage on top of your security tools to reduce alert fatigue:

**Semgrep with Assistant (AI triage):**

```yaml
# .github/workflows/semgrep.yml
name: Semgrep
on:
  pull_request: {}
  push:
    branches: [main]
jobs:
  semgrep:
    runs-on: ubuntu-latest
    container:
      image: semgrep/semgrep
    steps:
      - uses: actions/checkout@v4
      - run: semgrep ci
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}
```

Semgrep Assistant achieves 97% agreement with human security engineers on triage decisions — whether a finding is a true positive, false positive, or acceptable risk.

## Combining the Tools

Here's how these tools interact in a typical PR workflow:

```
Developer opens PR
  │
  ├─→ Danger.js checks structural rules (size, changelog, tests)
  ├─→ CodeRabbit/PR-Agent reviews code quality and logic
  ├─→ GitStream classifies PR and routes to reviewers
  ├─→ PR-Agent generates description and labels
  ├─→ Semgrep scans for security issues, AI triages findings
  │
  ▼
Human reviewer sees:
  - AI summary of the PR
  - Structural warnings from Danger.js
  - AI code review comments on specific lines
  - Security findings (pre-triaged, true positives only)
  - Estimated review time label
  │
  ▼
Reviewer approves → Tier 1 merge automation takes over
```

## Configuration Tips

### Reducing Noise

The biggest risk at Tier 2 is **alert fatigue**. Too many bot comments and developers ignore them all. Strategies:

1. **Start with one AI reviewer**, not three. CodeRabbit OR PR-Agent, not both.
2. **Use `chill` mode** on CodeRabbit, or configure PR-Agent to only flag medium+ severity.
3. **Danger.js: use `warn` sparingly, `fail` even more so.** Reserve `fail` for things that genuinely should block merge (missing tests for new files, security policy violations).
4. **Collapse bot comments** using GitHub's "resolve conversation" feature.
5. **Track signal-to-noise ratio.** If >50% of AI comments get dismissed without action, recalibrate.

### Cost Management

- **PR-Agent self-hosted**: You pay OpenAI/Anthropic API costs directly. Budget ~$0.05-0.30 per PR review depending on diff size and model.
- **CodeRabbit free tier**: Summaries only. Pro ($24/seat/mo) for full review.
- **Semgrep**: Free engine is enough for most teams. AppSec Platform adds AI triage.
- **Danger.js**: Free, runs in your CI minutes.
- **GitStream**: Free tier available.

### Privacy Considerations

| Tool | Code Sent To | Self-Hosted Option |
|------|-------------|-------------------|
| PR-Agent | Your LLM provider (OpenAI, Anthropic, Azure) | Yes |
| CodeRabbit | CodeRabbit's servers | No |
| Greptile | Greptile's servers (or self-hosted) | Yes |
| Danger.js | Nowhere (runs locally in CI) | N/A |
| GitStream | LinearB's servers | No |
| Semgrep | Code stays local; findings sent to Semgrep AppSec Platform | Engine is local |

If your org has strict data policies, the self-hosted stack is: **PR-Agent + Danger.js + Semgrep engine**.

## What This Gets You (Beyond Tier 1)

- **Faster review cycles** — AI catches the easy stuff before humans look
- **Better PR descriptions** — Generated automatically, consistent quality
- **Smarter routing** — PRs go to the right reviewers based on content
- **Security triage** — Only real issues surface, false positives suppressed
- **Structural enforcement** — Changelog, test, and size policies automated

## When to Move to Tier 3

You've outgrown Tier 2 when:
- You want agents to *write code*, not just review it
- You have a backlog of issues that could be solved by an agent autonomously
- CI failures are common and fixing them is repetitive
- You want to go from "issue filed" to "draft PR ready" without a human in the loop
- Your team is comfortable with agents making commits to branches

See [Tier 3: Autonomous Agents](tier-3-autonomous-agents.md).
