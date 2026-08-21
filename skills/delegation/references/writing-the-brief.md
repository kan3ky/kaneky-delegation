# Writing the brief

A brief is the entire interface between what you know and what the delegate
can act on. It has no access to the conversation that led you here, no memory
of what you already ruled out, and no sense of why the task matters unless
the brief says so. Everything the delegate gets wrong that a precise brief
would have prevented is a cost you pay twice: once in the bad first draft,
once in the review cycle that catches it.

The examples below are invented for teaching. The shape generalizes; the
specifics of any one brief never will.

## 1. The task in one sentence

Before anything else, state what "done" looks like in a single sentence a
stranger could repeat back correctly. If you cannot compress the task to one
sentence, you have not finished deciding what you want, and handing an
undecided task to a delegate just moves the deciding onto someone with less
context than you have.

```
# Weak — a topic, not a task
"Look into the caching layer and improve it."

# Strong — a task, checkable
"Reduce the cache-miss rate on the product-lookup endpoint from ~40% to
under 10% by fixing the key-normalization bug that treats 'SKU-123' and
'sku-123' as different keys."
```

The strong version also does something the weak one cannot: it tells the
delegate when to stop. A vague brief invites the delegate to keep finding
more things to "improve" indefinitely, because there is no stated finish
line.

## 2. Worked examples beat description

Telling a delegate the shape you want, in prose, leaves room for a plausible
but wrong interpretation. Showing it one real, complete example of the shape
you want — an existing file that does the thing correctly, a sample output,
a reference implementation — collapses that ambiguity, because there is
nothing left to guess about the format once a concrete instance exists to
copy from.

```
# Weaker — described
"Write unit tests for each handler, following our usual conventions."

# Stronger — pointed at a worked example
"Write unit tests for each handler in payments/. Read
orders/handler_test.go first — that's the pattern: table-driven cases,
one subtest per branch, mock the store via the Store interface, assert
on the response body and status code. Match that structure exactly,
substituting the payments domain types."
```

"Our usual conventions" is doing no work — it names a fact the delegate does
not have access to. A worked example is the same information in a form the
delegate can actually use: read the file, extract the pattern, apply it.
When more than one convention exists in the codebase, name the correct one
explicitly rather than leaving the delegate to average across whichever
examples it happens to find first.

## 3. The output contract

State exactly what gets produced and where, in a form that is checkable by
inspection rather than judgment:

- **Paths.** Exact file paths, not "somewhere appropriate." If the delegate
  has to guess a directory, two runs of the same brief will guess
  differently.
- **Structure.** For anything structured — JSON, a config file, a directory
  tree — give the schema or a filled-in example, not a description of the
  fields.
- **Format.** Naming conventions, casing, whether a trailing newline matters,
  whether existing files get edited in place or new ones get created
  alongside them.

A contract that only exists in prose gets satisfied approximately. A contract
with an example the delegate can diff its own output against gets satisfied
exactly, and exact is what makes the result checkable without re-reading the
whole thing by eye.

## 4. Explicit non-goals

Say what the delegate should *not* do, especially where a reasonable delegate
might otherwise assume it's helping:

```
Non-goals:
- Do not refactor code outside payments/, even if you notice something
  that looks wrong.
- Do not add new dependencies. If the task seems to need one, stop and
  report that instead of adding it.
- Do not change the public function signatures in this package — other
  callers depend on them and are out of scope for this task.
```

Scope creep from a delegate is rarely malicious — it is usually a delegate
trying to be thorough, fixing an adjacent thing it noticed along the way.
Without a stated boundary, "thorough" and "in scope" are the same instruction
from the delegate's side, and the diff you get back is larger and harder to
review than the task warranted.

## 5. The verification the agent must run before reporting

Naming the check the delegate must run before claiming success does two
things: it raises the odds the delegate catches its own mistake, and it gives
you something concrete to re-run yourself afterward (see
`verification.md`). Name the exact command, not a description of the goal:

```
# Weak
"Make sure the tests pass before reporting back."

# Strong
"Run `go test ./payments/... -run TestHandler -v` and paste the tail of
the output in your report — the PASS/FAIL summary line, not a
description of it. If any test fails, do not report success; report
which test and the failure output."
```

The strong version also forecloses the most common failure of this
instruction: a delegate that runs a *different*, easier check than the one
that actually validates the task, and reports that as if it satisfied the
brief.

## 6. What to do when blocked

State explicitly, every time, that an honest gap is a better outcome than a
fabricated fill. This has to be said because the default behavior of a
model asked to complete a task is to complete it — including the parts it
cannot actually source, verify, or determine, which it will then fill with
something plausible rather than leave visibly empty.

```
If you cannot find the information a step requires — a config value that
isn't in the repo, a credential you don't have access to, a file the
brief refers to that doesn't exist at the stated path — stop and report
exactly what's missing and where you looked for it. Do not guess a
plausible value and continue as if you had found it. A gap you report is
useful; a gap you paper over is a bug I won't find until it matters.
```

Ask explicitly for the negative case in the final report: "list anything you
could not verify or source," not just "list what you did." A report
structured only around accomplishments has no slot for a delegate to mention
what it skipped, and a delegate under no instruction to mention gaps
generally doesn't.

## Before / after

**Vague brief:**

> Clean up the export module, it's kind of a mess. Make sure it still
> works.

**The same brief, made precise, with the reason each addition earns its
place:**

> **Task:** Split `export/bundle.py` (currently 1,400 lines, one file) into
> `export/csv.py`, `export/json.py`, and `export/shared.py`, with no change
> in behavior.
> *— replaces "clean up" with a checkable structural outcome.*
>
> **Read first:** `export/bundle.py` (the file being split) and
> `import/bundle.py` (a sibling module already split this way — match its
> file boundaries and its `shared.py` pattern for code used by more than one
> format).
> *— gives a worked example instead of leaving "how to split it" to
> judgment.*
>
> **Output contract:** three new files at the paths above; `bundle.py`
> becomes a thin module that imports and re-exports the public names from
> all three, so nothing importing `export.bundle` today breaks.
> *— makes "done" a structural fact, not an opinion about cleanliness.*
>
> **Non-goals:** do not change any public function's signature or behavior;
> do not touch `export/xlsx.py`, which has a pending unrelated change; do not
> add new dependencies.
> *— forecloses the "while I was in there" expansion that turns a mechanical
> split into a redesign.*
>
> **Verify before reporting:** run `pytest tests/export/ -v` and paste the
> full pass/fail summary. Run `python -c "import export.bundle"` to confirm
> the re-export shim imports cleanly.
> *— names the exact commands, so "make sure it still works" becomes
> something both the delegate and I can act on identically.*
>
> **If blocked:** if any test currently fails on the unmodified file, stop
> and report that before making changes — don't fix pre-existing failures as
> part of this task, and don't assume they're pre-existing without checking.
> *— separates "was already broken" from "I broke it," which the delegate
> cannot tell apart without being told to check.*

Every line added earns its place by removing one specific way the vague
version could have been satisfied incorrectly while still sounding done.
