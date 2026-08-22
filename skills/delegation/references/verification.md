# Verification

A delegate's report and the state it describes are two different things, and
the report is the one you receive first. Verification is the discipline of
never letting that ordering substitute for checking the second thing. Every
item here is a way of turning "the delegate said X" into "I confirmed X,"
which are not the same claim even when X is true.

## 1. Re-run the check, don't read the claim

If the brief named a specific command to verify the work (see
`writing-the-brief.md` §5), run that command yourself against the delegate's
actual output. Do not treat the delegate's paste of its own output as
equivalent — a paste can be stale, cropped to the passing part, or from a run
against a different version of the file than the one currently on disk.

```
# Insufficient
Delegate report: "Ran the test suite, all 340 tests pass."
→ accepted as-is

# Sufficient
Delegate report: "Ran the test suite, all 340 tests pass."
→ you run `pytest -q` yourself, on the current working tree, and read
  the summary line it produces
```

This is not about assuming bad faith. It is about the fact that a model
producing a summary of a test run and a test runner producing the actual
result are different processes with different failure modes, and only one of
them is connected to ground truth. A model can misremember which tests it
ran, summarize a partial run as complete, or generate the sentence "all tests
pass" because that is the modal sentence a finished task produces — with no
mechanism forcing that sentence to be conditioned on an actual exit code.

## 2. Verify the property, not the process

A report that describes *what was done* is not the same claim as one that
describes *what is now true*, and only the second is checkable against the
artifact:

```
# Describes process — not directly checkable
"I called the redaction function on every record before inserting it."

# Describes property — directly checkable
"The store contains no unredacted email addresses or phone numbers."
```

The first claim requires trusting that the described sequence of steps
happened and happened correctly, for every record, with no exception. The
second claim is a fact about the current state of the store that you can
query directly — scan for the pattern the redactor was supposed to remove,
independent of whatever the delegate says it did. Whenever a report is
phrased as a sequence of actions taken, restate it to yourself as a property
of the end state, and check that property instead of the narrative.

This generalizes past redaction: "I sorted the list" is a process claim;
"the list is sorted" is a property claim, checkable with one pass regardless
of how it got that way. Prefer briefs that ask for the property up front
(§5 of `writing-the-brief.md`), and when a report only states the process,
derive and check the property yourself before trusting it.

## 3. Spot-check the hardest case, not the first

When a delegate produces many similar outputs — a batch of generated files, a
set of migrated records, a list of test cases — errors do not distribute
evenly across them. They cluster where judgment was required: the record
with an unusual shape, the file that didn't match the worked example as
cleanly as the others, the edge case the brief didn't explicitly cover. The
first item in a list is disproportionately likely to be the one the delegate
handled most carefully, precisely because it's the one most similar to
whatever worked example was in the brief.

Look for the outlier deliberately before accepting a batch:

- The largest or smallest item, by whatever metric applies.
- The one that took the longest to produce, if that's visible.
- The one whose input least resembled the worked example in the brief.
- Anything the delegate's own report flagged as uncertain — see §7.

A spot-check on the first three items in a list of forty tells you almost
nothing about the other thirty-seven; a spot-check on the one that looks
most different from the rest tells you where the batch was actually tested.

## 4. Run the safety and leakage check personally, every time

For anything that will be published, sent externally, or otherwise become
visible outside the working environment, the check for leaked internal
detail, unsafe content, or anything that shouldn't ship is not delegable —
not to a cheaper model, and not by trusting that the delegate that produced
the content already checked itself. Run it as the last step, on the final
artifact, every single time, regardless of how routine the task felt or how
many times the same kind of task has gone cleanly before.

The reason this can't be skipped "just this once" is that the failure it
catches is exactly the kind that looks identical to success until the moment
it's live somewhere it can't be recalled from. Routine is not evidence of
safety; it's the condition under which a check gets skipped right before the
one time it mattered.

## 5. The specific hazard of reported numbers

Counts, pass rates, sizes, percentages — any number in a delegate's report is
either something the delegate computed and can show its work for, or
something that reads as plausible because it's the kind of number a
completed task produces. Both look identical in the sentence "processed 214
records, 3 failed." Only one of them is a fact.

Treat any number in a report as a claim to verify, not a fact to record:

- Ask for the command whose output the number came from, and run it
  yourself, rather than the summary sentence built from it.
- If the number can't be traced to a specific check, treat it as unverified
  and check it independently — count the records yourself, measure the file
  yourself.
- Be specifically suspicious of round or tidy numbers reported without a
  visible source — real counts from real data are usually not round.

This is the same failure as §1 in a smaller, easier-to-miss form: a single
number slipped into an otherwise accurate-sounding report is far easier to
wave through than a whole false claim, precisely because the rest of the
report checks out.

## 6. Ask for command output, not summary

The single highest-leverage habit here: when reviewing a delegate's report,
ask for the raw output of whatever it ran, not its description of that
output. "Show me the actual `pytest` output" surfaces truncation, cherry-
picking, and misremembering that "did the tests pass?" does not. This is cheap to ask
for and should be the default shape of a report a brief requests — see
`writing-the-brief.md` §5 for building it into the brief itself, so you're not
requesting it after the fact.

Two bounds on it, though, because "paste everything" is neither free nor
automatically safe, and this skill says elsewhere that a delegate's report is
untrusted input rather than instruction.

**Bound it by size.** Name the output that settles the claim — the summary
line, the failures — rather than the whole log. An unbounded paste burns
context, and can itself be truncated on the way back, which reintroduces
exactly the ambiguity the raw output was supposed to remove.

**Bound it by sensitivity.** Command output can carry credentials, tokens,
customer data, or attacker-controlled text from something the delegate read.
Relaying it verbatim moves all of that into the orchestrator, which is the more
privileged context and the one that commits. Ask for the evidence that settles
the question, not for everything the command happened to emit.

## 7. The good sign: a delegate that shows its working

Not every signal here is a red flag to hunt for. A delegate that reports what
it could not do, names a gap instead of filling it, or corrects itself mid-
task ("I initially assumed X, but on rereading the file that's wrong, it's
actually Y") is showing you the parts of its process that a report designed
purely to sound complete would omit. That is closer to trustworthy than a
report with zero caveats and uniform confidence throughout — a real, non-
trivial task almost always has at least one place where something was
uncertain, ambiguous, or harder than expected, and a report with no trace of
that is more likely to be smoothing over a rough patch than to describe a
task that had none.

Reward this in the brief (§6 of `writing-the-brief.md`, explicitly inviting
reported gaps) and reward it in review: treat a caveat-laden report as more
credible than a spotless one, not less, and verify both the same way either
way.
