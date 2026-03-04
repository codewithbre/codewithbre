---
title: Software Design Patterns for AI Skills
subtitle: What a Decade of OOP Teaches Us About Designing LLM Workflows
date: 2026-03-03
status: draft — expand with examples and probabilistic section
---

# Software Design Patterns for AI Skills

*What a Decade of OOP Teaches Us About Designing LLM Workflows*

---

There's a growing body of writing on AI orchestration — how to chain agents,
how to build pipelines, how to coordinate parallel workstreams across multiple
models. That conversation is worth having. But it mostly operates at the
architectural level, above the unit of work itself.

This article operates one layer down.

Before you can orchestrate skills, you have to design them. And the principles
that make a skill worth composing are the same principles that made object-oriented
software worth composing thirty years ago. Single responsibility. Interface
segregation. Strategy over hardcoding. Composition over monolithic inheritance.

The patterns aren't new. The application is.

---

## What Is a Skill, Precisely

The word is overloaded. In LangChain it means a tool. In Semantic Kernel it means
a plugin. In AutoGen it means something closer to an agent behavior. In most
custom frameworks it means whatever the author needed it to mean that week.

For this article, a skill is: **a named, self-contained unit of work with a
defined input, a defined output, and instructions that tell an LLM how to
produce one from the other.**

A skill is not a prompt. A prompt is what you say to a model. A skill is an
encoded workflow — a protocol the model follows when it receives a certain kind
of task. The distinction matters because it determines what a skill is responsible
for and where it ends.

A skill that's too broad isn't a skill. It's a vague instruction. A skill that's
too narrow isn't worth the overhead of defining it. The design patterns in this
article are, in large part, tools for finding the right boundary.

---

## Pattern 1: Single Responsibility

*A skill does one thing. One input type. One output type. One workflow.*

In OOP, the single responsibility principle says a class should have one reason
to change. The equivalent for skills: a skill should have one reason to be
invoked.

A skill that handles bug fixes *and* feature implementation *and* refactoring
is not a skill. It's three skills that haven't been separated yet. When it
works, you don't know which part is working. When it fails, you don't know which
part failed. And when requirements change for one of those task types, you have
to modify a skill that's doing unrelated work.

**What this looks like in practice:**

A `fix-bug` skill and an `implement` skill may look similar on the surface —
both involve writing code, both produce a PR-ready summary. But they have
fundamentally different starting points. A bug fix starts from a broken state
and works backward to the cause. An implementation starts from a blank state
and works forward to a spec. Same output format. Completely different reasoning
process. They should be separate skills.

The test: can you describe the skill in one sentence without using "and"? If
not, it has more than one responsibility.

**Where this gets harder with LLMs:**

Unlike a class, a skill's behavior is not deterministic. A single-responsibility
skill can still produce variable output depending on model, temperature, and
context window state. Single responsibility doesn't guarantee reliability — but
violating it guarantees that when something goes wrong, you can't isolate why.

*[Expand: example of a skill that was too broad, what happened, how splitting it fixed it]*

---

## Pattern 2: The Strategy Pattern

*The router is a strategy selector. Skills are the strategies.*

The strategy pattern defines a family of algorithms, encapsulates each one, and
makes them interchangeable behind a common interface. The client selects a
strategy at runtime without knowing the implementation details.

This is exactly what a workflow router does.

The router is the context object. The skills are the concrete strategies. The
task type is the selection criterion. When a prompt arrives, the router
evaluates the task type and selects the appropriate skill to load. The model
doesn't need to know about every skill in the catalog — it only needs the one
that applies.

**Why this matters:**

Without the strategy pattern, the model either guesses which workflow to follow
(inconsistent) or you spell out every possible workflow in one place (unmaintainable).
With it, each skill is isolated, independently testable, and swappable. You can
replace a skill without touching the router. You can add a new skill without
modifying any existing one.

**The fallback is not optional:**

Every strategy pattern needs a default. In the router, the fallback is explicit:
when no route matches, engage directly without loading a skill. This is not a
failure state — it's a valid strategy for novel work that doesn't fit a known
pattern. The mistake is treating "no route matched" as an error. Sometimes the
right answer is no protocol.

*[Expand: the over-routing failure mode — when you add too many strategies and
the selector starts forcing work into the wrong one]*

---

## Pattern 3: Interface Segregation

*Don't ask a skill to do what it wasn't designed for.*

Interface segregation says clients should not be forced to depend on interfaces
they don't use. For skills: a skill should only receive the context it needs
to do its job. Nothing more.

This is a loading problem as much as a design problem. Every skill loads into a
context window. Everything in that context window influences the model's output.
A skill for trivial corrections loaded with the full architectural history of a
codebase is going to behave differently than the same skill with minimal context.
And not necessarily better.

**The design implication:**

Each skill should specify, implicitly or explicitly, what context it needs to
function. A `quick-fix` skill — rename, typo, import path — needs the target
file and the error. It does not need the global engineering principles, the PR
template, the security review checklist. Loading those alongside it doesn't
help. It adds noise.

**Where this collides with how most systems work:**

Most AI tooling loads everything by default. Global context, project context,
conversation history, whatever the framework decides to include. Interface
segregation pushes in the opposite direction: load the minimum viable context
for this skill, and trust the skill to ask for more if it needs it.

