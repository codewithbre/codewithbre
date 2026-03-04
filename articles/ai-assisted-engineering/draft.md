---
title: How I Set Up Claude Code to Actually Work
subtitle: A Developer's System for AI-Assisted Engineering
date: 2026-03-03
status: draft
---

# How I Set Up Claude Code to Actually Work

*A Developer's System for AI-Assisted Engineering*

---

Claude Code is powerful. Without guardrails, it's also unpredictable. This is
how I set it up to work within a system I control.

---

## The Problem with AI Out of the Box

The first time you use Claude Code — or any AI coding tool — it feels like magic.
You describe a task, the model writes code, and it mostly works. Then you ask it
something slightly less clear, and it refactors half your codebase while solving
a problem you didn't ask it to solve. Or it makes assumptions, fills in gaps with
reasonable-sounding logic, and hands you something that compiles but does the
wrong thing.

The issue isn't the model. The issue is that the model is doing exactly what you'd
expect a highly capable but completely unconstrained executor to do: it makes
decisions when decisions haven't been made for it.

I needed a system that changed that. Not one that hobbles the model's capability,
but one that channels it — so that when I give it a task, I can trust the output
because I trust the constraints it operated within.

---

## The Philosophy

Everything in my setup flows from one idea: **AI is the executor. I'm the driver.**

A capable model doesn't need to be told how to write code. It needs to be told
what to write, what not to touch, when to stop, and what done looks like.
The moment you hand over decision-making about scope, design, or what counts as
complete, you lose the ability to trust the output.

Think of it like a high-performance race car. Extraordinary capability. But
dangerous without discipline. You are the driver. Your configuration is the
circuit. Every task is a defined lap with a clear start, a specified route, and a
finish line you can verify.

A few other principles that shape the whole system:

**Small, scoped tasks with clear boundaries prevent hallucination and scope creep.**
The larger and vaguer the task, the more the model fills gaps with assumptions.
A task that can be stated in two sentences and verified in five minutes is
dramatically more reliable than one that spans files, systems, and intentions.

**Negative constraints over positive instructions.** Don't tell the model how to
work. Tell it what it must never do. Do not refactor unrelated code. Do not
install dependencies without discussion. Do not accept a task that has two valid
interpretations. Constraints are more durable than instructions because they
address failure modes, not expected paths.

**The stopping behavior is the most important behavior to encode.** Most systems
tell the model what to do. Mine also tells it when to stop. That asymmetry is
the right design for an AI executor — the risks are on the side of doing too
much, not too little.

---

## The Configuration Architecture

Claude Code's configuration system has three levels, and understanding how they
compose is the foundation of everything else.

**Global** (`~/.claude/`) holds your engineering principles. These don't change
between projects. They define what the model is never allowed to do and how it
reasons about every task before touching a line of code.

**Project** (`.claude/`) holds project-specific context: the architecture,
patterns, and conventions of this particular codebase. The model operating
contract for a specific repo.

**Local** (`.claude/settings.local.json`) holds personal preferences. Not shared.

Global rules define the baseline. Project rules override downward when the
project needs different behavior. Local settings are yours alone. Each level
inherits from the one above — never the other way around.

One compounding advantage of this setup: if you use both Claude Code and Cursor,
both tools read the same `.claude/` directory. Your commands, agents, and
principles are consistent across both interfaces. One system, two surfaces. You
can cross-validate a task through both tools using the same framework.

This matters more than it sounds. The philosophy in this article — AI as
executor, constraints over instructions, scope before execution — is not
specific to Claude Code. It applies equally in Cursor. The global `CLAUDE.md`
becomes the foundation for Cursor's always-applied rules. The commands become
Cursor skills. The agents become Cursor subagents. The workflow loop stays the
same regardless of which surface you're in.

In Cursor specifically, you can take this further with a workflow router: an
always-applied `.mdc` rule that maps task types to skills automatically. Instead
of manually invoking the right skill for each task, the rule tells the agent
which skill to load based on what it detects in the prompt. The same routing
table. The same fallback for work that doesn't fit a known pattern. The
difference is that in Cursor the router lives in a dedicated rule file; in
Claude Code it lives as a section in `CLAUDE.md`. Same concept, same behavior,
different location in the config hierarchy.

The practical upside: you build the system once. You refine it in one place.
And it applies everywhere you work.

---

## The Global CLAUDE.md

The global `CLAUDE.md` is the most important file in the system. It's what makes
behavior deterministic instead of model-dependent.

Without it, the model does what feels right. With it, the model does what you've
defined. Those are very different things.

The rules are organized around failure modes — the specific ways AI-assisted
development goes wrong in practice.

**Task Integrity** defines what a valid task looks like: a single functional
change, clear intent, clear location, clear output, completable and verifiable
in one pass. The model is required to reject tasks that don't meet this
definition. This is the most important rule in the file, and everything else
depends on it. A poorly scoped task produces a poorly scoped result regardless
of how good the other rules are.

**Ambiguity Resolution** is a forcing function. When a task has more than one
valid interpretation, the model doesn't guess. It states what's ambiguous, lists
the interpretations, and stops. This single rule eliminates an enormous category
of subtle errors — the ones where the output looks right but does the wrong thing
because the model silently chose one interpretation over another.

**Verification Before Execution** means nothing gets coded until the model has
restated what will change, which files are in scope, and what is explicitly out
of scope. You have a checkpoint before anything is written.

**Error Remediation** encodes the stopping behavior. When something unexpected
happens mid-task, the model stops, states the error, restates the original scope,
and proposes a path forward without taking action. It does not expand scope to
fix an error. It does not suppress the problem to maintain progress. It surfaces
the issue and waits.

