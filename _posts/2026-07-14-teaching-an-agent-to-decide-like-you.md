---
layout: post
title: Teaching an Agent to Decide Like You
---

*Notes from my DREAM fellowship in Lydia Chilton's lab at Columbia*

## The gap between doing and deciding

AI agents have gotten good at doing things. Ask one to send an email, book a table, or fill a spreadsheet, and it will. What they are still bad at is doing things *the way you would have done them.*

That gap matters most in the small, recurring decisions that fill a week. Who do I put on this project? Which of these three people do I ask first? Do I push the meeting or drop the person who keeps missing it? These are not tasks you can hand off with a prompt, because the instruction isn't the point — the *judgment* is. Delegating them means the agent has to know not just what you'd do, but why.

So the question I took up this summer was narrower than "can an agent act autonomously," and, I think, more useful: **can an agent learn a specific person's decision-making well enough to earn the right to act for them — incrementally, auditably, and without quietly going wrong?**

## Why coordination is the right place to test this

I needed a task where good judgment is genuinely required and where a naive automation would visibly fail. Everyday group coordination turns out to be close to ideal, and for reasons that generalize:

- **Preferences are private.** People don't tell you why they're actually in or out. You infer it from behavior, over time, and the behavior can be a bluff.
- **Commitment is unreliable.** A "yes" is not a promise. People over-commit, stall, go quiet, and renege at the last minute.
- **There is no infeasible state.** The event happens regardless. You never "fail" — you just do it better or worse. That makes it an optimization-flavored problem, not a constraint-satisfaction one, and it means the agent has to *judge* rather than *solve*.
- **Participation is endogenous.** This is the interesting one. Your allocation decisions change who is willing to participate next time. Choose the strongest option today and the people you passed over are less likely to be available tomorrow. The decision and the decision environment are coupled.

Any coordinator — a manager staffing shifts, a professor assigning office hours, a friend organizing a game — is standing in that same structure.

## What I built

The output is a domain-agnostic engine, plus a simulation harness to exercise it.

**A policy learner.** The system watches the user decide and induces explicit rules of the form `condition → action → rationale`. Crucially, it captures the *rationale*, not just the action, because the same action has many possible reasons, and the wrong "because" generalizes badly. When it proposes a rule back, the user is confirming the reason, not the move.

**An autonomy ladder.** A rule doesn't get to act just because it appeared once. It climbs: *proposed → confirming → autonomous*, and it falls back the moment the user overrides it. Autonomy is earned per-decision, not granted globally.

**Novelty flagging.** Before learning anything, the system asks whether this decision is *new* — an unfamiliar decision point, or one that contradicts a rule it already holds. If so, it stops, flags, and asks, rather than silently appending a policy. This turned out to do double duty (more below).

**Cross-domain memory.** Learned facts and policies persist and are scoped. Domain-specific knowledge stays local; general rules are promoted so they carry to the next problem. A new domain plugs in through a small adapter; the engine itself doesn't change.

**A multi-agent testbed.** To exercise all of this I simulate the other participants with LLM agents who have private motives they never state, who misreport, hold out for better offers, and bail at the last minute. Realistic failure — an ankle rolled in warmups, a "yes" that evaporates — is part of the environment, not an error.

## What I learned

**Cloning the action is easy; cloning the reason is the whole problem.** My first version happily learned "he picks the strongest option" from a single instance. But the same pick could have meant "I value raw quality," or "I needed a specific capability," or "he apologized to me last week." Learning the wrong reason produces an agent that is confidently wrong the first time the situations diverge. Everything downstream — when to act, when to ask — depends on getting the *because* right.

**Autonomy and interruption are the same mechanism.** I initially treated "act on your own" and "flag weird things" as opposing goals: one means asking less, the other means asking more. They aren't. The novelty detector is exactly the gate that decides when autonomy should escalate to the human. Act on the routine; raise a hand on the unfamiliar. An agent without that detector is not more autonomous — it is just less careful.

**A faithful clone will faithfully reproduce your mistakes.** If the system learns your rule and runs it, it will also run it when your rule is wrong. That forced an explicit design decision: is this a *mirror* or a *coach*? I landed on mirror-with-warning — it acts as you would, but flags when your own rule looks likely to backfire. That's a design stance, not a technicality, and it's the crux of what "aligned to a user" actually means.

**You cannot simulate reality, and you shouldn't try.** I spent real time chasing behavioral realism — do the simulated people *sound* like people? — before recognizing the better target is **decision fidelity**: does deciding inside the simulation exercise the same judgment the real task would? A model that is wrong in useful ways beats one that is merely lifelike.

**The simulated participants surprised me, and that was the signal.** Agents who were passed over started holding out, negotiating, and going quiet. One demanded a guaranteed role as a condition of participating — got the guarantee — and then didn't show up anyway. None of that was scripted. It's a reminder that these dynamics emerge from incentives, not from authorship, and it's the strongest evidence I had that the testbed was doing its job.

## Where this goes

The system is a prototype and a testbed; I have no user-study results yet, and that is the obvious next step: put real users in front of it and measure whether the learned policies actually match their judgment, whether the autonomy ladder earns trust at the right pace, and whether people can tell when the agent has learned the wrong reason.

Beyond that, three threads:

1. **Transfer.** Learned rules should carry across domains. Testing that a policy induced in one coordination task meaningfully helps in an unrelated one is the real claim, and it's untested.
2. **Better novelty and importance.** Right now novelty is structural (new or contradictory). Genuine out-of-distribution detection, and a notion of *stakes*, would let the agent interrupt for the right reasons rather than merely the unfamiliar ones.
3. **Failure at the edges.** The interesting cases are where the user's own policy is incoherent, or where two learned rules collide. Those are where delegation actually breaks, and where an agent's willingness to say "I don't know what you'd want here" is worth more than its ability to act.

The long-term goal is not an agent that does more. It's an agent that has earned enough of your judgment to be trusted with less supervision — and that knows, reliably, when it hasn't.
