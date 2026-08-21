---
name: delegation
description: Get reliable work out of subagents whose output you have not verified — writing a brief precise enough to produce good work, checking delegated work by re-running the check rather than reading the report, picking the cheapest model that clears the bar without moving the verification gate down with it, and recovering when a delegated run fails partway through. Use when orchestrating subagents, spawning parallel agents, delegating to a smaller or cheaper model, reviewing a subagent's summary before acting on it, or designing any loop where one process produces work and another must trust it.
---

# Delegation

A subagent's summary of what it did is generated text, and generated text has
the same failure modes regardless of who or what produced it. It will say "all
tests pass" because that is what a completed task sounds like, not because the
tests were run and passed. It will report "47 files processed" because a
plausible count is easier to produce than an honest one when the actual count
was never tallied. A confident report and a correct result are indistinguishable
in text — the only way to tell them apart is to look at what the report claims
to describe.

That is the spine of everything below:

> **Delegated work is untrusted until you have checked it yourself, and the
> report is not the check.**

The orchestrator's job is not to read reports. It is to run the check the
report claims would pass, on the actual artifact the delegate produced. This
skill is about the five decisions that make that practical: how to brief a
delegate so the first draft is usually good, which model to spend on which
task, where the line is that no delegate crosses, how to split work across
agents safely, and how to keep a long delegated run from losing its output to
a crash.

## 1. The brief is the quality control

Output quality tracks the precision of the specification far more than it
tracks the capability of the model doing the work. A precise brief handed to
a small, cheap model reliably beats a vague brief handed to the largest model
available — the large model fills the gaps the vague brief left with its own
guesses, and guesses are exactly what an untrusted delegate should not be
making on your behalf.

This makes brief-writing the single highest-leverage activity in the whole
loop. Time spent turning "clean up this module" into an exact list of files,
an exact output shape, and an exact definition of done is not overhead around
the delegation — it *is* the delegation. See `references/writing-the-brief.md`
for the anatomy of a brief that produces good work on the first pass, with a
worked before/after.

## 2. Cheapest model that clears the bar — with a hard gate that never moves

Most delegated work does not need a frontier model: bulk research, mechanical
transformation, file-by-file generation, vendoring, and other tasks that
decompose into many small, similar, low-judgment steps run fine on a small
model given a precise brief. Reserve the larger, more expensive model for the
fraction of the task that genuinely turns on judgment — resolving an ambiguous
requirement, reconciling conflicting evidence, deciding a design trade-off.

Three things never move down-tier, regardless of what the rest of the task
costs:

- **The verification pass.** A cheap drafter paired with a rigorous check is
  the pattern. A cheap check is not a gate — it is a second untrusted report.
- **Any safety or leakage check.** Anything that will be published, sent to a
  third party, or acted on outside the sandbox gets checked by the most
  careful process available, every time, with no exceptions carved out for
  time pressure.
- **The decision to commit, deploy, publish, or otherwise make the work
  externally visible or irreversible.** That decision concentrates in one
  accountable place — see §3.

State the expected cost of a fan-out before launching it, especially a large
parallel one, so the choice to spend that budget is made on purpose rather
than discovered afterward.

## 3. A trust boundary the delegate never crosses

Drafters produce changes. They do not commit, push, tag, deploy, or mutate
anything shared or irreversible. That boundary is not a courtesy to the
delegate — it is what makes the output auditable at all. If every subagent in
a fan-out can also land its own work, there is no single point where someone
with the full picture looked at the diff before it became permanent.

Concentrate the landing step in one place: one process (human or a
specifically trusted, higher-capability agent) runs the full verification
pass on everything a delegate produced, and only that process presses the
button that makes the change real. Everyone else hands off an artifact —
uncommitted, unpushed, unpublished — and stops.

This is not distrust of any particular delegate's competence. It is a
structural property: a system where the entity that *produces* work and the
entity that *ships* work are the same entity has no independent check between
generation and consequence, no matter how good the generator is on any given
day.

## 4. Parallelise on independence, not ambition

Agents touching disjoint files or disjoint resources run concurrently and
safely — their outputs cannot collide, so there is nothing to reconcile.
Agents that might touch the *same* file, the same record, or the same shared
state are not a parallelization opportunity; they are a race, and the result
you get back is whichever agent's write landed last, silently overwriting
the other's work with no error and no merge.

Before fanning out, partition the task explicitly by what each agent will
read and write, not by how the work happens to divide conceptually. Two
agents that each "handle authentication" from different angles are not
independent just because the brief describes them differently — check the
file list, not the task description.

## 5. Instruct incremental, durable output

An agent that writes and closes one complete file before starting the next
survives a mid-run crash, a transport failure, or an exhausted context window
with its completed work intact. An agent that builds the entire result in
memory or in a single in-progress buffer and writes only at the end loses
everything the moment anything interrupts it — the difference between a
total loss and a partial one.

Say this explicitly in every brief that produces more than one artifact: write
each output file to its final location and move on, rather than accumulating
everything and writing it all at the finish. It costs nothing when the run
completes cleanly and converts a crash from "start over" into "resume from
file twelve."

## Review checklist

Before delegating:

- [ ] Is the task in one sentence, and does the brief name the exact files to
      read first and why?
- [ ] Does the output contract specify paths, structure, and format precisely
      enough that "done" is checkable, not a judgment call?
- [ ] Are non-goals stated, so the delegate does not "helpfully" expand scope?
- [ ] Does the brief say what to do when blocked — name the gap, never invent
      a plausible fill?
- [ ] Is this the cheapest model that can clear the bar for *this* task, given
      how much of it is mechanical versus how much turns on judgment?
- [ ] If parallel, are the agents' file/resource sets actually disjoint?
- [ ] Does the brief ask for incremental, durable writes if it produces more
      than one output?

After the delegate reports back:

- [ ] Have you re-run the check yourself, rather than trusting the summary?
- [ ] Did you verify the property the task was supposed to produce, not that
      the process ran?
- [ ] Did you spot-check the hardest case in the output, not just the first
      one you looked at?
- [ ] For anything public or externally visible, did you personally run the
      safety/leak check — not delegate that too?
- [ ] Is any reported number (count, pass rate, size) something you checked,
      or something you are taking on faith?
- [ ] Did the delegate report anything it could not do or was unsure of? If
      the report is uniformly confident with zero caveats, that is itself
      worth a second look.

## References

- `references/writing-the-brief.md` — the anatomy of a brief that produces
  good work: task, worked examples, output contract, non-goals, required
  verification, and what "report the gap, don't invent" looks like in
  practice, with a full before/after.
- `references/verification.md` — the verification pass in depth: re-running
  checks instead of reading claims, verifying properties instead of process,
  where to spot-check, and the specific hazard of a delegate's self-reported
  numbers.
- `references/failure-and-recovery.md` — how delegated runs fail differently
  from single-threaded work, how to resume instead of restart, designing for
  resumability, and recognizing when a delegation loop costs more than doing
  the task directly.
