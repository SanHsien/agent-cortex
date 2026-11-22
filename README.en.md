# agent-cortex

**Language: English | [繁體中文](README.md)**

**Windows-first control plane for governed AI coding workflows.**

`paulsha-cortex` gathers the parts of a multi-agent engineering process that are easiest to distort into one vendor-neutral control plane. From work specification, dispatch and retry through deterministic verification, independent review, delivery and completion evidence, a single lifecycle authority manages all of it.

It is not a chatbot and it is not another IDE. Cortex coordinates headless executors that are already installed and signed in (for example `copilot`, `claude` and `codex`). It requires that "done" is backed by verifiable Candidate, Verification, Review and Delivery evidence, rather than trusting an agent's self-reported success.

> **Fork status**
> `SanHsien/agent-cortex` is the native-Windows development distribution of [`hamanpaul/paulsha-cortex`](https://github.com/hamanpaul/paulsha-cortex). It keeps Linux/systemd compatibility, but treats native Windows 11 + PowerShell + Python as the first-class development and CI path. This fork is for researching, developing and validating agent engineering governance; it does not currently claim to be production-ready.

## Why Cortex?

When multiple coding agents, worktrees, reviewers and retries coexist, the most dangerous problem is not that "the model cannot write code". It is that different tools each claim that their local state is the overall truth. Cortex converges these states into one verifiable lifecycle:

```text
intent (human-reviewed exact SHA; project artifact)
    ↓
spec / confirmed Todo authority
    ↓
plan → dispatch → candidate
                    ↓
             deterministic verify
                    ↓
             independent review
                    ↓
                 delivery
                    ↓
             completion record
```

Core principles:

- **Single lifecycle authority**: Cortex manages the state of `work / WorkflowRun / Job / Slice`; domain tools only provide artifacts.
- **Intent separated from dispatch authority**: `intent.md` holds the problem, goals and constraints; only a confirmed Todo/spec/plan authority can make work claimable.
- **Executor-neutral**: builders and reviewers can come from different headless executors, so the workflow is not tied to a single model vendor.
- **Evidence before completion**: process exit code 0 does not mean the task is done; verification, review and target-branch delivery must agree.
- **Fail-closed delivery**: when authority, repo, Candidate, PR, checks or review evidence cannot be shown to be consistent, the work is not projected as complete.
- **Operator-visible recovery**: `needs_human`, retry, request logs, inspect and doctor all give explicit recovery entry points, so nobody has to edit runtime state files directly.

## What the SanHsien fork changes

Compared with upstream, this fork specifically brings the native Windows path up to first-class capability:

- A Windows 11 per-user Startup service backend, with no administrator rights or WSL required.
- A loopback TCP monitor transport, corresponding to the Unix-socket path on Linux.
- Windows process wrapper, PID / file-lock / durability / path-safety and other platform reliability fixes.
- A PowerShell bootstrap and a canonical development gate.
- Windows-specific regression tests, GitHub Actions, CodeQL, and dependency / upstream tracking.
- Continued Linux/systemd compatibility; the bubblewrap foreign-review sandbox is still explicitly Linux-only.

For the detailed fork boundaries, licensing limits and upstream maintenance strategy, see [docs/FORK.md](docs/FORK.md) and [docs/UPSTREAM.md](docs/UPSTREAM.md).

## Platform support

| Capability | Windows 11 | Linux |
| --- | --- | --- |
| manager / monitor / workflow / review evidence | Native support | Supported |
| Background service | per-user Startup + PID/lock | systemd `--user` |
| monitor transport | loopback TCP endpoint manifest | Unix socket |
| Full pytest / build | Windows CI is the authoritative gate | Linux CI compatibility gate |
| bubblewrap foreign-review sandbox | Not supported, explicitly skipped | Supported |

## Install

Requirements: Python 3.10–3.13, Git, and at least one installed and signed-in headless executor CLI. For Windows-first use, PowerShell 7 is also recommended.

Installation for evaluation:

```powershell
pipx install git+https://github.com/SanHsien/agent-cortex.git
cortex --help
cortex --version
```

For formal acceptance, do not treat the mutable `main` as a fixed dependency; pin to a commit SHA that has passed the fork's CI:

```powershell
pipx install "git+https://github.com/SanHsien/agent-cortex.git@<commit-sha>"
```

You can also install directly after cloning:

```powershell
python -m pip install .
```

This fork does not currently publish a wheel / sdist. Upstream has no LICENSE at its repository root; a GitHub fork relationship by itself does not grant general redistribution or derivative-work rights. Until upstream grants explicit permission, this fork does not redistribute derived packages publicly.

## Getting started

If you are new to Cortex, read the documents below in order. This section is a stable index for the onboarding contract; the README itself does not duplicate the full operations manual:

1. [Quickstart](docs/onboarding/quickstart.md) — from installation and preflight through `cortex bootstrap` to your first workflow.
2. [Concepts](docs/onboarding/concepts.md) — the terms `spec`, `job`, `slice`, `work`, and the lifecycle.
3. [Admin](docs/onboarding/admin.md) — day-to-day operation of `cortex service`, `cortex inspect`, `cortex request`, model profiling and digest delivery.
4. [Runbook](docs/onboarding/runbook.md) — SOPs for manager degraded, timeout, executor and recovery.
5. [Troubleshooting](docs/onboarding/troubleshooting.md) — a quick lookup of common failures.
6. [Upgrade](docs/onboarding/upgrade.md) — upgrading and updating the pipx snapshot.
7. [Rollback](docs/onboarding/rollback.md) — returning to the last known good version.

Further reading: [Intent contract](docs/intent-contract.md), [Development](docs/DEVELOPMENT.md), [Unified Work Lifecycle](docs/unified-work-lifecycle.md), [Monitor config](docs/monitor-config.md), [Fork maintenance](docs/FORK.md), [Upstream ledger](docs/UPSTREAM.md), [Decisions](docs/DECISIONS.md).

## Usage

### Shortest path to get started

Use a dry run first to confirm the environment, then bootstrap Cortex:

```powershell
cortex bootstrap --dry-run
cortex bootstrap --instance cortex --repo-root (git rev-parse --show-toplevel)
```

Create a first sample workflow that stays at `dispatch: hold`:

```powershell
cortex init-sample --task "example feature" --change example-feature
```

First inspect the generated spec: change the glob plan to exact paths, and fill in `target_branch` and `verification`. Once everything is correct, **explicitly change `dispatch: hold` to `dispatch: auto`**. Only a slice with `dispatch: auto` and all required fields complete enters the ready set.

Next, check readiness and the runtime:

```powershell
cortex ready --specs-dir "$HOME/.agents/specs"
cortex status
cortex inspect doctor --json
cortex service status --instance cortex --json
```

Then run a full tick with the builder and reviewer named explicitly:

```powershell
cortex tick `
  --specs-dir "$HOME/.agents/specs" `
  --executor codex `
  --model "<builder-model-id>" `
  --review-executor claude `
  --review-model "<reviewer-model-id>"
```

> `cortex status` is the workflow / gate authority; `cortex service status` only answers whether the background service is alive. Neither can substitute for the other.

### Daily operations

```powershell
cortex jobs
cortex stat "$JOB_ID"
cortex request list --json
cortex request show "$REQUEST_ID"
cortex request logs "$REQUEST_ID" --json
cortex inspect status --json
cortex inspect job "$JOB_ID"
cortex inspect work <work-id> --repo owner/repo
```

When you hit `needs_human`, first read `attention[].next_actions` from `cortex status`, then use the matching operator action; do not edit `jobs.json` or other runtime authority files directly.

### Document map

| Need | Document |
| --- | --- |
| Create the first workflow | [Quickstart](docs/onboarding/quickstart.md) |
| Terms: spec / job / slice / work | [Concepts](docs/onboarding/concepts.md) |
| Intent format, human approval and authority boundaries | [Intent contract](docs/intent-contract.md) |
| service / inspect / request / model profiling / digest delivery | [Admin](docs/onboarding/admin.md) |
| Unified work read model, delivery closure, migration | [Unified Work Lifecycle](docs/unified-work-lifecycle.md) |
| monitor config precedence and ambient projects | [Monitor config](docs/monitor-config.md) |
| Common incidents and recovery | [Runbook](docs/onboarding/runbook.md) |
| Upgrade | [Upgrade](docs/onboarding/upgrade.md) |
| Rollback | [Rollback](docs/onboarding/rollback.md) |
| Troubleshooting | [Troubleshooting](docs/onboarding/troubleshooting.md) |
| Windows / Linux development | [Development](docs/DEVELOPMENT.md) |
| Fork differences and adoption limits | [Fork maintenance](docs/FORK.md) |
| Upstream evaluation watermark | [Upstream ledger](docs/UPSTREAM.md) |
| Current engineering decisions | [Decisions](docs/DECISIONS.md) |

The complete lifecycle, delivery, model profiling, monitor, digest and migration details are divided among the active documents above; the README is no longer a second admin / architecture manual.

## Security and current boundaries

- There is no Web UI; work intent is still carried mainly by `intent.md`, specs, Markdown and structured files.
- `cortex-intent/v1` is currently a docs/schema-first contract. It does not add `kind=intent`, automatic claim or dispatch.
- The sanitized environment used for verification is not a network or filesystem sandbox.
- Windows does not provide the Linux bubblewrap foreign-review sandbox; that capability is explicitly skipped, not passed off as isolated.
- v1 automatic foreign review is still bound by tier / policy boundaries.
- Executors must be installed, signed in and managed by the user; Cortex does not install or sign in to model CLIs on your behalf.
- This fork tracks a fast-moving upstream; before production adoption, re-review [docs/FORK.md](docs/FORK.md) and [docs/UPSTREAM.md](docs/UPSTREAM.md). Do not treat a green CI run alone as proof of maturity.

## Development

Windows-first development entry points:

```powershell
pwsh -File tools/bootstrap_dev.ps1
pwsh -File tools/dev_check.ps1 -Quick
```

Full tests:

```powershell
python -m pytest tests/ -q
```

CI also covers tests, persona scope, policy check, CodeQL, and the dependency / upstream maintenance workflows. Before making changes, read [AGENTS.md](AGENTS.md) and `.project-policy.yml`; Markdown in this repo is also part of the policy `code_paths`, so PRs must follow the changelog fragment and policy gate.

## Version

The sole authority for the current source version is [`VERSION`](VERSION); do not write a second, easily stale version number in the README. Synchronization between tag / release and the source version is verified by the repo policy and the release workflow.

This fork and upstream will keep diverging and syncing selectively. To judge what the fork currently adds and which upstream content it lags behind, look at [docs/FORK.md](docs/FORK.md), [docs/UPSTREAM.md](docs/UPSTREAM.md) and the GitHub compare view, rather than inferring from a historical README snapshot.

## Related tools

These five repos each govern one layer of AI coding. They can be used on their own or stacked:

| Layer | Repo | What it does |
| --- | --- | --- |
| Supply chain | [SkillSpector](https://github.com/SanHsien/SkillSpector) | Scan before you install: detects vulnerabilities and malicious patterns in agent skills, and outputs a risk score and a `SAFE` / `CAUTION` / `DO_NOT_INSTALL` recommendation |
| Dispatch decision | [agent-advisor](https://github.com/SanHsien/agent-advisor) | Risk-tiered routing `solo` / `delegate` / `audit` / `full`: decides whether this task should be delegated, and to whom |
| Action interception | [harness-guard](https://github.com/SanHsien/harness-guard) | Agent runtime hooks that actually intercept dangerous commands, unevidenced claims and red-light commits before and after an action and at wrap-up |
| Output quality | [ai-quality-gates](https://github.com/SanHsien/ai-quality-gates) | Executable specs and quantified thresholds: coverage, mutation testing, cyclomatic complexity, dependency structure, bounded loop policy |
| Delivery process | **paulsha-cortex (you are here)** | Multi-agent lifecycle: Candidate → Verify → Independent Review → Delivery → CompletionRecord |

Adjacent but a different layer: [agent-governance-toolkit](https://github.com/SanHsien/agent-governance-toolkit) governs agents that run autonomously after going live (policy enforcement, zero-trust identity, sandboxed execution and auditable records), not coding agents that write programs. [opencodex](https://github.com/SanHsien/opencodex) is a provider proxy that decides which LLMs can run behind these agents; it does not itself constrain agent behavior.

## Provenance

Upstream: [`hamanpaul/paulsha-cortex`](https://github.com/hamanpaul/paulsha-cortex).

This repository is a maintained Windows-first fork. The upstream governance model and much of the core runtime remain upstream's work; this fork's public value lies in native Windows support, cross-platform reliability fixes, Windows regression coverage and selective upstream maintenance, not in re-claiming upstream's design as its own original work.

## Multi-agent pipeline

This repo is one link in the [agent-relay](https://github.com/SanHsien/agent-relay) multi-agent dispatch and acceptance pipeline (role: Windows-native control plane, starting new sessions). agent-relay points at this repo through the `AGENT_RELAY_CORTEX` environment variable or the `cortex_path` setting, and imports `launcher.build_*_argv` to build worker commands. For the protocol specification and the directory of all related repos, see agent-relay.
