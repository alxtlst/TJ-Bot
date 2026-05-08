---
name: dor
description: Perform a Definition of Ready (DoR) check for an issue/ticket before implementation. Use when the user asks to assess readiness, clarify requirements, or prepare an issue for work (Definition of Ready, DoR, ready for dev, ticket grooming, refinement).
disable-model-invocation: true
---

# DoR — Definition of Ready check

## Goal

Given an issue/ticket (title + description + comments/context), produce a crisp readiness assessment:

- Decide **Ready** or **Not Ready**
- List **blocking gaps** (must-fill) vs **non-blocking suggestions**
- Provide a minimal set of **questions to ask** to make the issue ready
- Provide a **test plan** appropriate for the change

## Inputs

Collect and use whatever is available:

- Issue title and description
- Repro steps / current behavior / expected behavior
- Acceptance criteria, screenshots, logs, links to designs/specs
- Constraints (performance, security, compatibility, rollout)
- Dependencies (services, migrations, feature flags)

If information is missing, do not guess—surface it as a gap and ask targeted questions.

## Readiness checklist (evaluate all that apply)

### 1) Problem & context

- [ ] The problem is stated in user/business terms (not just a proposed solution).
- [ ] Current behavior is described (what happens today).
- [ ] Expected behavior is described (what should happen).
- [ ] Scope boundaries exist (in-scope and explicitly out-of-scope).

### 2) Acceptance criteria

- [ ] Clear, testable acceptance criteria exist (prefer bullet list).
- [ ] Edge cases are covered (errors, empty states, permissions, concurrency as relevant).
- [ ] Non-functional requirements captured if relevant (performance, accessibility, reliability).

### 3) Repro / evidence (bugfixes)

If the issue is a bug:

- [ ] Repro steps are provided or a minimal reproducer is available.
- [ ] Expected vs actual results are unambiguous.
- [ ] Environment info is present if relevant (versions, OS, config, feature flags).
- [ ] Logs/screenshots/traces are attached when they materially reduce ambiguity.

### 4) Solution constraints & architecture impact

- [ ] Constraints are explicit (APIs to use/avoid, data sources, allowed deps, coding standards).
- [ ] Data model impact is known (schema changes, migrations, backfills).
- [ ] Backwards compatibility expectations are stated.
- [ ] Rollout/feature-flag expectations are stated when risk warrants it.

### 5) Dependencies & sequencing

- [ ] External dependencies are identified (other teams/services, credentials, infra).
- [ ] Internal dependencies are identified (modules, services, shared libs).
- [ ] Ordering is clear for multi-step work (migrations before code, backend before UI, etc.).

### 6) Verification

- [ ] A test plan exists (what will be tested and how).
- [ ] Observability requirements exist when relevant (logs/metrics/alerts, dashboards, SLO impact).
- [ ] Definition of done is consistent with the repo norms (lint/tests/docs/PR expectations).

## Decision rule

Mark **Not Ready** if any of the following are true:

- The expected outcome cannot be stated in a testable way.
- The scope is ambiguous enough to risk building the wrong thing.
- Critical dependencies are unknown (blocked-by info missing).
- For bugs: there is no viable way to reproduce or validate the fix.

Otherwise, mark **Ready**, even if there are optional improvements.

## Output format (always use this)

Produce a single markdown report using this template:

```markdown
## Definition of Ready (DoR) check

**Issue**: <title or identifier>
**Decision**: ✅ Ready / ❌ Not Ready

### Summary
- <1–3 bullets on what the issue is asking for>

### Blocking gaps (must resolve before implementation)
- [ ] <gap 1>
- [ ] <gap 2>

### Non-blocking suggestions (nice to have)
- <suggestion 1>
- <suggestion 2>

### Questions to make this ready
1. <question 1>
2. <question 2>

### Acceptance criteria (proposed or confirmed)
- [ ] <AC 1>
- [ ] <AC 2>

### Test plan
- <unit/integration/e2e/manual steps as appropriate>

### Risks & rollout
- **Risks**: <brief>
- **Rollout**: <flag/migration/monitoring notes if any>
```

## Guidance on writing gaps/questions

- Prefer **few, high-leverage** questions over exhaustive interrogations.
- Make questions answerable with concrete outputs (examples, screenshots, API responses, exact copy).
- When proposing acceptance criteria, label them as proposed unless the issue already states them.
