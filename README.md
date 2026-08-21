# kaneky-delegation

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-6C4FF7)](https://docs.claude.com/en/docs/claude-code)
[![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)](#install)
[![Config](https://img.shields.io/badge/config-zero-brightgreen)](#install)

A Claude Code skill for getting reliable work out of subagents you do not
trust — writing briefs precise enough to produce good work on the first
pass, checking delegated work without reading the report, picking the
cheapest model that clears the bar, and recovering when a delegated run
fails partway through.

Most delegation advice stops at "spawn an agent and read what it says." This
covers what happens after: the report says "all tests pass" because that's
what a finished task sounds like, the fan-out costs more than the task would
have cost done directly, and a subagent that goes quiet mid-run either lost
its progress or never had anywhere durable to put it.

## Install

```sh
git clone https://github.com/kan3ky/kaneky-delegation
cp -r kaneky-delegation/skills/delegation ~/.claude/skills/
```

No configuration. No dependencies. Ask Claude Code to write a brief for a
subagent, review a delegate's report, plan a parallel fan-out, or recover a
delegated run that failed partway, and the skill loads itself.

## Use it

Ask in plain language. The skill loads when the work matches it.

```
Write a brief for a subagent to migrate this data format.
This subagent's report says all tests pass — how do I actually check that?
Can these three research tasks run in parallel safely?
This delegated run died halfway through a 40-file generation — now what?
Should this task even be delegated, or is it faster to just do it?
```

It answers with what to check, how to check it, and what the delegate's
report cannot tell you — and it will say plainly when a task is not worth
delegating at all.

## What it catches

| Failure | Why it survives a confident report |
|---|---|
| **A report describing the process instead of the result** | "I redacted every record" is a claim about what happened; "the store contains no unredacted values" is a claim about what's true now. Only the second is checkable against the actual artifact. |
| **A vague brief on a capable model** | Output quality tracks specification precision more than model size. A vague brief gets satisfied by the delegate's best guess at what you meant, and the guess is where the divergence starts. |
| **A plausible, unchecked number in the report** | "Processed 214 records, 3 failed" reads identically whether the delegate counted or produced the kind of number a finished task tends to report. Only tracing it to a command's actual output tells them apart. |
| **The first item in a batch getting checked, not the hardest one** | Errors cluster where judgment was required, not at the front of the list. A spot-check on item one tells you how carefully the easiest case was handled. |
| **Two agents editing one file "in parallel"** | Independence is a property of what gets read and written, not of how the task description divides. Overlapping writes produce silent last-write-wins, not an error. |
| **A subagent that built everything in memory, then crashed before writing it** | Incremental, durable output turns a transport failure into "resume from file twelve." A single end-of-run write turns the same failure into a total loss. |
| **A delegate looping on a step it can't resolve, instead of naming the gap** | The default completion behavior fills a gap with something plausible rather than leaving it visibly empty — unless the brief explicitly asks for the honest gap instead. |
| **A fan-out that costs more than the task would have cost directly** | The overhead of briefing, verifying, and recovering delegated work is real and sometimes exceeds just doing it — a cost worth checking, not assuming away because delegation was the plan. |

## The idea behind it

Every item shares one shape:

> **A confident report and a correct result are indistinguishable in text.**

A subagent says "all tests pass" whether or not it ran them to completion. It
reports a clean count whether or not anything was actually tallied. Nothing
about the sentence changes based on whether it's true — which means the
sentence itself is never the evidence. The evidence is whatever you check
yourself, against the artifact, after the report arrives.

The skill is a set of ways to make that check cheap enough to actually do
every time: a brief precise enough that there's less to check, a
verification habit that looks at properties instead of process, and a
recovery design that assumes the run will fail partway at some point and
survives it when it does.

## Scope

Covers briefing, verification, model selection, parallelization safety, and
failure recovery for delegated work in any orchestration setup — a
Task-style subagent tool, an SDK agent loop, or a hand-rolled orchestrator.
It does not ship a specific harness, a specific model router, or a specific
CI integration; it is the reasoning that applies before you wire any of
those up, and it says plainly when a task should not be delegated at all.

## Contributing

Failure reports are the most useful contribution — especially ones where the
subagent's report looked completely fine. Include the symptom, what the
report claimed, and the check that would have caught the gap.

## Part of a collection

One of twelve Claude Code skills about failures that look like success —
GitOps, diagnosis, integrations, auth, end-to-end testing, visual verification,
agent guardrails, agent memory, extraction, providers, delegation and corpus
curation.

```sh
/plugin marketplace add kan3ky/kaneky-skills
```

Browse them at [kan3ky/kaneky-skills](https://github.com/kan3ky/kaneky-skills).

## Licence

MIT.