The rest — output quality, git rules, completion protocol — rounds out the
contract. No lint suppression. No `any` types. No direct commits to main. Every
completed task ends with a PR-ready summary: what changed, which files, what was
verified, what the known limitations are.

---

## Commands

Commands are the interface between you and the model's behavior. Each one
encodes a workflow — a specific sequence of reasoning and action appropriate to
a type of task. Rather than writing the same instructions every time, you invoke
a command and the model knows the protocol.

- `/implement` — scoped implementation with a verification loop. Restate scope,
  confirm, execute, summarize.
- `/fix-bug` — locate the bug, restate scope, fix only that. Not the surrounding
  code.
- `/analyze` — understand the architecture before planning. Read-only output.
- `/breakdown` — decompose work into executable tasks. Human reviews before
  anything is executed.
- `/create-pr` — generate a PR summary with a scope compliance checklist.
- `/onboard-project` — scan a repo and produce a project-level CLAUDE.md draft
  for your review.
- `/quick-fix` — trivial corrections without the full verification loop.
  Rename, typo, import path. Self-enforcing scope gates.
- `/config-log` — document changes to the `.claude/` configuration itself.

The `/breakdown` command is the entry point for agentic work. Before any task
is executed, it runs through breakdown to produce a sequence of valid, scoped
subtasks. Each one is a clean handoff — to you, to another session, or to a
downstream agent in a pipeline.

---

## Agents: Read-Only by Design

Agents in this system are specialists with constrained access. Every agent is
restricted to reading files — no write access. An agent that cannot modify files
cannot break anything. It can only observe and report.

- `code-reviewer` — quality, consistency, adherence to project patterns.
- `security-reviewer` — vulnerabilities, injection risks, auth gaps, data exposure.
- `scope-auditor` — verifies that completed work stayed within CLAUDE.md
  boundaries. Runs post-execution.
- `dependency-analyst` — researches packages and presents facts. The human
  makes the decision.

The read-only constraint is not a limitation. It's what makes these agents safe
to run automatically, in parallel, or as post-execution checks in a pipeline.
A reviewer that can't write can't cause harm.

On model selection: I run heavier models (Opus, Sonnet) for planning, task
decomposition, and ambiguity resolution — where reasoning quality matters most.
Faster, cheaper models for execution once the task is well-defined. A
well-scoped task is a well-scoped task regardless of which model executes it.
This is how you scale throughput without scaling cost linearly.

---

## The Workflow Loop

Every task, regardless of size, runs through the same loop:

1. Decompose the work into valid tasks.
2. The model verifies each task against CLAUDE.md.
3. The model restates scope and waits for approval.
4. You approve.
5. The model executes.
6. The model reports completion with a PR-ready summary.
7. The scope auditor verifies adherence post-execution.

The loop is what makes the system auditable. You can inspect any step. You
know what was approved, what was executed, and what was verified. There's no
"I think it did the right thing" — there's a record.

---

## In Practice

The commands and agents aren't used in isolation. They map to situations.

**Starting fresh on a new codebase:** Run `/analyze` first. Understand what
you're working with before writing anything. Then run `/onboard-project` to
generate a project-level CLAUDE.md. Review the output before accepting it.
That becomes the model's operating contract for the project.

**Implementing a feature:** Never start with implementation. Start with
decomposition. Run `/breakdown` on the feature description, review the task
list, reject or refine anything too vague or too large. For each task, run
`/implement`. After each implementation, run the `code-reviewer` agent. When
all tasks are complete, run `/create-pr`.

**Fixing a bug:** Run `/fix-bug` with a clear description of the observed
behavior and where it occurs. The model analyzes, restates the scope, and
fixes only what is broken. Run `scope-auditor` post-fix to confirm nothing
outside the stated scope was changed.

**The model encountering ambiguity:** You send a prompt like "update the API
layer." The model doesn't begin. It surfaces the ambiguity: what does "update"
mean here? Remove unused endpoints? Standardize error handling? It lists the
interpretations, identifies what information is missing, and stops. Only after
the scope is defined does it produce a scope statement and begin. This is the
system working as designed — and it catches more errors than any linter.

---

## The Limits of the System

One thing I've learned building and refining this setup: **the system only
works for work you already understand.**

The commands and workflows exist because certain kinds of work have happened
enough times that a pattern emerged. That pattern became a protocol. The protocol
got a command. When you encounter that kind of work again, the system handles
it reliably.

Novel work is different. When you're designing something you've never built,
making a technical decision for the first time, or exploring an unfamiliar
domain — there is no protocol for that. Trying to force it into the nearest
existing workflow produces output that follows the process but misses the work.

The right response is not to add more commands. It's to recognize the
distinction and let the model engage directly when no protocol applies.
The system's value is in the predictable, repeatable work — the 80%. Not the
exploratory, creative work that defines itself as you do it. Confusing the two
makes both worse.

---

## Closing

This isn't about constraining AI. It's about designing a system where the AI
can do its best work within boundaries you trust.

The model is not the unreliable part. The unreliable part is the handoff —
the moment between your intent and the model's execution. A well-configured
system makes that handoff explicit, structured, and reversible. The model
restates what it heard. You confirm or correct. Only then does it act.

When the system is working, you don't wonder what the model did or hope it
stayed in scope. You know, because the protocol requires it to tell you.

The setup takes time. The payoff is every task after that.

---

*Draft — 2026-03-03. For publication when ready.*
