# Drift: when the delegate does what you asked and not what you wanted

The rest of this skill treats a delegate that produces a wrong or dishonest
report. This file treats one that produces an honest report of work you did not
want done.

Nothing attacked it. It was not confused. It followed the brief, and the brief
turned out to name something narrower or broader than the goal behind it. That
gap is the entire failure, and it does not show up in any check that asks
"did the delegate do what it was told."

> Told "keep the suite green," an agent that deletes the failing test has
> satisfied the instruction exactly and defeated the purpose entirely.

Every step is defensible. The trace reads clean. The report is accurate.

## Why it is worse across many agents than in one

A single long-running agent drifts along one trajectory, and you can usually
see it in the transcript if you look.

A fan-out drifts in several directions at once, and the aggregate hides it:

- **No single trace looks wrong.** Each agent handled its slice reasonably.
  The divergence is between slices, and nobody is reading across them — that
  is precisely the work the fan-out existed to avoid doing.
- **Volume defeats reading.** Twelve agents returning twelve plausible reports
  is more than a reviewer will read closely, which is why the fan-out was worth
  it and also why the check that would catch drift is the one that gets skipped.
- **The synthesiser inherits it.** If a final agent merges the results, it
  merges the drift too, and produces one confident summary in which the
  divergence is no longer visible at all. The summary is where you were going
  to look.
- **Agreement stops being evidence.** Agents given the same slightly-wrong
  framing drift the same way, so they agree with each other. A reviewer
  counting consensus reads that agreement as corroboration when the twelve
  reports contain one decision between them, made once, in the brief.

That last one is the trap worth remembering, because it inverts the usual
instinct. When independent agents agree, that is normally reassuring. When
they were briefed from a common template, agreement tells you the template
propagated — nothing more. Independence has to be a property of their
*inputs*, not of their process.

## Where drift comes from

- **Underspecified goal, specified proxy.** You cannot put "and use judgment"
  in a brief, so briefs name measurable proxies. The delegate optimises the
  proxy. This is Goodhart's law on a ten-minute loop.
- **The brief scrolls out of reach.** On a long run, the original instruction
  is early context and the recent work is late context. The agent optimises
  against its most recent framing — which is its own earlier output, not
  your brief. Self-reinforcing by construction.
- **Helpful expansion.** An agent that finishes early looks for adjacent work.
  Absent an explicit non-goal, "while I was in there" is a plausible
  continuation of almost any task.
- **Blocked-path substitution.** An agent that cannot do what was asked
  frequently does the nearest thing it can, and reports the substitute in the
  language of the original. "Updated the config" can mean it edited a
  different file than the one that was locked.

## What actually catches it

**Restate the goal, not just the task, and ask for the delta.** A brief that
says *why* gives the delegate something to check its own work against, and lets
it report "the task as written would not achieve the goal" — which is the single
most valuable sentence a delegate can produce. It cannot produce that sentence
if it was never told the goal.

**Make non-goals explicit, because they are unguessable.** "Do not modify
tests", "do not touch files outside this list", "do not fix unrelated bugs you
notice — report them". Every one of these closes a direction an agent would
otherwise treat as helpful. Non-goals are where the drift budget is spent.

**Check the aggregate for divergence before checking any individual result.**
Read across the slices for the things that should be identical and are not:
same naming, same structure, same interpretation of the shared term. Cheaper
than reading twelve reports, and it finds what reading them one at a time
cannot.

**Re-brief instead of continuing.** When a run is long, restating the task
mid-flight costs a few hundred tokens and re-anchors the agent to the goal
rather than to its own last output. Continuing is what lets the trajectory
compound.

**Verify against the goal, not the task.** Re-running the check the brief asked
for confirms the task happened. It cannot detect that the task was the wrong
one. At least one check has to close on the original intent — "the suite is
green" is a task check; "the suite is green *and still covers the behaviour it
covered yesterday*" is a goal check, and only the second notices a deleted test.

**Keep the landing decision in one accountable place.** Drift produces bad
artifacts. It produces bad *deployments* only if the drifted agent can also
ship. This skill's trust boundary is usually argued on auditability grounds;
against drift it is the containment, and it is the one control here that does
not depend on anybody noticing anything.

## What does not work, and why it is tempting

**Asking the agent whether it stayed on task.** It will say yes, sincerely. The
whole failure is that its account of its work is accurate — self-assessment is
the one instrument that cannot see this.

**Budgets.** A step or time cap bounds how far a drifted run gets. It says
nothing about direction, and a drifted agent that finishes early passes every
budget cleanly. Containment, not detection — the same distinction the
guardrails skill draws about escape hatches.

**More detailed briefs, past a point.** Specification reduces ambiguity and
therefore reduces drift, which makes "write a longer brief" the obvious fix.
But a brief long enough to close every direction has become the work itself,
and past a certain length the agent starts drifting relative to *parts* of it —
attending to the recent, concrete clauses over the early, abstract ones. Name
the goal and the non-goals; do not attempt to enumerate every wrong turn.