*[Expand: practical example of context bloat degrading skill reliability, how
to think about what to include vs. exclude per skill type]*

---

## Pattern 4: Composition over Inheritance

*Build skill chains, not skill hierarchies.*

Inheritance in OOP creates a is-a relationship. A `SeniorEngineer` extends
`Engineer` which extends `Employee`. Behavior is accumulated through the
hierarchy. The problem is that inheritance is rigid — change a parent class and
you change every child.

Composition builds behavior by assembling pieces. A `SeniorEngineer` has a
`CodeReviewer`, a `MentorshipCapability`, and a `SystemDesignSkill`. You get
the same flexibility with none of the coupling.

Skills compose the same way.

A full implementation workflow is not one skill. It's a chain: `breakdown` hands
off to `implement`, which invokes `code-reviewer`, which references
`problem-solving` if something breaks. Each skill in the chain has one
responsibility. Each knows when to hand off to the next. None of them are aware
of the full pipeline — they only know their own job and where to pass the work.

**Why this matters for maintainability:**

If the review criteria change, you update `code-reviewer`. You don't touch
`implement` or `breakdown`. The chain still works because the interface between
skills — the handoff point — hasn't changed.

If you'd built one monolithic `implement-everything` skill instead, updating the
review criteria means editing a skill that also handles scoping, execution,
and summarization. Every change is higher risk.

*[Expand: a concrete chain from your own setup — plan-work → implement-ticket →
code-reviewer — walking through what each skill hands off and what it expects to
receive]*

---

## Pattern 5: The Command Pattern (and Where It Breaks)

*Encapsulate a request as an object. Parameterize. Queue. Audit.*

The command pattern wraps a request — with its target, its parameters, and its
execution logic — into a self-contained object. This enables queuing, logging,
undo, and replay. The invoker doesn't need to know how the command works.
It just calls execute.

Skills are a natural implementation of this pattern. A skill encapsulates a
request type (fix this bug), its parameters ($ARGUMENTS), and its execution
protocol (analyze → restate → fix → summarize). The invoker — you, or a
pipeline — just calls the skill. The skill knows how to handle it.

**The audit benefit:**

Because the command is self-contained, you can log every invocation. You know
what skill ran, what it received, what it produced, and in what sequence.
In a multi-agent pipeline this is not a convenience — it's how you debug what
went wrong and verify that what should have run did run.

**Where it breaks with LLMs:**

Classic command pattern assumes deterministic execution. Run the same command
with the same inputs, get the same output. LLMs don't work that way. The same
skill, the same task, the same inputs can produce meaningfully different outputs
across runs. This breaks the undo/replay assumption entirely.

This is where the pattern requires adaptation. Instead of relying on replay,
you rely on scope constraints. Instead of undo, you rely on the model restating
what it will do before doing it — a human checkpoint that replaces the
reversibility you'd normally get from a command log. The command pattern gives
you auditability. The verification-before-execution protocol gives you
the recoverability that determinism would have given you for free.

*[Expand: concrete example of the human checkpoint replacing the undo mechanism —
what the restatement looks like, why it works, where it fails]*

---

## The Underlying Shift: From Determinism to Contract

Software design patterns were built on a foundation of deterministic execution.
Call a method with these inputs, get this output. Every time. The patterns
are designed around that guarantee.

LLM skills don't have that guarantee. Same inputs, different outputs.
The same skill that worked yesterday may produce subtly different output today
with the same prompt. This isn't a bug — it's the nature of probabilistic
systems.

So the patterns still apply, but the way you validate them has to change.

You can't unit test a skill the way you unit test a function. You can't assert
equality on the output. What you can do is test the shape of the output: does it
contain the expected sections? Does it stay within the stated scope? Does it
surface ambiguity rather than resolve it silently? These are contract tests, not
unit tests — and they're the appropriate tool for validating probabilistic behavior.

The design patterns give you the structure. Contract testing gives you the
confidence. Together, they're the equivalent of what type safety and unit testing
give you in deterministic software: not a guarantee that the output is correct,
but a guarantee that it's the right shape and within the right boundaries.

*[Expand: what a contract test for a skill actually looks like — what to assert,
what to skip, how to handle the inherent variability]*

---

## Closing

The insight isn't that OOP solves AI engineering. It's that the problems are
recognizable. Unclear boundaries between responsibilities. Brittle monolithic
workflows that break when one thing changes. Context that bleeds across units
that should be isolated. Chains that can't be reasoned about because nothing
has a single job.

These problems were solved in software by the same patterns that apply here.
Not because the patterns are universal laws, but because the underlying failure
modes are universal. Complexity without structure produces the same failure
modes regardless of whether the executor is a CPU or a language model.

The difference is that software fails loudly. A type error doesn't compile.
A null reference throws. LLMs fail quietly — producing plausible-looking output
that does the wrong thing. The patterns don't just make skills easier to
maintain. They make failures visible.

That's why they're worth applying.

---

*Draft — 2026-03-03.*
*Sections marked [Expand] are placeholders for your own examples and experience.*
*The probabilistic/contract testing section is the differentiator — go deepest there.*
