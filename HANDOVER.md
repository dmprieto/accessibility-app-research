# Handover brief — spike1-autoscroll

Written for someone picking this up cold. Read this before `README.md`, and read the
retractions before trusting any number in either.

## CLOSED — 2026-08-12

Spike 1 is closed. Its question was answered on two devices; everything after that was
scope creep, some of it valuable and none of it what this spike was for.

**Read the confidence levels below before acting on anything here.** The engine results rest
on instrumented measurement across two devices. The perceptual results rest on **one test
user**, and one user cannot support design decisions — they generate hypotheses for a beta,
not inputs to a build.

| Finding | Confidence | Basis |
|---|---|---|
| Held-finger chaining scrolls third-party apps smoothly | **High** | Two devices, 5 Android majors apart, zero engine changes for the second |
| Privacy properties are OS-enforced | **High** | `capabilities=32` on both devices, plus no `INTERNET` in the built APK — **two checks, not one**, both runnable by anyone. See standing rule 4 in DECISIONS |
| Touch-to-stop works; no spurious cancellations | **High** | 9-min soak, 8-item provocation pass, physical positive control |
| Delivered speed within ~1% after rate correction | **High** | Swept 4–100 dp/s, both devices |
| Re-grip costs ~450ms, travel-bound | **High** | n=30+ across devices, 12ms spread |
| **The re-grip is not what bothers readers** | **Medium** | Consistent across 4 sessions, both text sizes, both segment durations — but one reader |
| Continuous "interference" tracks text size | **Low–Medium** | One controlled pair (matched re-grips and jitter), volunteered unprompted |
| Jitter reduction is imperceptible | **None — unresolved** | Serial memory-based comparison could not answer it. Do not act on this either way |
| A single dp/s cannot serve mixed media or varying line height | **Medium** | Two independent observations, both content-driven and ability-independent |

### Two constraints added after closure — read before designing the control surface

- **Standing rule 6: the app may not initiate, plan, or sequence anything.** Play's
  Accessibility API policy (effective 28 Jan 2026, as cited) excludes autonomous initiation,
  planning and execution. The engine satisfies this by construction — `eventTypes = 0` means
  it cannot observe state, so it cannot decide on it. **The exposure is entirely in the
  control surface**, and the most tempting fix to the restart problem — auto-resume once an
  interruption clears — breaks both this rule and rule 4. Restart must be a user action.
- **`isAccessibilityTool="true"` is a functional requirement**, not a submission tactic. It
  **is now set** in `res/xml/autoscroll_service.xml` with the reasoning inline. **The
  declaration ratchet that would keep it set is not built and belongs in the shipping repo**
  — DECISIONS.md carries the spec, including why it must read the built artifact rather than
  the source.

**The Play track is paused** until spike 2 delivers a control surface. Nothing to submit
until a user can start and stop the app without adb. The Play-track decisions taken while it
is paused — the applicationId, the developer account, the upload key, and the nearest
comparable app — are recorded in DECISIONS.md and not restated here. Two of them carry the
same trigger as the pause itself: a control surface plus the exported-receiver fix.

**The production applicationId is `io.github.dmprieto.reading`**, settled 14 Aug 2026 and
irreversible at first publication. **It has never been built or run.** The spike's
`dev.spike.autoscroll` is unchanged and stays in every command in this file — those were run
against a build carrying it. DECISIONS.md has the criterion, the superseded candidate and the
reasoning; none of it is repeated here.

### What NOT to do next

- **Do not tune anything on this evidence.** Segment duration, lead-in, margin: all have
  measured mechanical effects and no reliable perceptual mandate. One user, heavily primed
  by the end, is not a basis for changing a default.
- **Do not build fast-forward yet.** The need is real; the design is not settled, and the
  obvious version violates the sustained-input rule.
- **Do not treat the reader's fatigue reports as requirements.** They are the first sign
  that visibility was the wrong success criterion — which is worth a great deal, and is not
  the same as knowing what the right one is.

### What this spike actually cost, as a process note

The engine question was answered once two devices agreed. Everything after that — the
control-surface defects, the six enabled-but-not-working states, the perceptual sessions,
the lurch metric, the jitter chase — arrived because the work kept going rather than
stopping at the answer. Much of it is genuinely useful and none of it was in scope.

The tell was consistent: after the second device, **every new finding was about something
other than the scroll engine.** That is the signal to close a spike and open the next one,
and it appeared several hours before the spike actually closed.

## What this is

A throwaway spike testing one assumption: can an AccessibilityService produce smooth, slow,
continuous scrolling in third-party apps by holding a single synthetic finger down and
extending it via `continueStroke()`, rather than dispatching repeated discrete swipes?

**Answer: yes.** Verified on a Moto G54 (Android 15, phone) and a Samsung SM-P200
(Android 11, tablet) — five Android majors apart, two manufacturers, both form factors,
both navigation modes. The second device required **zero engine changes**.

