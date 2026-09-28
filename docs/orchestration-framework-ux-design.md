# Rook Orchestration Framework UX Design

**Status:** Proposed<br>
**Date:** 2026-09-27<br>
**Scope:** Phase 1 operator console<br>
**Implementation:** ASP.NET Core Razor Pages with progressive enhancement<br>
**Conformance target:** WCAG 2.2 Level AA

## 1. Purpose and authority

This document is the canonical interaction and accessibility specification for
the Phase 1 Rook operator console. The
[solution architecture](./orchestration-framework-solution-architecture.md)
owns system boundaries, authorization enforcement, data, and deployment. The
[implementation tasks](./orchestration-framework-tasks.md) own delivery order
and verification work.

The prototypes below are content and interaction contracts, not
pixel-perfect mock-ups. They use the current B.C. Design System tokens and
released component guidance reviewed on 2026-09-27. Exact package versions and
released components must be rechecked and pinned during implementation.
Project-specific tables, timelines, and status summaries must not be described
as official B.C. Design System components.

## 2. UX principles

1. Use plain, task-oriented language and expose the next safe action.
2. Distinguish current, stale, unavailable, pending, blocked, failed,
   cancelled, no-change, and completed states.
3. Show source and freshness for external observations.
4. Never present stale, missing, or integrity-failed evidence as success.
5. Use native HTML controls and progressive enhancement.
6. Do not rely on colour, icons, motion, or position alone.
7. Keep navigation, forms, validation, and status understandable without
   client-side JavaScript.
8. Enforce authorization on the server; control visibility is only a UX aid.

## 3. Information architecture and shell

```text
Skip to main content
+--------------------------------------------------------------------------+
| B.C. government | Rook                                      [Account menu] |
+--------------------------------------------------------------------------+
| Portfolio | Repositories | Runs | Administration*                         |
+--------------------------------------------------------------------------+
| Breadcrumbs                                                               |
| <one page H1>                                      [contextual action]    |
| Introductory status or alert, only when relevant                           |
|                                                                          |
| Main page content                                                         |
+--------------------------------------------------------------------------+
| Privacy | Accessibility | Contact/Support | Build/version                 |
+--------------------------------------------------------------------------+
* Administration is visible and authorized only for Administrators.
```

The header, footer, buttons, fields, tags, alerts, and dialogs follow released
B.C. guidance. Native links navigate; native buttons act. The DOM and focus
order match the visual order. The shell works without client-side JavaScript.

## 4. Role and control matrix

| Capability | Viewer | Operator | Maintainer | Administrator |
| --- | --- | --- | --- | --- |
| View authorized portfolio, repository, run, and evidence metadata | Yes | Yes | Yes | Yes |
| Download authorized evidence permitted by evidence classification and subject authorization | Internal only | Internal and Confidential by repository policy | Internal/Confidential; Restricted with approval and step-up | Any class; Restricted with approval and step-up |
| Refresh provider status; start/cancel assessment | No | Yes | Yes | Yes |
| Create/release repository hold; set expiring priority override | No | Yes | Yes | Yes |
| Onboard repository | No | No | Yes | Yes |
| Pause/resume the fleet | No | No | No | Yes |

Disabled controls are used only when explaining a temporarily unavailable
action helps the operator. A control the subject can never use is omitted.
Server-side authorization applies in every case.

Repository offboarding, provider-binding edits, policy editing, and
role-mapping administration are deferred beyond Phase 1. The pilot consumes
externally provisioned OIDC claim/group mappings and the immutable
binding/policy configuration confirmed during onboarding; the operator console
does not imply that those values are editable.

## 5. Screen A - Portfolio

```text
H1 Portfolio
[System alert: Assessments paused. Existing read-only data remains available.]

Summary
[Eligible 18] [Needs attention 4] [Active runs 2] [Stale providers 3]

Filters
[Search repositories________________] [Status v] [Eligibility v] [Apply]

Repositories
| Repository (link) | Eligibility | Latest pipeline | Active run | Updated |
| api-catalogue     | Eligible    | Passed          | Assessing  | 2 min   |
| permits-ui        | On hold     | Unavailable     | --         | 18 min  |

[Pagination with current page announced]
```

