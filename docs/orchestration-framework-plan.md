# Rook Orchestration Framework Plan

**Status:** Proposed<br>
**Date:** 2026-09-27<br>
**Initial scale:** 25-100 repositories with continuous daily maintenance<br>
**Phase 1 autonomy:** Read-only assessment; no repository or pipeline mutation<br>
**Target initial autonomy (Phase 2):** Create draft pull requests; humans
approve and merge

## Document set

This file is the entry point for the Rook orchestration framework. Detailed
design and implementation content is maintained in three focused documents:

| Document | Canonical responsibility | Update when |
| --- | --- | --- |
| [Solution architecture](./orchestration-framework-solution-architecture.md) | System boundaries, components, workflows, data, security, hosting, reliability, testing strategy, risks, and architecture decisions | A technical boundary, quality attribute, provider contract, deployment choice, or security control changes |
| [UX design](./orchestration-framework-ux-design.md) | Operator information architecture, role-visible controls, screen prototypes, interaction states, content behavior, responsive behavior, and accessibility acceptance | An operator workflow, screen, state, control, or accessibility requirement changes |
| [Implementation tasks](./orchestration-framework-tasks.md) | Dependency-ordered Phase 0/1 checklist, independent tests, parallel work, exit gates, and later-phase roadmap | Work is added, completed, blocked, reordered, split, or its verification changes |

The three detailed documents are authoritative for their named responsibility.
Do not copy detailed content back into this overview. Cross-cutting changes
must update every affected document in the same pull request.

## Executive recommendation

Build Rook as a .NET 10 modular monolith with role-isolated deployments for:

1. the control-plane web/API;
2. trusted integration and publication workers;
3. an ephemeral agent runner with a credential broker; and
4. a separate credential-free verifier Job.

Use Microsoft Agent Framework for explicit agent composition while
deterministic C# owns state transitions, policy, retries, budgets, and
verification. Start with the GitHub Copilot SDK behind an application-owned
agent-backend interface. Pin and attest Crow and Raven packages; do not use
moving repository branches as runtime dependencies.

Host the MVP in OpenShift Emerald with a single in-cluster PostgreSQL instance.
Start with GitHub and GitHub Actions. Phase 1 is a manual, read-only pilot.
Phase 2 adds evidence-gated draft pull requests for human review.

## Goals

- Maintain repositories that have completed developer-led Crow modernization.
- Detect framework, dependency, and feasible security maintenance work.
- Produce attributable, evidence-backed draft pull requests after the
  read-only pilot proves the control plane and containment model.
- Support manual, scheduled, and event-triggered runs with durable state.
- Keep source-control and CI/CD provider bindings independent.
- Make workflow, agent, model, Crow, Raven, policy, and evidence versions
  reproducible.

## MVP non-goals

- Autonomous merge, deployment, production mutation, or CI/CD execution.
- Unattended maintenance of repositories that have not been onboarded.
- Treating model, agent, or MCP claims as verification.
- Cross-repository changes in one run.
- Long-term semantic memory or self-modifying agents and skills.
- Using JARVIS as the Rook execution-state store.

## Delivery phases

| Phase | Outcome | Gate |
| --- | --- | --- |
| Phase 0 | Disposable entitlement, permission, package, workflow-durability, and containment spikes | Approved go/no-go record with measured cost and bounded release conditions |
| Phase 1 | Accessible operator console and one manual read-only assessment against an approved GitHub pilot | Normalized GitHub Actions state, evidence, audit, backup/restore result, accessibility evidence, and proof of no provider mutation |
| Phase 2 | Isolated implementation, deterministic verification, independent review, and draft PR publication | At least 20 proposals without policy escape, duplicate PR, secret exposure, or unexplained verification |
| Phase 3 | Scored scheduling, security remediation, and additional provider adapters | Measured reviewer throughput and operations support expansion to 25 repositories |
| Phase 4 | Scale toward 100 repositories and add approved agent backends | Measured concurrency and canary evidence support expansion |

## Confirmed product decisions

- The Phase 1 UX covers portfolio status, repository onboarding/detail, run
  detail/evidence, holds, overrides, cancellation, and fleet pause.
- The operator console uses ASP.NET Core Razor Pages with progressive
  enhancement.
- Authorization uses `Viewer`, `Operator`, `Maintainer`, and `Administrator`
  roles plus repository- and operation-level policy.
- Phase 1 may read repository and GitHub Actions state but may not commit,
  push, comment, label, dispatch workflows, or create pull requests.

## Open implementation decisions

The detailed architecture tracks the complete decision list. The decisions
most likely to block the pilot are:

- approved workforce OIDC client, claim mapping, role owners, and outage
  behavior;
- approved GitHub pilot repository and read-only credential mechanism;
- Copilot unattended-use approval, entitlement owner, and credential lifecycle;
- OpenShift namespace, quota, egress, image-promotion, and recovery ownership;
- evidence storage, classification, and retention;
- PostgreSQL backup/restore ownership and accepted MVP RPO/RTO.

## Navigation

- Start a technical review with the
  [solution architecture](./orchestration-framework-solution-architecture.md).
- Review or implement operator screens from the
  [UX design](./orchestration-framework-ux-design.md).
- Execute work from the
  [implementation tasks](./orchestration-framework-tasks.md), one unchecked
  task at a time.
