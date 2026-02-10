# Tier 3: Autonomous Agents

> AI agents that independently write code, fix bugs, heal CI, and open pull requests — with humans reviewing the output rather than doing the work. This is the frontier of repo automation.

## Prerequisites

You need [Tier 1](tier-1-policy-based-automation.md) (solid CI, branch protection, auto-merge policies) and [Tier 2](tier-2-ai-assisted-review.md) (AI review to catch issues in agent-generated code) before going autonomous. Without these safety layers, autonomous agents can merge broken code.

The key insight: **Tier 1 and 2 become your safety net for Tier 3.** When an agent opens a PR, your AI reviewer checks it, your CI validates it, and your merge policies gate it. The tiers compose.

## Architecture Principles

### The Golden Rule: Agents Produce Draft PRs, Never Merge Directly

Every autonomous action should result in a pull request, never a direct push to main. This gives you:
- A reviewable artifact (the PR diff)
- CI validation before merge
- An audit trail
- A "reject" button

### The CNCF Four Pillars

The [CNCF's 2026 framework](https://www.cncf.io/blog/2026/01/23/the-autonomous-enterprise-and-the-four-pillars-of-platform-control-2026-forecast/) for autonomous development provides the right mental model:

1. **Golden Paths** — Agents follow prescribed workflows (issue templates, PR templates, branch naming conventions)
2. **Guardrails** — Branch protection, required reviews, CI checks that agents cannot bypass
3. **Safety Nets** — Automated rollback, feature flags, canary deployments
4. **Manual Review** — Humans approve the risky stuff; agents handle the routine

### Risk Stratification

Not all tasks should be automated equally:

| Risk Level | Examples | Agent Policy |
|-----------|----------|-------------|
| **Low** | Fix typos, update docs, bump patch dependencies | Agent commits → auto-merge if CI passes |
| **Medium** | Fix a bug with a clear reproduction, add a test | Agent commits → human review → merge |
| **High** | Refactor core logic, change APIs, update auth | Agent drafts a plan → human approves plan → agent implements → human reviews code |
| **Off-limits** | Delete data, modify permissions, change infrastructure | Humans only |

## Option 1: GitHub Copilot Coding Agent

**The simplest path to issue-to-PR automation on GitHub.**

### Setup

1. Enable Copilot for your organization
2. The coding agent is available for Copilot Enterprise and Copilot Pro+ subscribers
3. Assign an issue to Copilot by using `@github-copilot` or assigning the `copilot` user

### How It Works

1. You assign a GitHub issue to Copilot
2. Copilot creates a branch, reads the codebase, plans the approach
3. It writes code, runs tests, iterates on failures
4. It opens a draft PR linked to the issue
5. You review and merge (or request changes)

### Best Practices

- **Write detailed issue descriptions.** The agent's output quality is directly proportional to issue clarity. Include acceptance criteria, expected behavior, and affected files.
- **Use issue templates** to enforce structure:

```markdown
<!-- .github/ISSUE_TEMPLATE/agent-task.md -->
---
name: Agent Task
about: A task suitable for AI agent implementation
labels: ["agent-eligible"]
---

## Description
<!-- Clear description of what needs to change -->

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Affected Files
<!-- List known files, or "agent should determine" -->

## Testing
<!-- How to verify the change works -->
```

- **Label issues appropriately.** Use labels like `good-first-issue`, `bug`, `documentation` to signal agent-eligible tasks.

### Limitations

- Works within a single repository
- Cannot run arbitrary external services
- May struggle with tasks requiring deep architectural understanding
- Limited to the context window (very large codebases may exceed it)

## Option 2: SWE-agent (Open Source, Self-Hosted)

**Best for: teams that want full control and customization.**

### Setup via GitHub Actions

```yaml
# .github/workflows/swe-agent.yml
name: SWE-agent Auto Fix
on:
  issues:
    types: [labeled]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  agent-fix:
    if: github.event.label.name == 'agent-fix'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install SWE-agent
        run: |
          pip install sweagent

      - name: Run SWE-agent
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          sweagent run \
            --agent.model.name claude-sonnet-4-5-20250929 \
            --agent.model.per_instance_cost_limit 2.00 \
            --env.repo.github_url ${{ github.event.repository.html_url }} \
            --problem_statement.github_url ${{ github.event.issue.html_url }}

      - name: Create PR from patch
        if: success()
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          BRANCH="agent/fix-issue-${{ github.event.issue.number }}"
          git checkout -b "$BRANCH"
          git add -A
          git commit -m "fix: address #${{ github.event.issue.number }}

          Automated fix generated by SWE-agent."
          git push origin "$BRANCH"
          gh pr create \
            --title "fix: ${{ github.event.issue.title }}" \
            --body "Automated fix for #${{ github.event.issue.number }}.

          Generated by SWE-agent. Please review carefully." \
            --draft \
            --label "agent-generated"
```

### Key Configuration

- **`per_instance_cost_limit`**: Cap API costs per issue. Start at $2.00 and adjust.
- **`timeout-minutes`**: Prevent runaway agents. 30 minutes is generous for most fixes.
- **Label trigger**: Using `agent-fix` as the trigger label keeps humans in control of when the agent runs.

## Option 3: OpenHands (Open Source, Full Environment)

**Best for: tasks that need a full development environment (install packages, run servers, browse docs).**

### Setup

```yaml
# .github/workflows/openhands.yml
name: OpenHands Agent
on:
  issues:
    types: [labeled]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  openhands:
    if: github.event.label.name == 'agent-fix'
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - name: Run OpenHands Resolver
        uses: All-Hands-AI/OpenHands@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          LLM_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          LLM_MODEL: anthropic/claude-sonnet-4-5-20250929
        with:
          issue_number: ${{ github.event.issue.number }}
```

OpenHands spins up a sandboxed Docker environment where the agent can:
- Install dependencies
- Run test suites
- Execute build commands
- Browse documentation
- Make multi-file changes

## Option 4: Self-Healing CI with Gitar

**Best for: automatically fixing CI failures without human intervention.**

Instead of an agent waiting for issues, this approach reacts to CI failures:

```yaml
# .github/workflows/self-heal.yml
name: Self-Healing CI
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

permissions:
  contents: write
  pull-requests: write

jobs:
  heal:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.workflow_run.head_branch }}

      - name: Download CI logs
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh run view ${{ github.event.workflow_run.id }} --log > ci-log.txt

      - name: Analyze and fix
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # Use Claude Code or your agent of choice to analyze
          # the failure log and attempt a fix.
          #
          # This is a simplified example. In practice you would
          # call your agent framework here with the log context.
          echo "Analyzing CI failure..."
          # agent analyze ci-log.txt --auto-fix

      - name: Push fix if changes made
        run: |
          if [ -n "$(git status --porcelain)" ]; then
            git add -A
            git commit -m "fix: auto-heal CI failure

            Automated fix based on CI log analysis."
            git push
          fi
```

[Gitar](https://gitar.ai/) provides this as a managed service — it reads CI logs, reproduces failures locally, applies fixes, and validates the pipeline returns to green.

## Option 5: Custom Agent Orchestration

For teams that want to build their own agent workflows, use a framework:

### CrewAI Example: Issue Triage Agent

```python
from crewai import Agent, Task, Crew

triager = Agent(
    role="Issue Triager",
    goal="Classify GitHub issues and route them appropriately",
    backstory="You are an experienced developer who understands codebase architecture.",
    tools=[github_api_tool, codebase_search_tool],
)

fixer = Agent(
    role="Bug Fixer",
    goal="Fix bugs described in GitHub issues",
    backstory="You write clean, tested code that follows existing patterns.",
    tools=[file_editor_tool, test_runner_tool, git_tool],
)

triage_task = Task(
    description="Analyze issue #{issue_number}. Determine if it's a bug, feature, or question. "
                "Identify affected files and estimate complexity (low/medium/high).",
    agent=triager,
    expected_output="Classification with affected files and complexity estimate.",
)

fix_task = Task(
    description="If the issue is a low-complexity bug, implement a fix with tests. "
                "If it's medium or high complexity, write a detailed implementation plan instead.",
    agent=fixer,
    expected_output="Either a code fix with tests, or an implementation plan.",
    context=[triage_task],
)

crew = Crew(
    agents=[triager, fixer],
    tasks=[triage_task, fix_task],
    verbose=True,
)
```

### LangGraph Example: PR Review Pipeline

```python
from langgraph.graph import StateGraph

def classify_pr(state):
    """Classify PR by risk level based on files changed."""
    # Analyze diff, return risk level
    ...

def ai_review(state):
    """Run AI code review on the diff."""
    ...

def security_scan(state):
    """Run security analysis on changed files."""
    ...

def human_gate(state):
    """Route to human review if risk is medium+."""
    ...

def auto_approve(state):
    """Auto-approve low-risk changes."""
    ...

graph = StateGraph()
graph.add_node("classify", classify_pr)
graph.add_node("ai_review", ai_review)
graph.add_node("security_scan", security_scan)
graph.add_node("human_gate", human_gate)
graph.add_node("auto_approve", auto_approve)

graph.add_edge("classify", "ai_review")
graph.add_edge("classify", "security_scan")
graph.add_conditional_edges(
    "ai_review",
    lambda state: "human_gate" if state["risk"] != "low" else "auto_approve",
)
```

## Guardrails and Safety

### Cost Controls

Autonomous agents can be expensive. Set hard limits:

```yaml
# In your workflow
env:
  # Per-issue cost cap
  AGENT_COST_LIMIT: "3.00"
  # Monthly budget
  AGENT_MONTHLY_BUDGET: "500.00"
  # Max issues per day
  AGENT_DAILY_LIMIT: "10"
```

Track costs by logging API usage per workflow run and aggregating in your monitoring.

### Preventing Runaway Agents

1. **Timeout every workflow** (`timeout-minutes: 30`)
2. **Cap iterations** — most agent frameworks support a max-turns parameter
3. **Sandbox execution** — OpenHands uses Docker; SWE-agent can be containerized
4. **Never give agents write access to main** — they push to feature branches only
5. **Require CI + human approval** before any agent branch merges

### What Agents Should Never Do

- Push directly to `main` or release branches
- Modify CI/CD pipeline definitions (`.github/workflows/`)
- Change access permissions or secrets
- Delete branches, issues, or data
- Modify security-critical code (auth, encryption, access control) without human review
- Deploy to production

Enforce this with branch protection rules and `CODEOWNERS`:

```
# CODEOWNERS
# These files always require human review
.github/workflows/   @platform-team
*.yml                @platform-team
src/auth/            @security-team
src/infrastructure/  @platform-team
```

### Labeling Agent-Generated Code

Always label agent PRs so reviewers know what they're looking at:

```yaml
# In your agent workflow, add labels
- name: Label PR
  run: gh pr edit --add-label "agent-generated,needs-human-review"
```

Reviewers should apply **extra scrutiny** to agent-generated code. Common failure modes:
- Correct logic but poor error handling
- Tests that pass but don't actually test the right thing
- Changes that work in isolation but break integration
- Subtle security issues (the agent "solved" the problem but introduced a vulnerability)

## Monitoring and Observability

Track these metrics for your autonomous agents:

| Metric | What It Tells You | Target |
|--------|-------------------|--------|
| **Success rate** | % of agent PRs that merge without revision | > 60% |
| **Revision rate** | % of agent PRs that need human edits | < 30% |
| **Rejection rate** | % of agent PRs closed without merging | < 10% |
| **Time to merge** | Average time from agent PR to merge | < 24 hours |
| **Cost per PR** | Average API cost per agent-generated PR | < $3.00 |
| **CI pass rate** | % of agent PRs that pass CI on first push | > 80% |
| **False fix rate** | % of "fixes" that don't actually solve the issue | < 15% |

If your rejection rate exceeds 20%, the agent is wasting reviewer time. Recalibrate by:
- Improving issue descriptions
- Narrowing the scope of tasks assigned to agents
- Switching to a more capable model
- Adding better context (CLAUDE.md, repository documentation)

## The Complete Tier 3 Workflow

```
Issue filed with "agent-fix" label
  │
  ├─→ Agent reads issue, analyzes codebase
  ├─→ Agent writes code, runs tests locally
  ├─→ Agent opens draft PR
  │
  ▼
Tier 2 kicks in:
  ├─→ AI code review (CodeRabbit/PR-Agent)
  ├─→ Danger.js structural checks
  ├─→ Security scanning
  │
  ▼
Tier 1 kicks in:
  ├─→ CI runs full test suite
  ├─→ Branch protection enforced
  │
  ▼
Human reviewer:
  ├─→ Reviews with AI-provided context
  ├─→ Approves or requests changes
  │
  ▼
Merge automation:
  └─→ Squash merge, delete branch, close issue

CI failure detected on any branch
  │
  ├─→ Self-healing agent reads logs
  ├─→ Pushes fix to same branch
  ├─→ CI re-runs automatically
  │
  ▼
If fix fails → alert human
```

## Getting Started

**Week 1:** Install GitHub Copilot Coding Agent (or OpenHands). Assign one simple, well-described bug to the agent. Review the PR carefully. Learn how the agent thinks.

**Week 2-3:** Assign 3-5 more issues. Develop a sense for what the agent handles well (clear bugs, test additions, docs) vs. poorly (vague requirements, architectural decisions).

**Week 4:** Set up the self-healing CI workflow. Let it run on a non-critical repo first.

**Month 2:** Add label-triggered agent workflows for your main repos. Define your risk stratification policy. Start tracking metrics.

**Month 3+:** Build custom agent orchestration if the off-the-shelf tools don't cover your needs. Consider multi-agent setups where one agent triages and another implements.

## What This Gets You (Beyond Tier 2)

- **Issue-to-PR in minutes** for well-defined tasks
- **Self-healing CI** that fixes flaky tests and trivial failures
- **Faster iteration cycles** — humans review output instead of writing everything
- **Lower barrier to contribution** — file an issue, get a PR
- **Scalable bug fixing** — agents work in parallel on multiple issues