### 5.1 Behavior

- Summary counts never rely on colour alone.
- Filters use a GET form so filtered results have a stable URL.
- At 320 CSS pixels, each row becomes a labelled definition/card presentation
  without changing reading order.
- At wider sizes the semantic table retains a caption. Sort buttons announce
  the active direction.
- Empty, loading, partial, stale, unavailable, unauthorized, and error states
  include plain-language text and a permitted next action.

## 6. Screen B - Repository onboarding and detail

```text
H1 Add repository
Error summary (only after invalid submit; focus moves here)

Repository
GitHub organization [________________]
Repository name     [________________]
Default branch      [________________]  (read-only after successful discovery)

Pipeline observations
[x] Observe GitHub Actions

Workflow eligibility
[x] Read-only framework assessment

[Review repository] [Cancel]

-- after successful review --
H1 Review repository
Provider identity, default branch, archived/fork state, toolchain facts
Pipeline bindings and latest known state
Eligibility: Eligible / Blocked with reasons
Policy and package versions to use
[Confirm onboarding] [Back and edit]
```

### 6.1 Behavior

- Discovery does not create a repository profile.
- Confirmation shows exactly what Rook will read.
- Confirmation states that Phase 1 will not commit, push, comment, label,
  dispatch workflows, or create pull requests.
- Existing repository detail uses ordinary page sections for Overview,
  Provider status, Policy, Runs, Holds, and Audit.
- An Operator can request a read-only provider refresh from Provider status.
  The action uses POST-redirect-GET, joins an equivalent active refresh, and
  keeps the last observation labelled stale or unavailable until a newer
  observation is durably recorded.
- If tabs are later justified, they remain ordinary links to distinct URLs.

## 7. Screen C - Start assessment

```text
H1 Start read-only assessment
Repository: api-catalogue

Checks before start
- Repository is eligible
- No active hold
- GitHub and GitHub Actions observations are current
- Copilot credential profile: Available (expires 16:40)
- Workflow/Crow/Raven versions and digests

This assessment reads repository and pipeline information. It cannot modify
the repository or trigger a pipeline.

[Start assessment] [Cancel]
```

### 7.1 Behavior

- The POST uses an idempotency key.
- If an equivalent run is active, redirect to it with an informational alert.
- A failed check identifies the reason, provenance or freshness, owner, and
  available release or retry action.
- Starting an assessment uses POST-redirect-GET. Refreshing the destination
  page cannot repeat the command.

## 8. Screen D - Run detail and evidence

```text
H1 Assessment run R-2026-0042                     [Cancel run]
Status: Assessing
Repository: api-catalogue @ 84a2...   Started by: A. Operator
Started: 16:04   Last update: 16:06   Correlation: ...

Timeline
[Completed] Preflight     16:04  View evidence
[Completed] Observe       16:05  View evidence
[In progress] Assess      16:06  Last heartbeat 20 sec ago
[Not started] Plan

Configuration
Workflow / policy / model / Crow / Raven / credential-profile references

Evidence
| Kind | Source | Created | Size | Integrity | Action |
| ...  | ...    | ...     | ...  | Verified  | Download |

Audit history
Timestamp | Actor/workload | Action | Outcome/reason
```

### 8.1 Behavior

- Ordinary page refresh is the baseline.
- Optional polling updates one concise `role="status"` region only when the
  user-relevant status or active step changes.
- Polling stops in a terminal state and pauses while the page is hidden.
- Polling respects reduced-data and reduced-motion preferences.
- Updates do not move focus or repeatedly announce the timeline.
- Evidence integrity is `Verified`, `Mismatch`, or `Not checked`, never an
  icon alone.
- A mismatch blocks download, identifies the integrity failure, and offers
  reassessment or support instead of presenting the evidence as usable.
- Evidence classification is displayed with every item. A denied download
  states whether the subject lacks the required class authorization, the item
  is restricted, publication scanning is incomplete, or integrity validation
  failed; the page never renders restricted evidence as active content.

## 9. Screen E - Holds, overrides, and fleet pause

