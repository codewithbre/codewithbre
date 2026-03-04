---
title: The Router Limit — Skills Work Until the Work Changes
date: 2026-03-03
context: Realized while porting the Cursor workflow-router pattern to Claude Code
status: article seed
---

# The Router Limit: Skills Work Until the Work Changes

## The Insight

Building a router — a rule that maps task types to skills — is one of the
highest-leverage things you can do with an AI coding setup. Instead of manually
choosing the right workflow for each task, the model reads the task and routes
itself. The right protocol loads automatically. You stop thinking about the
process and start thinking about the work.

But while building this, something became clear: **the router only works for
work you've already seen.**

A router is a map of known territory. You can only route to destinations you've
already defined. `implement`, `fix-bug`, `quick-fix`, `review`, `breakdown` —
these commands exist because that work has happened enough times that a pattern
emerged. The pattern was distilled into a workflow. The workflow was given a route.

Novel work has no route. When you're figuring something out for the first time —
designing a system you've never built, making a decision you haven't made before,
exploring a domain you don't yet understand — there is no skill for that. There
is no pre-defined process. There is just the work.

---

## The Tension

There is a real tension between the two modes:

**System mode:** You know what kind of work this is. You have a protocol. The
model reads the task, routes to the right skill, follows the workflow, produces
the output. Everything is predictable, auditable, repeatable.

**Creative mode:** You don't know what this is yet. You're exploring. The work
is defining itself as you do it. No skill applies. The model needs to think
alongside you, not execute a protocol at you.

The instinct — especially after building a router — is to add more routes.
Maybe if you define enough skills, every type of work will have a home.

That instinct is wrong.

More routes don't solve the problem. They make it worse. An over-routed system
tries to classify everything. When it can't, it forces work into the nearest
category. An exploration that isn't quite a breakdown gets `/breakdown` called
on it prematurely. A design decision that isn't quite a bug gets routed to
`/fix-bug`. The model follows the protocol for a task the protocol wasn't
designed for. The output looks like work but isn't.

The right answer is the opposite: **a router with a clear fallback**. Route
confidently when the work is known. Don't route when it isn't. Make the fallback
explicit — "when no route matches, engage directly."

---

## What This Means for How AI Tooling Gets Built

Most AI tooling optimization moves in one direction: more structure, more
process, more defined workflows. The assumption is that predictability is always
better.

That is true for repeatable work. It is false for creative work.

The trap is that the skills and workflows you build are evidence of work you've
done before. They accumulate. And as they accumulate, they become a kind of
bureaucracy — every new task gets evaluated against the existing process catalog,
and anything that doesn't fit gets squeezed into the nearest thing that does.

The better model: **a lightweight, honest router**. Fewer routes, each
well-defined. A clear fallback for everything else. The router's job is not to
classify everything — it's to identify the work that genuinely benefits from
structure, and get out of the way of everything else.

The system's value is in the predictable 80%. Not the unpredictable 20%.
And confusing the two is how you make the 80% worse while failing the 20%.

---

## Open Questions for the Article

- Is "router" even the right metaphor? A router implies fixed routes. What you
  actually want is more like a dispatcher with a triage function — skilled at
  deciding when to dispatch and when to handle it directly.

- Does the same tension exist in human project management? Sprint planning,
  ticket systems, process definitions — do they suffer from the same
  over-classification problem? What does a "fallback" look like in a team?

- What is the signal that a workflow has been over-engineered? When does adding
  a new skill make the system better vs. make it more brittle?

- Is there a version of this that's adaptive — where the router learns from
  fallback cases and, over time, suggests new routes from work that's recurred
  enough to merit one? Or does that just recreate the over-routing problem on
  a longer timescale?

- The skills you've built map to your work at a specific company (FLOW, FDSE
  migration, Snyk). Skills are reusable within a context, not across contexts.
  Is the right unit of reuse the skill, or the principle behind the skill?

---

*Article seed. Return to this when the router has been in use long enough to
produce real examples of where it helped and where it got in the way.*
