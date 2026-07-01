---
title: "ADR-0001: Record architecture decisions"
tags: [adr]
status: accepted
date: 2026-07-01
---

# ADR-0001: Record architecture decisions

## Status

Accepted

## Context

This is a solo-built product intended to grow and scale. Decisions made early (which
models, which licenses, which architecture) will be questioned later — by future-me, by
collaborators, or by an auditor. Without a written record, the *reasons* behind a choice
are lost, and the same debates get re-litigated.

## Decision

We will keep **Architecture Decision Records (ADRs)** — one short Markdown file per
significant decision — in `adr/`, using a lightweight Nygard/MADR-style template:
*Status, Context, Decision, Consequences*. ADRs are numbered sequentially and are
**immutable**: a reversed decision is recorded as a new ADR that supersedes the old one.

## Consequences

- Good: the "why" behind the system is always discoverable; onboarding (incl. future-me) is faster; decisions are deliberate, not accidental.
- Good: pairs with the "Definition of Done = documented" gate — a hard-to-reverse choice triggers an ADR.
- Cost: a few minutes per significant decision. We mitigate by writing ADRs *only* for choices that are expensive to reverse, not every tweak.
