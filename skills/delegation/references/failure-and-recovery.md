# Failure and recovery

Delegated work fails in ways single-threaded work does not, because a
delegated run has a transport between you and the work, a bounded context
window doing the work, and a process on the other end that can get stuck
without telling you. None of these failure modes are exotic — they are the
ordinary cost of running work somewhere other than directly in front of you —
but each one is invisible until you know to look for its specific shape.

## 1. How delegated runs fail

**Transport errors mid-run.** The connection between orchestrator and
delegate drops, times out, or errors partway through a task that was
otherwise proceeding normally. The delegate may have made real progress
before the drop; that progress is not automatically lost, but it is not
automatically visible either — see §2.

**Exhausted context.** A long task accumulates transcript — files read, tool
output, intermediate reasoning — until the delegate runs out of room to hold
both what it has already done and what remains. The failure mode here is
rarely a clean error; it's more often degraded quality near the end of a run,
as the delegate starts losing track of earlier constraints from the brief
while still technically producing output.

**Silent truncation.** A result comes back shorter than the task implies, with
no error attached — a generation cut off mid-file, a list that stops partway
with no indication it isn't the full list. This looks identical to a
completed, shorter-than-expected result unless you check the artifact against
what the brief actually asked for.

**Looping on a blocked step.** A delegate hits something it cannot resolve —
a missing dependency, an ambiguous instruction, a check that keeps failing —
and instead of stopping to report it (per `writing-the-brief.md` §6), retries
the same failing approach repeatedly, burning time and budget without
progress and without surfacing that it's stuck.

Each of these produces a different repair, which is why naming which one
happened matters before deciding what to do next.

## 2. Resume rather than restart

The expensive part of most delegated tasks is not the final write — it's
everything that led up to it: files read, external calls made, a partially
built understanding of a codebase or dataset. A full restart discards all of
that and pays for it again, even for a task that was 90% done when it failed.

Where the harness or transport supports it, **resume the existing run from
its transcript** rather than starting a fresh one. A resumed run keeps the
gathered context — the files it already read, the approach it had converged
on — and only has to redo the work between the last durable checkpoint and
the failure. This is the direct payoff of §3 below: a run that has been
writing durable, incremental output has a real checkpoint to resume from; a
run that was building everything in memory does not, and for that run,
resuming the transcript recovers the delegate's understanding but not its
half-finished artifact.

If resuming isn't available, at minimum inspect what the failed run already
produced before discarding it — a re-launched task that starts from scratch
next to a mostly-complete result from the failed attempt is wasted work
sitting right next to the thing that made it unnecessary.

## 3. Design for resumability from the start

Because partial output is the normal outcome of a long or fan-out delegation,
not an edge case, design the task to survive being interrupted at any point:

- **File-at-a-time output.** Write and close each complete artifact before
  starting the next, rather than accumulating everything into one buffer
  written at the end (this is `SKILL.md` §5 — it's worth restating here
  because recovery is where the payoff actually lands).
- **Idempotent writes.** A step that writes a file should be safe to re-run —
  overwrite cleanly, don't append and duplicate, don't require the previous
  attempt to have been cleanly rolled back first. A recovery step that has to
  first figure out whether the previous attempt half-succeeded is a second
  task hiding inside the first.
- **No half-written structured files.** If an output is a single JSON,
  YAML, or similar structured file describing many things, a crash mid-write
  produces a file that is not valid at all — worse than missing, because it
  can silently break whatever reads it next. Prefer many small files (one
  per record, one per unit of work) over one large structured file, precisely
  because "twelve of forty files exist" is a recoverable, checkable state and
  "the JSON array is truncated after entry twelve" usually is not.

## 4. When to stop resuming and re-scope

Resuming is the right default, but not unconditionally. Stop resuming and
re-scope the task instead when:

- The same failure recurs after a resume — the delegate hits the identical
  blocked step again, which means the resume preserved the problem along
  with the progress.
- The transcript being resumed has grown large enough that a fresh delegate,
  briefed with just the current state and the remaining work, would
  understand the task faster than one carrying the full history of how it
  got there.
- The reason for the original failure was a flaw in the brief itself — an
  ambiguous instruction, a missing non-goal, an output contract that turned
  out to be wrong — in which case resuming just continues executing the flawed
  brief faster. Fix the brief and restart cleanly instead.

The signal to watch for is a resume that fixes the symptom (the run
continues) without fixing the cause (the run was going to hit the same wall
regardless of where it restarted from).

## 5. The meta-failure: when delegation costs more than doing it directly

Every mechanism in this skill — a precise brief, personal verification, a
resumable design — has a cost, and for a small enough task that cost can
exceed what the task would have cost done directly, without a delegate at
all. This is worth naming plainly because the sunk cost of having already set
up a delegation loop makes it easy to keep paying that overhead rather than
notice it stopped being worth it partway through.

Signs a task should not have been delegated, or should stop being delegated
from here:

- **The brief took longer to write than the task would have taken to do.**
  This is not automatically a loss — a precise brief is often reusable — but
  for a genuine one-off, it's a sign the delegation added a step rather than
  removing one.
- **Verification requires rebuilding most of the delegate's context anyway.**
  If checking the work means re-reading everything the delegate read, the
  delegate's read pass bought nothing.
- **The task turns entirely on a judgment call that can't be specified in
  advance.** A brief precise enough to remove all judgment from a task that
  is *fundamentally* a judgment call either isn't possible or has quietly
  made the decision itself while pretending to just specify the format.
- **Recovery has been attempted more than once on the same task.** A task
  that has failed and been resumed or re-scoped multiple times is signaling
  that something about its shape doesn't fit delegation, not that the next
  attempt will be the one that works.

None of this means delegation was the wrong call in general — most of what
this skill covers exists because delegation is usually worth it. It means the
same verification instinct applied to a delegate's output belongs pointed at
the delegation decision itself: check whether it's working, don't assume it
is because it was the plan.
