# Awesome Repo Automation [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of tools, frameworks, and resources for automating git repository management — from AI agents that make autonomous decisions about code changes to bots that handle merging, releasing, and security scanning.

## Contents

- [AI Coding Agents](#ai-coding-agents)
- [AI Code Review](#ai-code-review)
- [Autonomous Repo Management](#autonomous-repo-management)
- [Agent Frameworks](#agent-frameworks)
- [Dependency Automation](#dependency-automation)
- [Security Scanning](#security-scanning)
- [Merge Automation](#merge-automation)
- [Release Automation](#release-automation)
- [Testing Automation](#testing-automation)
- [Linting and Formatting](#linting-and-formatting)
- [Bot Frameworks](#bot-frameworks)
- [GitOps](#gitops)
- [Monorepo Tools](#monorepo-tools)
- [Developer Portals](#developer-portals)
- [Dev Environment Automation](#dev-environment-automation)
- [Conventional Commits](#conventional-commits)
- [Standards and Specifications](#standards-and-specifications)
- [Articles](#articles)
- [Books](#books)
- [Research Papers](#research-papers)
- [Guides](#guides)
- [Related Awesome Lists](#related-awesome-lists)

## Guides

_Step-by-step implementation guides organized by automation maturity level._

- [Tier 1: Policy-Based Automation](guides/tier-1-policy-based-automation.md) - Set-and-forget workflows using Renovate, Mergify, semantic-release, and branch protection — no AI required.
- [Tier 2: AI-Assisted Review and Triage](guides/tier-2-ai-assisted-review.md) - Add AI code review, PR classification, and security triage while keeping humans in the loop.
- [Tier 3: Autonomous Agents](guides/tier-3-autonomous-agents.md) - Issue-to-PR agents, self-healing CI, and custom agent orchestration with guardrails and safety nets.

## AI Coding Agents

_Autonomous agents that can plan, write, test, and deploy code changes across entire repositories._

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) - Anthropic's agentic CLI tool that reads files, writes code, runs commands, and manages multi-step workflows autonomously.
- [GitHub Copilot Coding Agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) - Assign GitHub issues to Copilot and it autonomously plans, codes, tests, and opens a draft PR.
- [Cursor](https://www.cursor.com/) - AI-native IDE with background agents that autonomously execute tasks, fix errors, and manage multi-file changes in parallel.
- [Devin](https://devin.ai/) - Fully autonomous AI software engineer with its own sandboxed environment for planning, coding, testing, and deploying.
- [SWE-agent](https://github.com/SWE-agent/SWE-agent) - Open-source autonomous agent from Princeton that takes a GitHub issue and automatically produces a fix using custom Agent-Computer Interfaces.
- [OpenHands](https://github.com/OpenHands/OpenHands) - Open-source platform for AI agents that modify code, execute commands, browse the web, and interact with APIs in sandboxed Docker environments.
- [Aider](https://github.com/paul-gauthier/aider) - Open-source terminal-based AI pair programmer that edits multiple files, auto-commits changes, and runs linters and tests.
- [Amazon Q Developer](https://aws.amazon.com/q/developer/) - AI assistant with specialized agents for code transformation, software development, documentation, and unit test generation.
- [Google Jules](https://jules.google) - Autonomous coding agent that clones repos into cloud VMs, works asynchronously, and delivers PRs with interactive planning.
- [OpenAI Codex CLI](https://github.com/openai/codex) - Open-source CLI agent for local terminal use with cloud sandbox delegation for larger tasks.
- [Augment Code](https://www.augmentcode.com/) - AI coding agent with a Context Engine that maintains live understanding of entire codebases, dependencies, and architecture.
- [Cody](https://sourcegraph.com/cody) - AI assistant by Sourcegraph that leverages code search for deep codebase understanding across monorepos and multi-repo setups.
- [Tabnine](https://www.tabnine.com/) - Privacy-first AI code assistant supporting fully air-gapped and on-premises enterprise deployments across 600+ languages.

## AI Code Review

_Tools that automatically review pull requests using AI, catching bugs, suggesting improvements, and enforcing standards._

- [CodeRabbit](https://www.coderabbit.ai/) - AI-first PR reviewer that builds a repository-wide code graph and provides line-by-line feedback, PR summaries, and agentic pre-merge checks.
- [Greptile](https://www.greptile.com/) - Codebase-aware AI reviewer that builds a full repository graph to assess the blast radius of every change.
- [Qodo Merge](https://www.qodo.ai/products/qodo-merge/) - AI code review agent with an open-source core (PR-Agent) that ranks issues by severity and supports slash commands for automation.
- [Ellipsis](https://www.ellipsis.dev/) - AI code review agent that catches bugs and auto-applies fixes, then verifies they compile and pass tests.
- [Sourcery](https://www.sourcery.ai/) - AI code reviewer focused on Python refactoring with intelligent suggestions that adapt to team feedback.
- [Bito AI](https://bito.ai/) - PR review agent combining LLM-powered analysis with static analysis tools for multi-angle code quality feedback.
- [SonarQube](https://www.sonarsource.com/products/sonarqube/) - Continuous code quality platform with 6,500+ rules, quality gates, and AI-powered fix suggestions across 35+ languages.
- [Codacy](https://www.codacy.com/) - Automated code review platform with static analysis, dependency scanning, and AI guardrails across 40+ languages.
- [DeepSource](https://deepsource.com/) - Unified DevSecOps platform with autonomous agents that observe every line, reason about changes, and create fix PRs.
- [What The Diff](https://whatthediff.ai/) - AI tool that generates PR descriptions, non-technical summaries for stakeholders, and public changelogs.
- [Graphite](https://graphite.com/) - Developer productivity platform with an AI code reviewer, stacked PRs, and an automated merge queue.
- [Cursor BugBot](https://www.cursor.com/) - Agentic AI code review built into the Cursor IDE and GitHub with a "Fix in Cursor" button for detected issues.

## Autonomous Repo Management

_Tools and capabilities for autonomous decision-making in repository workflows — auto-triage, auto-assign, auto-merge, and self-healing CI._

- [Mergify](https://mergify.com/) - YAML-driven merge automation platform with policy-based auto-merge, priority queues, freeze windows, and intelligent reviewer assignment.
- [GitAuto](https://gitauto.ai/) - AI agent that reads GitHub issues or Jira tickets, writes code, and creates pull requests in approximately three minutes.
- [Sweep AI](https://sweep.dev/) - AI junior developer that transforms GitHub issues into complete pull requests with tests and documentation.
- [Gitar](https://gitar.ai/) - Autonomous CI healing engine that reads CI logs, reproduces failures, applies fixes, and validates the pipeline returns to green.
- [Allstar](https://github.com/ossf/allstar) - GitHub App by OpenSSF that continuously enforces security best practices on repositories using policy-as-code.
- [PullApprove](https://www.pullapprove.com/) - Framework for code review assignment and policies with automatic reviewer routing based on flexible expressions.
- [GitStream](https://linearb.io/dev/gitstream) - Workflow automation for git repos that classifies PRs by type and routes them through appropriate review and merge policies.

### Auto-Merge Strategies

- [GitHub Merge Queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue) - Built-in GitHub feature that queues PRs and tests them against the latest base branch before merging.
- [Kodiak](https://github.com/chdsbd/kodiak) - Open-source GitHub bot that automatically updates and merges PRs when all branch protection requirements are met.
- [Bulldozer](https://github.com/palantir/bulldozer) - Open-source GitHub App by Palantir that merges PRs within seconds of all preconditions being satisfied.

### Self-Healing CI

- [Nx Cloud](https://nx.dev/) - Monorepo platform with self-healing CI that automatically analyzes failures, identifies root causes, and posts fixes in PRs.
- [Mabl](https://www.mabl.com/) - AI-powered test automation that analyzes DOM snapshots, network activity, and screenshots to self-heal tests when UI changes.

## Agent Frameworks

_General-purpose frameworks for building custom AI agents that automate repository and DevOps workflows._

- [CrewAI](https://github.com/crewAIInc/crewAI) - Multi-agent orchestration framework with role-based agents, sequential/hierarchical collaboration, and YAML-based workflow configuration.
- [LangGraph](https://github.com/langchain-ai/langgraph) - Graph-based agent orchestration framework with durable execution, human-in-the-loop controls, and multi-agent patterns.
- [Microsoft AutoGen](https://github.com/microsoft/autogen) - Multi-agent framework with graph-based workflows, human-in-the-loop patterns, and native CI/CD support via GitHub Actions.
- [Probot](https://github.com/probot/probot) - Framework for building GitHub Apps in Node.js that listen to webhook events and automate repository workflows.
- [Octokit](https://github.com/octokit) - Official GitHub API client SDKs for JavaScript, .NET, Ruby, and Python with complete REST and GraphQL API coverage.

## Dependency Automation

_Tools that automatically create pull requests to keep dependencies updated and secure._

- [Dependabot](https://github.com/dependabot) - GitHub-native dependency updater that opens PRs for security and version updates with compatibility scores and grouped updates.
- [Renovate](https://github.com/renovatebot/renovate) - Highly configurable open-source dependency update tool supporting 90+ package managers across GitHub, GitLab, Bitbucket, and Azure DevOps.
- [Snyk](https://snyk.io/) - Developer-first security platform that finds vulnerabilities in dependencies, containers, code, and IaC, with automated fix PRs.
- [Socket](https://socket.dev/) - Supply chain security platform that proactively detects malicious packages through behavioral analysis rather than just CVE databases.
- [Depfu](https://depfu.com/) - Dependency update service with intelligent PR drip-feeding that reduces update noise by maturing versions before suggesting them.
- [Mend](https://www.mend.io/) - Enterprise SCA platform and parent company of Renovate, providing vulnerability scanning across 200+ languages with automated fix PRs.
- [Sonatype Lifecycle](https://www.sonatype.com/products/open-source-security-dependency-management) - Enterprise SCA with Golden Pull Requests that guarantee zero-build-break dependency updates.
- [Scala Steward](https://github.com/scala-steward-org/scala-steward) - Open-source bot that keeps Scala/JVM dependencies and build plugins up-to-date with automatic code migrations via Scalafix.
- [Updatecli](https://github.com/updatecli/updatecli) - Declarative dependency management engine for custom update scenarios beyond standard package managers.
- [Aikido Security](https://www.aikido.dev/) - Unified application security platform combining SCA, SAST, DAST, container scanning, and secrets detection.

### CLI Audit Tools

- [npm-check-updates](https://github.com/raineorshine/npm-check-updates) - Find newer versions of Node.js dependencies with doctor mode that tests each upgrade individually.
- [pip-audit](https://github.com/pypa/pip-audit) - Audit Python environments for known vulnerabilities with automatic fix support.
- [cargo-audit](https://github.com/rustsec/rustsec) - Audit Rust Cargo.lock files against the RustSec Advisory Database with experimental auto-fix.
- [Trivy](https://github.com/aquasecurity/trivy) - Comprehensive open-source scanner for vulnerabilities, misconfigurations, secrets, and SBOMs across containers, repos, and Kubernetes.
- [OWASP Dependency-Check](https://github.com/dependency-check/DependencyCheck) - Open-source SCA utility that identifies publicly disclosed vulnerabilities by matching against the NVD.

### Dependency Monitoring

- [Libraries.io](https://libraries.io/) - Open-source discovery service indexing 9.9M+ packages across 36+ package managers with a SourceRank quality algorithm.
- [libyear](https://libyear.com/) - Simple metric measuring dependency freshness as a single number in years, usable as a CI quality gate.

## Security Scanning

### Secret Detection

- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning) - Built-in GitHub feature that detects tokens and credentials with push protection to block secrets before they land.
- [GitGuardian](https://www.gitguardian.com/) - Enterprise-grade secrets security with 450+ detectors, automated incident response, and honeytoken decoys for threat detection.
- [Gitleaks](https://github.com/gitleaks/gitleaks) - Fast, lightweight open-source secret scanner for Git repositories written in Go with custom regex support.
- [TruffleHog](https://github.com/trufflesecurity/trufflehog) - Deep secret scanner with 600+ detectors and automatic API validation that scans repos, S3 buckets, and Docker images.

### Static Analysis (SAST)

- [CodeQL](https://github.com/github/codeql) - GitHub's semantic code analysis engine that treats code as data via a purpose-built query language.
- [Semgrep](https://github.com/semgrep/semgrep) - Lightweight static analysis tool using pattern-based code matching with 20,000+ rules across 35+ languages.
- [Snyk Code](https://snyk.io/product/snyk-code/) - Developer-first SAST combining symbolic and generative AI for real-time vulnerability detection with auto-fix suggestions.
- [Bandit](https://github.com/PyCQA/bandit) - Security linter for Python that uses AST analysis to find common security issues like injection attacks and hardcoded secrets.

### Infrastructure as Code

- [Checkov](https://github.com/bridgecrewio/checkov) - Open-source IaC scanner with 1,000+ built-in policies covering Terraform, Kubernetes, CloudFormation, Helm, and Dockerfiles.

## Merge Automation

- [Mergify](https://mergify.com/) - Merge queue platform with batching, bisect-on-failure, priority queuing, freeze windows, and YAML-based merge rules.
- [GitHub Merge Queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue) - Native GitHub feature that creates temporary merge groups, runs CI, and merges sequentially.
- [Kodiak](https://github.com/chdsbd/kodiak) - Open-source GitHub bot that auto-merges PRs when configured labels are applied and all requirements are met.
- [Bulldozer](https://github.com/palantir/bulldozer) - Open-source event-driven merge bot by Palantir with dynamic merge method selection and PolicyBot integration.
- [Graphite](https://graphite.com/) - Stacked PR platform with an automated merge queue that keeps the main branch green.
- [Trunk Merge](https://trunk.io/) - Dynamic parallel merge queues based on PR impacted targets for monorepo-scale merging.
- [Aviator](https://www.aviator.co/) - Merge queue with bot-driven PR management and automated conflict resolution.

## Release Automation

- [semantic-release](https://github.com/semantic-release/semantic-release) - Fully automated versioning and publishing based on Conventional Commit messages with a modular plugin architecture.
- [Release Please](https://github.com/googleapis/release-please) - Google's release automation that maintains a living Release PR accumulating changes and changelogs until merged.
- [Changesets](https://github.com/changesets/changesets) - Monorepo-first versioning tool that decouples version intent from commit messages using explicit changeset files.
- [git-cliff](https://github.com/orhun/git-cliff) - Highly customizable Rust-powered changelog generator with Conventional Commits support and Jinja2-style templates.
- [Release Drafter](https://github.com/release-drafter/release-drafter) - GitHub Action that automatically drafts release notes as PRs are merged, categorizing changes by labels.
- [Auto](https://github.com/intuit/auto) - Release automation by Intuit using semantic version labels on pull requests instead of commit messages.

## Testing Automation

### Code Coverage

- [Codecov](https://about.codecov.io/) - Coverage reporting with detailed PR comments, branch comparisons, and visual coverage reports.
- [Coveralls](https://coveralls.io/) - Coverage tracking service with unlimited history and per-repository pricing, free for open source.

### Visual Testing

- [Chromatic](https://www.chromatic.com/) - Visual testing and UI review for Storybook component libraries with TurboSnap for testing only what changed.
- [Percy](https://percy.io/) - Automated visual regression testing by BrowserStack with cross-browser and cross-device screenshot comparisons.

### PR Automation

- [Danger.js](https://github.com/danger/danger-js) - Open-source tool that automates common code review chores via a Dangerfile with programmable rules for every PR.

### Mutation Testing

- [Stryker](https://stryker-mutator.io/) - Open-source mutation testing framework that inserts bugs into source code and verifies tests catch them, for JavaScript, .NET, and Scala.

### Flaky Test Detection

- [BuildPulse](https://buildpulse.io/) - Identifies flaky tests with quarantining, root cause analysis, and impact metrics for all major test frameworks.
- [Launchable](https://www.launchableinc.com/) - AI-driven predictive test selection that reduces test time by up to 80% with flakiness scoring.

## Linting and Formatting

- [pre-commit](https://github.com/pre-commit/pre-commit) - Multi-language package manager for Git pre-commit hooks that manages installation and execution of linters before every commit.
- [MegaLinter](https://github.com/oxsecurity/megalinter) - Aggregated linter suite analyzing 50+ languages with 100+ linters, auto-fixing, and parallel execution.
- [Trunk Check](https://trunk.io/) - Unified linting, formatting, and security scanning managing 100+ tools with a "Hold the Line" feature for enforcing rules on new code only.
- [Biome](https://github.com/biomejs/biome) - Rust-powered unified linter and formatter for web projects that is 10-20x faster than Node.js-based tools.
- [dprint](https://github.com/dprint/dprint) - Pluggable code formatting platform written in Rust using WebAssembly plugins, 20-60x faster than Prettier.
- [Super-Linter](https://github.com/super-linter/super-linter) - GitHub's multi-language linter aggregator that runs as a GitHub Action with a single Docker image.

## Bot Frameworks

- [Probot](https://github.com/probot/probot) - Framework for building GitHub Apps in Node.js with an event-driven architecture and 52+ pre-built bots.
- [Octokit](https://github.com/octokit) - Official GitHub API client SDKs for programmatic interaction with GitHub's REST and GraphQL APIs.
- [actions/stale](https://github.com/actions/stale) - GitHub Action that warns and closes issues and PRs after configurable periods of inactivity.
- [Lock Threads](https://github.com/dessant/lock-threads) - GitHub Action that locks closed issues, PRs, and discussions after inactivity to prevent necrobumping.
- [actions/labeler](https://github.com/actions/labeler) - Official GitHub Action that automatically labels PRs based on changed file paths or branch names.
- [actions/first-interaction](https://github.com/actions/first-interaction) - Official GitHub Action that welcomes first-time contributors when they open their first issue or PR.

## GitOps

- [Argo CD](https://github.com/argoproj/argo-cd) - Declarative GitOps continuous delivery tool for Kubernetes with a rich web UI, multi-cluster management, and progressive delivery.
- [Flux CD](https://github.com/fluxcd/flux2) - GitOps toolkit for keeping Kubernetes clusters in sync with Git, Helm, and OCI sources using composable controllers.
- [Atlantis](https://github.com/runatlantis/atlantis) - Self-hosted tool that automates Terraform plan and apply through pull request workflows with state locking.
- [Crossplane](https://github.com/crossplane/crossplane) - Kubernetes-native framework for managing cloud infrastructure declaratively with continuous reconciliation and drift correction.

## Monorepo Tools

- [Nx](https://github.com/nrwl/nx) - Polyglot monorepo platform with intelligent caching, affected analysis, distributed task execution, and self-healing CI.
- [Turborepo](https://github.com/vercel/turborepo) - High-performance build system for JavaScript/TypeScript monorepos with content-aware hashing and remote caching.
- [Lerna](https://github.com/lerna/lerna) - Tool for managing JavaScript monorepos specializing in versioning and npm publishing, now powered by Nx.
- [moon](https://github.com/moonrepo/moon) - Rust-based monorepo management tool inspired by Bazel with an integrated toolchain and deterministic builds.

## Developer Portals

- [Backstage](https://github.com/backstage/backstage) - Open-source framework by Spotify for building internal developer portals with a software catalog, templates, and plugin ecosystem.
- [Port](https://www.port.io/) - No-code internal developer portal with a universal software catalog, self-service actions, and custom dashboards.
- [Cortex](https://www.cortex.io/) - AI-powered internal developer portal focused on enforcing reliability standards with scorecards and automated insights.

## Dev Environment Automation

- [Dev Containers](https://containers.dev/) - Open specification for using containers as full-featured development environments with composable features and templates.
- [GitHub Codespaces](https://github.com/features/codespaces) - Cloud-hosted development environments built into GitHub with instant setup and scalable VM configurations.
- [Gitpod](https://gitpod.io/) - Cloud development environments that spin up pre-configured, containerized environments from Git repositories.

## Conventional Commits

_Tools for enforcing and working with the [Conventional Commits](https://www.conventionalcommits.org/) specification._

- [commitlint](https://github.com/conventional-changelog/commitlint) - Linter that validates commit messages against Conventional Commits format with Husky git hook integration.
- [Commitizen](https://github.com/commitizen/cz-cli) - Interactive CLI that prompts developers to create properly formatted commit messages following configurable conventions.
- [Husky](https://github.com/typicode/husky) - Modern native Git hooks manager for enforcing commit conventions and running pre-commit checks.
- [lint-staged](https://github.com/lint-staged/lint-staged) - Run linters on staged Git files, commonly paired with Husky for pre-commit formatting and validation.

## Standards and Specifications

- [Semantic Versioning](https://semver.org/) - Versioning scheme using MAJOR.MINOR.PATCH to communicate the nature of changes.
- [Keep a Changelog](https://keepachangelog.com/) - Convention for maintaining human-readable changelogs with Added, Changed, Deprecated, Removed, Fixed, and Security categories.
- [OpenSSF Scorecard](https://github.com/ossf/scorecard) - Automated tool that evaluates open-source projects against security best practices, assigning scores from 0-10.
- [SLSA](https://slsa.dev/) - Supply-chain Levels for Software Artifacts framework by Google for securing the software supply chain at multiple levels.
- [OpenSSF Best Practices Badge](https://www.bestpractices.dev/) - Self-certification program for open-source projects to report adherence to security best practices across three tiers.
- [in-toto](https://in-toto.io/) - Framework ensuring software supply chain integrity by making transparent what steps were performed, by whom, and in what order.
- [Sigstore](https://sigstore.dev/) - Automated digital signing and verification of software artifacts with keyless identity-based signing.
- [CycloneDX](https://cyclonedx.org/) - OWASP standard for Software Bill of Materials designed for security use cases and vulnerability tracking.
- [SPDX](https://spdx.dev/) - Linux Foundation standard for Software Bill of Materials with rich metadata for license compliance.

## Articles

- [Eight Trends Defining How Software Gets Built in 2026](https://www.anthropic.com/research/2026-agentic-coding-trends) - Anthropic's analysis of how engineers are shifting from writing code to coordinating AI agents.
- [The Autonomous Enterprise and the Four Pillars of Platform Control](https://www.cncf.io/blog/2026/01/23/the-autonomous-enterprise-and-the-four-pillars-of-platform-control-2026-forecast/) - CNCF's framework for golden paths, guardrails, safety nets, and manual review in autonomous development.
- [Pull Request Best Practices](https://articles.mergify.com/pull-request-best-practices/) - Mergify's guide to modern PR workflows, automation rules, and merge queues.
- [Best Practices for Repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories) - GitHub's official guide covering README files, CODEOWNERS, licensing, and security features.
- [OWASP Software Supply Chain Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html) - Comprehensive guide to securing dependencies, build systems, and update channels.
- [Trunk-Based Development](https://trunkbaseddevelopment.com/) - Canonical resource for trunk-based development practices used by Google, Netflix, and Microsoft.
- [DORA DevOps Research](https://dora.dev/) - Research demonstrating deployment frequency, lead time, change failure rate, and time to restore as key DevOps metrics.

## Books

- [Accelerate](https://itrevolution.com/product/accelerate/) - Research-backed argument for DevOps identifying 24 capabilities that drive improvement, by Nicole Forsgren, Jez Humble, and Gene Kim.
- [Continuous Delivery](https://continuousdelivery.com/) - Foundational text on deployment pipelines, automated testing, and release engineering, by Jez Humble and David Farley.
- [The DevOps Handbook](https://itrevolution.com/product/the-devops-handbook-second-edition/) - Practical guide to integrating development, QA, IT operations, and security, by Gene Kim, Jez Humble, Patrick Debois, and John Willis.
- [Site Reliability Engineering](https://sre.google/sre-book/) - How Google runs production systems covering SLOs, error budgets, and incident response, available free online.
- [Team Topologies](https://teamtopologies.com/) - Framework for organizing teams for fast flow with four team types and three interaction modes, by Matthew Skelton and Manuel Pais.
- [Infrastructure as Code](https://infrastructure-as-code.com/) - Patterns and practices for managing infrastructure with automation tools, by Kief Morris.

## Research Papers

- [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793) - Princeton NeurIPS 2024 paper on custom ACI design for LLM agents interacting with codebases.
- [OpenHands: An Open Platform for AI Software Developers as Generalist Agents](https://arxiv.org/abs/2407.16741) - Multi-agent platform with delegation, sandboxing, and model-agnostic design.
- [The Rise of AI Teammates in Software Engineering 3.0](https://arxiv.org/abs/2507.15003) - Documents 400K+ PRs by AI agents and adoption rates of 15-22% on GitHub.

## Related Awesome Lists

- [awesome-actions](https://github.com/sdras/awesome-actions) - Curated list of awesome GitHub Actions.
- [awesome-ci](https://github.com/ligurio/awesome-ci) - Comprehensive list of continuous integration services and tools.
- [awesome-devops](https://github.com/wmariuss/awesome-devops) - DevOps platforms, tools, practices, and resources.
- [awesome-gitops](https://github.com/weaveworks/awesome-gitops) - GitOps resources covering Argo CD, Flux, and progressive delivery.
- [awesome-code-review](https://github.com/joho/awesome-code-review) - Articles, papers, tools, and resources for code review.
- [awesome-bots](https://github.com/DopplerHQ/awesome-bots) - Comprehensive list of bot frameworks, SDKs, and tools.

## Contributing

Contributions welcome! Read the [contribution guidelines](contributing.md) first.