**Companion document:** [DECISIONS.md](DECISIONS.md) carries the product decisions with
their reasoning, the standing design rules, the retractions in one table, and the open items
with what would settle each. It is the part meant to outlive this repo.

## Scope: spike 1 is closed for engine work

Everything found after the second device landed concerns the **control surface**, which was
an explicit non-goal here. `README.md` ends with a "Handover to a control-surface spike"
section carrying one reproducible open bug and seven verified defects. Start there, not in
`ScrollEngine.kt`.

## Read the retractions before trusting any number

Several findings were recorded, disproved, then corrected in place. The corrections matter
more than usual, because a later reader cannot re-run these and has to know which held up.

- **Chain-age decay does not exist.** Cancellations at 48s and 62s looked like decay; the
  cause was sub-pixel segments produced by the decel ramp. The pre-fix bounds (36s good /
  48s bad) are **superseded and not comparable** — do not carry them forward.
- **The band is 25%–75%**, not 12%–88%. It was widened mid-spike on a theory that was wrong
  in both directions, then narrowed back.
- **A window change alone does not cancel the stroke, but a user-initiated app switch
  does** — a real switch is a finger, and fingers cancel in ~10ms. Both the original claim
  and its first correction were wrong.
- **The "Moto zombie service" never existed.** A `logcat -G 64M` broke logcat's reader;
  missing log lines were misread as a dead app. It cost a force-stop, three re-enables and
  a phone reboot before a screenshot showed the app had been scrolling throughout.
- The `NOTES.md` Moto block is flagged **HISTORICAL** — measured under a different band,
  age cap and slop margin than the current code.

## Design decisions that look like oversights and are not

Each has a `REJECTED DESIGN` block in `ScrollEngine.kt` with the evidence attached.

- **Never catch up for dead time after a re-grip.** The velocity discontinuity is what
  readers notice, not the pause — measured: same ~460ms pause, opposite verdict, purely
  from changing the restart profile.
- **Stop on *every* cancellation.** Do not add window-event disambiguation to tell a touch
  from an app switch; that requires window-state events, which forfeits `eventTypes = 0`.
  Touch-to-stop is the safety property, and it works: a finger cancels in ~10ms, with zero
  spurious cancellations in 9 minutes hands-off.
- **`canRetrieveWindowContent="false"`, `eventTypes = 0`, no INTERNET** with
  `tools:node="remove"`. OS-enforced, verified on both devices via `capabilities=32`. Do
  not weaken these — the product's whole privacy claim rests on them.
- **`yPos` is an unrounded `Double`.** Per-tick rounding was the original stall bug.
- **Segment default stays 100ms.** A blind A/B against 250ms returned a null, and a flat
  raise is the wrong shape of fix — deriving duration from `MIN_SEGMENT_PX` floor clearance
  is the recorded production recommendation.

## The most transferable output

A method rule, learned six times over: **before believing a result, ask what a null
instrument would have produced.**

| Instrument | Looked like | Actually |
|---|---|---|
| `adb shell input tap` | touches don't cancel | synthesised via InputManager, never reaches the digitiser |
| `am start` to launcher | app switches don't cancel | changes the window *without* a touch, and only the touch cancels |
| Plain A/B, one comparison | 250ms segments are smoother | unreplicated; the A/B/A control showed the discrimination didn't hold |
| A primed observer across sessions | later sessions felt worse | sensitisation — the observer got more critical, not the setting worse |
| `logcat -G 64M` | the app is dead | the log reader silently stopped returning data |
| A recorded buffer size, never checked | the capture was complete | the device refused that size; the check was written, not run |

Verify the instrument **before** the measurement, not after. Use a positive control from
the real modality, and repeat the baseline.

## The headline subjective result

**The re-grip is invisible to an unprimed reader on good hardware.** A naive reader
(Moto G54, PDF with figures, 12 dp/s) reported the velocity comfortable to read and did not
mention any bump, or losing their place, unprompted. The primed observer on the same device
and content class had called it "noticeable" — so this is independent support for the
sensitisation seen in the segment A/B/A.

Carry the caveats with the number: **n = 1, and a 2-minute session giving a derived — not
logged — count of ~3–4 re-grips** (travel binds every ~32.5s at 12 dp/s on this device).
An unprompted null is only evidence of absence if there were events to miss, so the honest
claim is *"no disturbance reported across ~3–4 re-grips"*. Real, but thin.

**Treat the Moto result as supporting evidence, not the citable one.** The tablet session
is planned at 5 minutes with logging, which gives ~9 re-grips on that device and a counted
rather than derived figure. That becomes the result to quote.

## The single most important finding: the engine optimised the wrong artifact

Four reader sessions, 9 re-grips in every one. The discrete re-grip "jump" was noticed in
one of three sessions, described as harmless, and **never** linked to fatigue. What did
track fatigue, in perfect rank order, was a *continuous* artifact the reader called "a small
interference" — jitter, whose perceived cost scales with text size.

