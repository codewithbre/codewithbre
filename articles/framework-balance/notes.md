---
title: Framework Balance — Model Intelligence vs. System Constraints
date: 2026-02-24
context: Observed during first interaction with the .claude setup
---

# Framework Balance: Model Intelligence vs. System Constraints

A running log of observations on where the CLAUDE.md system adds leverage, where it over-specifies, and where gaps exist.

---

## What the System Does Well

**Task integrity protocol is high-leverage.**
The rules around what constitutes a valid task — single functional change, clear intent, clear location, clear output — give the model a vocabulary for rejecting or deferring ambiguous work. Without this, that behavior is model-dependent and inconsistent. With it, it becomes deterministic and testable.

**Ambiguity resolution and verification-before-execution are forcing functions.**
These sections change model behavior at the moment it matters most: before execution begins. Surfacing uncertainty rather than guessing is where most AI-assisted engineering breaks down. These rules address that directly.

**Stopping behavior is explicitly encoded.**
Most systems tell the model what to do. This one tells it when to stop. That asymmetry is the right design choice for an AI executor — the risks are on the side of doing too much, not too little.

---

## Where the Balance Is Off

**Some rules over-specify reliable model behavior.**
Rules like "do not hardcode values" and "do not use `any`" are low-leverage. A model that would do these things won't be stopped by a rule. A model that wouldn't do them doesn't need the reminder. These are fine to have, but they dilute the weight of the high-leverage rules.

**The high-leverage rules are task scope and stopping behavior.**
These are the rules that actually change outcomes in practice:
- Task Integrity
- Ambiguity Resolution
- Error Remediation
- Verification Before Execution

These deserve the most prominence. Consider whether the output quality rules could be condensed to make the stopping/scoping rules stand out more.

---

## Identified Gaps

**No explicit rule for multi-task prompts.**
The system has no rule that says: *when a prompt contains multiple tasks, identify them, separate them, and execute only the first valid one.* In the first interaction, the model handled a two-task prompt correctly — but by inference, not because a rule encoded that behavior.

This is the highest-priority gap given the stated goal of building a system that detects tasks within prompts. A rule like the following would make this explicit:

> When a prompt contains more than one distinct task, identify each task, state them separately, and confirm which to execute first. Do not begin any task until the task boundary is agreed upon.

---

## Open Questions

- Should multi-task detection be a CLAUDE.md rule, a dedicated `/breakdown` behavior, or both?
- Is there a way to make the high-leverage rules visually distinct in CLAUDE.md (e.g., a priority marker) without restructuring the whole document?
- As the system matures, which rules have been violated most often in practice? That data should drive which rules get more specificity.

---

*This document is a living log. Add entries as new observations emerge.*
