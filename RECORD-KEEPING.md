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

### A publication gate must live where copying happens, not only in the prose it guards

When a passage is held back from publication for a reason of its own — an unfixed vulnerability
written up in full, a quote not yet cleared, anything whose *reason* to stay private differs from
the file's — **the hold is marked in the text with the literal token `[[DO-NOT-PUBLISH]]`, on its
own line at the top of the passage, naming the condition that releases it.** The prose reasoning
stays too; the token is what a machine can find.

**Two obligations follow, and they bind the copy, not the reader:**

- **A file carrying `[[DO-NOT-PUBLISH]]` is not copied into, or pushed to, any repo more public
  than the one it sits in** until the developer clears the token. Not the passage — the *file*.
- **Every sanitisation or copy-to-publish task greps for the token as an explicit criterion,
  alongside personal and device data.** A task told to remove personal data will remove personal
  data; a publication hold recorded only as an argument is invisible to it.

*Why:* on 2026-08-19 a worked escalation chain, carrying its own in-prose deferral —
*"not routed to the public repo yet, by decision"* — was published to a remote anyway, by a
sanitisation task correctly scoped to personal and device data. **A correct application of the
criteria it was given published the thing the document said not to publish**, because the gate
lived where a reader of the document would see it and a file-copying task would not. This is the
run's own recurring defect — a check pointed at the wrong thing — at the level of the process. A
gate that only humans reading in context can see is not a gate.

### Raw device dumps are personal data, and the distilled line is the half a sweep misses

*(Added 2026-09-04. This guardrail is the **agent's**, surfaced by the doc-cleanup pass — not a recorded
developer/coordinator decision; marked so per rule 1's provenance discipline.)*

Raw device captures — `adb shell settings get secure` / dumpsys output, e.g. `secure_*.txt` — are
**personal data**: they carry `android_id`, `serial`, `bluetooth_address`, `ssid` (a home network name),
`account` and `owner`. They are never published raw; if a derived result must go public it is the
**finding**, after an identifier check — never the dump. A **filename sweep** (`secure_*`) catches the
dump files.

**The half a filename sweep misses is a value distilled out of a dump into a prose line.** When an
identifier is lifted from a capture into a markdown sentence — a device id quoted in a finding, an SSID
named in a run note — the file it lands in has no telltale name, and a sweep for `secure_*` passes over
it. That distilled line is the half that matters, and it is the **same shape as the escalation-section
gate above**: a task scoped to one form (files named like dumps) acting on content it wasn't asked to
read (identifiers living inside other files). So a copy-to-publish or sanitisation task greps for the
**identifier patterns themselves**, not only the dump filenames — alongside `[[DO-NOT-PUBLISH]]` and
personal/participant data.

**A found instance, not a hypothetical:** `control-path-charter/NEXT-SESSION-switch-arm.md:44` carries
the **SM-P200 serial in plain prose** (plus settings-derived values like `accessibility_enabled=1`) —
a value distilled out of a device capture into a markdown line, exactly the half a `secure_*` sweep
misses. Local-only, so nothing is exposed today; recorded here so the class acts on a real case rather
than a described risk.

## 6. Review is not optional, and not self-review

Changes to these documents get a reader who **did not work on the changes under review**. Not a
second pass by the same editor.

**When every available instance worked on the change, the developer is the reader** — not a relaxed
bar, an escalation. Rule 5 already puts the developer at the point where things become public; this
routes a conflicted review to the same place rather than tempting a widening of "worked on" to
manufacture an eligible agent. The bar is what catches the defects; it is the thing to protect, so
the escape hatch is escalation, never dilution.

**The test is the rationale, not a list of roles.** The bar exists because serial self-comparison
against a memory of one's own intent is a null instrument — so ask, of a given change, whether the
reader would be reading against such a memory.

- **A reviewer whose finding caused an edit would not.** The memory is of the finding, not of the
  prose. That is the benefit named below, and **without it no review round could ever close.**
- **A reader who composed a change's substance would**, however it reached the page.
- **Where a change was fully determined by the finding** — restoring a deleted clause, surfacing an
  overwritten sentence — **nobody composed it, and there is no memory of one's own intent to be blind
  to.**

**Applied per change, not per round.** A reader may be barred from one change and qualified for the
rest of the same round.

This is standing rule 3 applied to the process rather than to readers. An editor reviewing
their own work converges on "it's fine" for the same reason the sensitised reader did: serial
self-comparison against a memory of one's own intent is a null instrument for the defects that
matter. It cannot see a claim it already believes.

**A reader who has reviewed this document before still qualifies.** *This paragraph clarifies; the two
below add a benefit and a preference that were not here before.*

The bar above is **participation in the changes under review**. **"Not a second pass by the same
editor"** is the sentence that reads as barring a repeat reviewer, and it does not: the heading is
*not self-review*, and the failure mode beneath it is *"an editor reviewing their own work"*. Both
name the writer. A returning reader has not worked on the changes under review, and unfamiliarity is
required nowhere in this rule.

**One thing a returning reader can do better than a new one:** check whether its own earlier findings
were applied **as the finding intended**. A fresh reader given the prior review can check that
something changed; it cannot always tell whether the change answers what was meant, where the written
finding under-specifies. *This is not the null instrument described above — that is an editor
re-reading a memory of their own prose. Here the object is someone else's edit, and the memory being
consulted is of the finding, not of the text.*

**The cost is real and is not resolved here.** This rule borrows standing rule 3's reasoning by
analogy — that is the claim of the *"standing rule 3 applied to the process"* paragraph above, and
this paragraph does not extend it — and rule 3's evidence is a
returning observer degrading across four sessions, in `DECISIONS.md`. **One in-repo instance is
checkable here:** rule 2's *Why* records `HANDOVER.md`'s confidence table *"missed twice, including
once after being checked"* — a defect that survived a pass over text already read, which is the
failure this rule guards against. *Two further instances, from the pre-spike-2 device run, are
**not in this repo** and a reader here cannot check them; recorded as reported, at that ceiling.* **No comparison of a returning against a fresh reader on the same document has ever been run
here**, so which is better is unmeasured and neither is claimed. Prefer a reader who has not seen the
document when the stakes justify it.

**Settled by the developer, 2026-08-19.** The bar is participation in the changes under review — not
authorship of the file, and not unfamiliarity with it. Anyone who did not work on those changes
qualifies, including a reader who has reviewed this document many times and including someone who
wrote earlier, unrelated parts of it. The cost recorded above is real and unmeasured; it stays a
preference, not a bar.

*This closes the item previously left open here, which read: "**Open, and for the developer rather
than a reviewer:** this narrows what rule 6 inherits from standing rule 3 — enough to name the cost,
not enough to require freshness. That call is not confirmed."* **The opening clause is the in-file
warrant for the label above** — it is why this was the developer's to settle and not a reviewer's,
and an earlier version of this quotation dropped it.

**Retracted in `212eefa`, and preserved here because rule 1 requires it.** This section
previously asserted **"The bar above is authorship"** — stated in the file and in commit `839e944`'s
own subject line, *"Clarify rule 6: the bar is authorship, not unfamiliarity"*, three days earlier.
**That is now wrong**: the bar is participation in the changes under review. *The first version of
this settled paragraph preserved the open item, which was never wrong, and silently overwrote the
sentence that was — applying rule 1's retention to the wrong one of the two.*

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
