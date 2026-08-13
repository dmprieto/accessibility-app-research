# Record-keeping

How this repo's documents may be changed, and by whom.

These are not product decisions — those are in [DECISIONS.md](DECISIONS.md). These are the
terms the record itself is kept under. They bind everyone who edits these files, human or
agent; every rule below exists because something went wrong, and most of the failures came
from both.

---

## Scope and expiry

**This file governs the research phase only.** Right now the documentation *is* the
deliverable, and five files restate each other, so drift between them is the main defect
class. In the shipping repo the deliverable is code and the equivalent disciplines already
exist and are better — the CI ratchet, review, tests. These rules are replaced there, not
carried over.

**So this file is meant to be deleted when research closes** — with the exception in *What
graduates* at the end, which must be carried forward first. Deleting it without doing that
would repeat the exact failure it exists to prevent.

---

## 1. Every claim carries its provenance

The vocabulary already in use across these documents, made explicit:

| Label | Means |
|---|---|
| **Measured** | Produced by an instrument, on stated hardware, on a stated date |
| **Inferred** | Follows from something measured, but was not itself observed |
| **Recorded from design discussion** | Reasoned, never prototyped, never measured |
| **Retracted / corrected** | Was believed, is now known wrong. The wrong version stays visible |

**A claim without one of these is incomplete**, and confidence language is not a substitute:
"clearly", "obviously" and "verified" are not provenance.

Worked examples in the repo: `getDefaultSensor(TYPE_PROXIMITY)` returning null on device 2 is
marked **inferred**, because it follows from the hardware inventory rather than from a call.
Tilt's rejection and the edge slider's deferral are marked **recorded from design
discussion**, so they do not sit at the same apparent weight as the measured items beside
them.

**State verification limits rather than papering over them.** An agent cannot run `adb`, see a
device, or check `aapt2` output. Sourcing a command from this repo is not the same as running
it, and the difference must be said out loud each time. An editor who stops distinguishing "I
verified this" from "I read this here" corrupts provenance from the inside while appearing
more useful.

*Why:* on 2026-08-13 the README claimed the privacy properties were "verified on both devices
in one `dumpsys` command". Two of the three are runtime properties; the third is a fact about
the built APK and no `dumpsys` command can answer it. The claim was confident, plausible, and
wrong, and it took three passes to clear.

## 2. One definition. Everything else points at it

A load-bearing claim has exactly one authoritative home. Every other mention **points** rather
than restates. Where duplication is genuinely needed — a section that must be readable on its
own — say which side is the definition, so a future drift has a stated resolution rather than
requiring a judgment call.

The worked example is standing rule 4 and the ratchet spec: the spec restates the
runtime-versus-artifact distinction because whoever builds it needs it in front of them, and
the note there records that if the two ever disagree, **the spec gives way and rule 4 wins.**

**Summaries are allowed to be summaries.** `HANDOVER.md`'s confidence table does not need
every qualification `DECISIONS.md` carries. It needs to be *true*, and to point somewhere
that is complete.

*Why:* the same "one command" claim survived in four places at different levels of detail, and
each correction pass found another. The last one — `HANDOVER.md`'s confidence table — was
missed twice, including once after being checked.

## 3. Verify the claiming unit, not the fragment

A documentation defect usually lives in a *relationship*, not a sentence: a row title against
its cell, a heading against its body, a section count against the rows beneath it, a summary
against its definition. Checking the fragment you were pointed at will return clean while the
claim it belongs to is false.

Before accepting any line as correct, read the smallest unit that makes a claim to a reader.

*Why:* `HANDOVER.md:19`'s basis cell read as accurate in isolation and was reported as fine.
The row *title* said "Privacy properties" plural, which made it a claim the cell could not
support. This is now the seventh entry in standing rule 5's null-instrument table, and it was
generated during a correction pass, not inherited.

## 4. Reserved decisions

**These are never settled by an agent, and never settled by inference.** The default on
ambiguity is to stop and ask — not to pick the reasonable option and flag it.

- **Anything concerning a participant.** Consent, what they were told, what may be quoted,
  what may be recorded about them. Only the person who was in the room knows, and a
  plausible-sounding reconstruction is a fabricated consent record.
- **Anything committing scope.** What v1 contains, what is deferred, what is required. Note
  the failure mode has two directions: recording a superseded scope, and recording an
  expanded one the evidence cannot carry. Standing rule 3 governs the second.
- **Anything resolving a conflict between standing rules**, or granting an exemption from
  one. A design existing is not evidence that a rule was waived.
- **Items already parked.** Once parked, an item is not re-raised. See
  [PARKED.md](PARKED.md) for what un-parks one.

*Why:* the consent line had gone stale and claimed the file's only quotes were the
developer's own, when it carried three of Observer A's. Rewriting it required a fact —
whether they were told before the session — that nobody but the developer had. Writing it
either way from inference would have produced a false consent record in a public repo with
permanent history.

## 5. Writing and publishing are separate authorities

Committing is reversible. Pushing is not: this repo is public, git history is permanent, and
it carries a real participant's words. **An agent may write and commit freely. A human decides
what becomes public.**

*Why:* Observer A's quotes were already published before anyone asked whether the disclosure
covered publication. It resolved well — they were asked and agreed on those terms — but the
question arrived after the answer could still have mattered.

## 6. Review is not optional, and not self-review

Changes to these documents get a second reader who did not write them. **Not a second pass by
the same editor.**

This is standing rule 3 applied to the process rather than to readers. An editor reviewing
their own work converges on "it's fine" for the same reason the sensitised reader did: serial
self-comparison against a memory of one's own intent is a null instrument for the defects that
matter. It cannot see a claim it already believes.

*Why:* on 2026-08-13 a paired review caught, in both directions, what neither reader found
alone — one found the stale consent line and the drifted confidence-table row; the other found
that the proposed `dumpsys` fix was itself wrong (the check is `aapt2 dump badging`, against
the artifact) and that the phrase needing correction was in seven places, not four.

---

## What graduates

**Do not delete this file without carrying these forward.** They are not research-phase
conveniences — they get more load-bearing after research closes, not less.

- **The participant half of rule 4**, and all of `NOTES.md` stage 4. Right now there is one
  observer who agreed twice and about whom nothing bodily is recorded. A beta means twelve
  disabled testers, consent decided before sessions rather than confirmed after, and
  retention and withdrawal actually specified. `NOTES.md` rule 5 already says this is
  human-subjects work; that is the thing this becomes.
- **The provenance vocabulary (rule 1).** The shipping repo needs the measured/inferred
  distinction the moment it records anything it did not measure.
- **Rule 5, if the shipping repo is public**, and unconditionally while this one is.

Rules 2, 3 and 6 can go. They exist because documentation is currently the deliverable and
five files restate each other. Where code is the deliverable, the ratchet, review and tests
do this job better.