Two mechanically matched sessions prove the scaling: 9 re-grips each, 20% vs 19% jitter
outliers, same app, content and duration, differing only in font scale — interference at the
small size, none at the large.

**The engine was built, tuned and measured around making the re-grip invisible.** The lead-in
profile, the margin sweep, the catch-up invariant, ~431ms of dead time characterised to a
12ms spread. That work is correct. It optimised the artifact that does not matter.

**The obvious follow-up was tested and came back ambiguous, not negative.** 250ms segments
cut relative jitter 2.7x with re-grips unchanged. The reader could not distinguish the
sessions — but that comparison was *serial*, judged against a memory from half an hour and
one session earlier, on their fourth repetition. Their fatigue, the only measure not
requiring recall, improved despite being later in the day.

**Serial self-comparison is itself a null instrument for subtle continuous differences.**
Settle it between-subjects: fresh readers, one condition each, no memory or habituation
involved.

Net: the re-grip is confirmed unimportant, the continuous artifact is real and tracks text
size, its relationship to jitter is unresolved, and segment duration stays at 100ms pending
a design that can actually answer the question.

**The reader is now exhausted as an instrument** — four sessions, heavily primed, reports
converging on "it's fine". Anything further needs someone fresh.

## The result that reopened the question

A 6-minute session on the **noisy** device, 9 counted re-grips, returned a *negative*
unprompted verdict: *"after a while there seemed to be a very small interference kind of the
text getting stuck for a brief moment and that made me feel tired."* Same reader, same
content class, who had been clean on the Moto at 2 minutes.

**Fatigue is a separate axis from visibility, and nothing in this repo measures it.** The
artifact is small enough to read through and still accumulates a cost. Do not treat the
earlier "invisible to an unprimed reader" line as settling the question — it was 2 minutes,
and this effect is explicitly time-dependent.

**Three confounds, so do not carry "the artifact causes fatigue" forward as a finding.**
Device (noisy vs smooth), duration (6 min vs 2, and the effect is time-dependent), and —
raised unprompted by the reader — **the font was smaller**. The artifact itself was never
identified: 9 discrete ~431ms re-grips, or continuous jitter at 36% outliers.

The font point may be the whole thing. 12 dp/s is a *distance* rate; reading demand is a
*line* rate, and they are related by line height. Smaller text at the same dp/s asks the
reader to read faster. That is also the second independent sign that **one dp/s setting
cannot serve varying content** — the first was images having no reading rate at all.

NOTES.md carries the three tests that separate the confounds. **Do not tune anything until
the artifact is identified** — the lead-in lever fixes the re-grip restart and does nothing
for jitter, and neither touches a speed-model gap.

## Still open — needs a person, not an instrument

**The tablet sessions were run** — see the sections above. What remains is not device
coverage:

- **A second reader.** Every perceptual result in this repo comes from one person across
  four sessions, who was naive at the start and heavily primed by the end. That is the
  binding limitation on all of it, and no amount of further sessions with the same reader
  fixes it.
- **A between-subjects design** for anything comparative. Serial self-comparison could not
  resolve the jitter question and will not resolve the next one either.
- The visual-check list on the tablet: long-press at margin 0, selection UI, overscroll on
  lift. Never run.

All are runnable with adb starting the scroll, which sidesteps the control surface entirely.

## A design rule that outranks the control-surface findings

**No control may require sustained input.** Sustained contact is exactly the effort this
app exists to remove; weakness, tremor or spasticity can make holding anything steady for
several seconds impossible.

This has now been reached twice independently — once on the edge slider, once on
fast-forward. Both times the natural design was a held control, and both times it was wrong
for the target population. It applies to anything the control-surface spike proposes.
Latching or discrete actions satisfy it; momentary hold does not.

Related open question from the same naive reader: **a single speed does not serve mixed
media.** Images have no reading rate, so a 300dp figure at 12 dp/s takes 25s to pass with
nothing to read. Fast-forward as a *transient* is nearly free in the engine; as a *setting*
it is expensive and reintroduces continuous control. Whether it should latch or skip is a
beta question, and an informant outside the target population cannot settle it. See README.

Observer-recording rules are in `NOTES.md` stage 4 and must hold from the first participant
session: pseudonymous IDs, no health or demographic data in the repo, quotes only with the
person's knowledge. Git history is permanent.

## Practical gotchas

- **Android Studio reported successful installs that never landed**, on both devices.
  Check `lastUpdateTime` in `dumpsys package dev.spike.autoscroll`.
- **`dumpsys accessibility` is not a liveness check.** It reported "bound" on the SM-P200
  with no process at all. Use `pidof` plus the `SVC connected` log line.
- **Force-stop silently disables the accessibility service**, clearing both
  `enabled_accessibility_services` and `accessibility_enabled`.
- **Raise the log buffer to `-G 16M`, not more**, and verify with `logcat -g`. 64M broke
  the reader on the Moto.
- There is no Gradle wrapper in the tree — open in Android Studio once to materialise it.