```text
H1 Place repository on hold
Repository: permits-ui
Reason [________________________________________________________]
Release condition [_____________________________________________]
Expiry (optional) [yyyy-mm-dd hh:mm]
[Place hold] [Cancel]

H1 Set manual priority
Priority (required) ( ) P0 ( ) P1 ( ) P2 ( ) P3 ( ) P4
Rank (optional) [____]
Reason [________________________________________________________]
Expires [yyyy-mm-dd hh:mm]
[Set priority] [Cancel]
```

### 9.1 Behavior

- A review page displays scope and effect before a destructive or fleet-wide
  action.
- Releasing a hold and pausing or resuming the fleet require a reason.
- Cancellation remains pending until the worker and OpenShift Job reconcile a
  terminal outcome.
- Use dedicated confirmation pages unless a tested, B.C.-aligned dialog
  provides a clear benefit.

## 10. Reachable states

| State | Visible behavior and next action |
| --- | --- |
| Loading or pending | Name what is loading, preserve navigation, prevent duplicate submission, and provide a manual refresh path |
| Empty | Explain why no records are present and show the permitted primary action |
| Assessment complete | Show `AssessmentComplete`, summarize one or more findings, link typed evidence, and state that Phase 1 made no repository change |
| No change | Show `NoChange`, explain that no actionable findings were recorded, and link the completed assessment evidence |
| Other success | Confirm the completed operation and link to its durable result |
| Validation error | Focus an error summary linked to fields, retain safe input, and put specific errors beside fields |
| Unauthorized | Explain that access is unavailable and provide the approved support path without revealing resource existence |
| Blocked or held | Show reason code, owner, release condition or expiry, and permitted next action |
| Provider unavailable | Identify the provider, last successful observation, safe actions, and retry path; do not present stale data as current |
| Stale | Show source timestamp and freshness threshold; permit refresh when authorized |
| Cancellation pending | State that work may still be stopping; do not report `Cancelled` until reconciled |
| Failure | Give a stable support/reference code, retained progress, retry eligibility, and non-sensitive reason |

## 11. Accessibility acceptance

All screens must meet WCAG 2.2 Level AA:

- semantic landmarks, one H1, logical headings, lists, tables, links, buttons,
  labels, and error relationships;
- keyboard operation without traps;
- visible, sufficiently contrasted, and unobscured focus;
- no information conveyed through colour, icon, position, sound, or motion
  alone;
- WCAG 2.2 target-size minimums;
- 200% text sizing, 400% zoom, and 320 CSS-pixel reflow;
- forced-colours support and reduced-motion behavior;
- specific field errors plus a focused, linked error summary after invalid
  submission;
- polite, atomic status updates only when user-relevant state changes;
- no focus movement for background status refresh;
- persistent labels and instructions before controls;
- accessible names, roles, values, descriptions, and state announcements.

Automated checks supplement, not replace, keyboard and screen-reader smoke
tests. Record browser, viewport/zoom, assistive technology, and tooling
versions for the pilot review.

## 12. Unicode and content acceptance

Unicode test fixtures include Indigenous-language characters, decomposed
combining sequences, syllabics, and a supplementary-plane character through:

- form input and validation;
- PostgreSQL storage and retrieval;
- search and sorting where applicable;
- provider/API transit;
- display and evidence metadata;
- export/import;
- logs where retaining display text is intentional.

Preserve supplied spelling and code points. Protocol identifiers and security
decisions use ordinal comparison. Do not silently transliterate, strip
diacritics, replace malformed input, or treat display text as an identifier.

## 13. Update checklist

When changing this UX:

1. update the affected screen, states, role visibility, and accessibility
   acceptance in this document;
2. update authorization/data contracts in the
   [solution architecture](./orchestration-framework-solution-architecture.md)
   when behavior crosses those boundaries;
3. add or revise executable work in the
   [implementation tasks](./orchestration-framework-tasks.md);
4. recheck the current B.C. Design System release status and WCAG guidance;
5. preserve no-JavaScript, keyboard, zoom/reflow, Unicode, stale/unavailable,
   and failure behavior.
