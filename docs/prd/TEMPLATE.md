---
id: NNN
title: <short feature name>
status: draft
owner: <email>
created: YYYY-MM-DD
updated: YYYY-MM-DD
related: []
grilled: false        # set true once the PRD has passed the /grill-me adversarial pass
---

<!--
This structure mirrors the `spec-writer` skill's PRD template so the skill's
output drops in without reshaping. Omit a section only if it is clearly
irrelevant — err on the side of including it.
-->

# <Feature name>

## 1. Summary

2–3 sentences: **what** we are building and **why**. Name the target user
segment and the primary outcome.

## 2. Problem Statement

Current pain or gap. Who experiences it, in what context. Evidence — metrics,
anecdotes, research, support tickets — where available.

## 3. Goals and Non-Goals

**Goals** — numbered, outcome-shaped, measurable where possible.

1. …

**Non-Goals** — explicitly out of scope. This section prevents the most
expensive arguments later.

- …

## 4. User Stories

A long, numbered list covering core flows, edge cases, permission variations,
and failure scenarios.

1. As a `<actor>`, I want `<capability>`, so that `<benefit>`.
   - Acceptance notes inline where helpful.

## 5. Functional Requirements

Expected behavior per flow: system states and transitions, how actors interact,
validation rules, error handling, retries, rate limits, correctness constraints.

## 6. Non-Functional Requirements

Performance, scalability, latency. Reliability, availability, degradation
behavior. Security, privacy, compliance.

## 7. Implementation Notes and Architecture

High-level architecture in text form. Modules/services to create or change and
how they interact. Ties back to concrete code — classes, endpoints, tables.

## 8. Out of Scope

Features, flows, user types, or edge cases explicitly **not** addressed here.
Note likely follow-ups.

## 9. Risks and Open Questions

- **Risks:** technical, product, UX, organizational.
- **Open questions:** must be resolved before implementation — *blocking /
  non-blocking* — owner.
- **Fragile assumptions:** the ones that would hurt most if wrong.

## 10. Success Metrics

How we will know this worked — input and output metrics. Directional
expectations are fine when precise numbers are not yet available.
