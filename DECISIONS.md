# Decisions

Product and design decisions with the reasoning that produced them, the rules that outrank
individual decisions, and the open items with what would settle each.

Assembled from the spike's working documents so the reasoning survives the throwaway repo.
The code is disposable; this is not.

---

## Standing rules

These outrank individual decisions. A proposal that violates one is wrong regardless of how
good it looks in isolation.

### 1. No control may require sustained input

Sustained contact is precisely the effort this app exists to remove. Weakness, tremor or
spasticity can make holding anything steady for several seconds impossible.

Reached twice independently — once on an edge slider, once on fast-forward, where the
natural design is hold-to-boost. Both times the obvious answer was a held control, and both
times it was what an able-bodied person reaches for. Latching (tap on, tap off) or discrete
fixed-distance actions satisfy the rule; momentary hold does not.

### 2. Visibility was the wrong success criterion. Fatigue is the right one.

The engine was built, tuned and measured to make the re-grip **invisible** — the lead-in
profile, the margin sweep, the catch-up invariant, ~431ms of dead time characterised to a
12ms spread. That work is correct, and it optimised an artifact that does not matter.

Across four reader sessions the discrete re-grip was noticed in one, described as harmless,
and **never** linked to fatigue. What tracked fatigue, in perfect rank order, was a
*continuous* artifact nobody was optimising.

The lesson generalises past this spike: **a criterion that is easy to measure will get
optimised whether or not it is the one that matters.** Re-grip visibility was measurable to
the millisecond, so it received all the attention. Fatigue is slow, subjective and needs a
person, so it was never a target until it appeared by accident.

For any future spike: write down in advance what would make the *success criterion itself*
wrong, and check that as deliberately as the criterion.

### 3. No design decision may be made against a single reader

One user, sensitised by repetition, generates **hypotheses for a beta — not inputs to a
build**. Every perceptual result in this project comes from one person across four sessions.

The failure mode is specific and was observed here: by the fourth session their reports had
converged toward "it's fine", and a serial comparison against a memory from half an hour
earlier could not resolve a 2.7x mechanical difference. Sensitisation and habituation act in
opposite directions and both corrupt the comparison.

Anything comparative needs **between-subjects**: fresh readers, one condition each.

### 4. The privacy properties are structural, not conventional

`canRetrieveWindowContent="false"`, `eventTypes = 0`, and no INTERNET with
`tools:node="remove"`. The OS enforces all three; verified on both test devices via
`dumpsys accessibility` reporting `capabilities=32` — gestures only.

This is the project's strongest verifiable claim and its main trust asset. Anything that
raises that bitmask spends it. `CAPABILITY_CAN_REQUEST_FILTER_KEY_EVENTS` would move it from
32 to 40 and was rejected on those grounds alone.

### 5. Before believing a result, ask what a null instrument would have produced

Six measurements in this spike returned clean-looking data about things the instrument could
not see. Verify the instrument *before* the measurement, use a positive control from the
real modality, and repeat the baseline.

| Instrument | Looked like | Actually |
|---|---|---|
| `adb shell input tap` | touches don't cancel | synthesised via InputManager; never reaches the digitiser |
| `am start` to launcher | app switches don't cancel | changes the window *without* a touch, and only the touch cancels |
| Plain A/B, one comparison | 250ms segments are smoother | unreplicated; A/B/A showed the discrimination didn't hold |
| A primed observer across sessions | later sessions felt worse | sensitisation, not the setting |
| `logcat -G 64M` | the app is dead | the log reader silently stopped returning data |
| The `VISIBLE LURCH` log line | the lead-in is an ease-in | measured a tenth of the effect; the ramp carries ~90% |

### 6. The app may not initiate, plan, or sequence anything

Play's Accessibility API policy, effective 28 January 2026, excludes apps that autonomously
initiate, plan and execute actions or decisions. Automation uses must serve a narrow, clearly
understood purpose. Deterministic, rule-based behaviour following a static human-defined
script is explicitly *not* prohibited, and verified `isAccessibilityTool="true"` apps are
exempt where the functionality serves the disability-support purpose.

*Source as supplied, checked 12 Aug 2026: Use of the AccessibilityService API,
support.google.com/googleplay/android-developer/answer/10964491. Recorded as cited — not
independently verified in this session.*

**The engine satisfies this today by construction.** One fixed gesture, started and stopped
by the user, no branching. `eventTypes = 0` means it cannot observe state, so it cannot
decide on it. **Rule 4 is what enforces rule 6** — weakening the capabilities bitmask would
make violations of this rule possible for the first time.

**The exposure is in the control surface, not the engine.** Disqualifying shapes, all of
which are natural designs someone will propose:

- **Queued or sequenced actions.** That is a macro.
- **Any start that is not a user action in that session** — auto-start on foreground,
  scheduled start, start-on-boot.
- **Auto-resume after an interruption.** The restart problem is real: a touch to dismiss an
  ad stops the scroll, and the user may not be able to touch again. "Just resume once the ad
  is gone" is the obvious fix. It requires observing that the ad is gone, which breaks rule
  4, and it is autonomous initiation, which breaks this one. **Restart must be a user
  action.**
- **Deriving speed from anything the app inferred rather than the user set** — already
  rejected separately for `fontScale`.

---

## Threat to the v1 premise: one speed setting cannot serve varying content

**This is not a tuning problem and not a nice-to-have. It challenges the set-and-forget
premise directly.**

`dp/s` is a **distance** rate. Reading demand is a **line** rate. The two are related by
line height — and **the app cannot know line height.** Not through poor design: because
knowing it would require reading the screen, which is the one property that cannot be traded
away (standing rule 4).

So the same setting means different reading speeds in different documents, and the user
cannot know in advance which they will get. Two independent observations from the naive
reader, both content-driven and ability-independent:

- **Images have no reading rate at all.** A 300dp figure at 12 dp/s takes 25 seconds to pass
  with nothing to read.
- **Font size changes the line rate at fixed dp/s.** Smaller text means more lines per
  second for the same scroll speed.

**Both point the same way: users likely need a speed *control*, not just a speed *setting*.**
That moves adjustment from a 1.1 nice-to-have toward a **v1 requirement**, and expands the
control-surface budget from two intents (start, stop) to four (start, stop, faster, slower).

**Any such control must satisfy standing rule 1: discrete latching steps.** Not a continuous
slider, not sustained input. The obvious designs — hold-to-boost, a drag slider — are exactly
what an able-bodied person reaches for and exactly what this audience may be unable to
operate.

### REJECTED: deriving speed from `Resources.getConfiguration().fontScale`

Considered as a cheap partial mitigation, and rejected. Recorded so nobody re-proposes it.

Two rationales were offered and they pointed opposite ways, which is what prompted the
scrutiny that killed the idea:

- **Line-rate normalisation** — a *mechanism*, from arithmetic:
  `dp/s = linesPerSecond × lineHeight`, so holding reading demand constant means dp/s scales
  **up** with line height. This is the effect the reader sessions actually observed.
- **User-preference proxy** — a *correlation*: people who choose large system text may want
  to read more slowly, so **down**. Plausible, unevidenced, never tested.

They are not the same kind of claim, and they compose rather than compete —
`dp/s ∝ lineHeight × desiredLinesPerSecond`, with normalisation setting the first term and
preference the second. If both were real they would partially cancel, a third outcome
neither rationale predicted.

**What sinks it regardless: `fontScale` does not reach the content.** It is the system
text-size preference and only affects apps that lay out in `sp`. A PDF ignores it entirely;
so does browser zoom, and so does a site's own CSS. All four surfaces were encountered during
testing. As a signal for auto-deriving speed it is a poor proxy for the thing that matters.

> **Precision worth preserving:** the *controlled* text-size test (sessions 1 and 2) was
> Wikipedia in **Chrome**, where `fontScale` does apply — the scale was set through it and the
> screen change verified. That result stands and is the best-supported perceptual evidence in
> this repo. It was the *originating* observation, in a PDF viewer, where the size difference
> came from the document rather than `fontScale`. Rejecting the idea does not weaken the
> finding.

**And auto-derivation is the wrong shape of answer anyway.** It would move the scroll speed
for reasons the user cannot see — precisely the behaviour that erodes trust in an assistive
tool. The threat-to-premise section above already argues for user-facing discrete speed
steps, which solve the same problem directly, observably, and under the user's control.

### Explicitly not doing: another font-size test with the existing reader

Four sessions in, heavily primed, reports converging on "it's fine". **The instrument is
exhausted**, and another session would risk a false positive more than it would produce a
result. The question moves to the beta.

## Decisions

### `isAccessibilityTool="true"` is a functional requirement, not a submission tactic

Setting the flag routes the Play declaration to the accessibility-tool branch, which asks
whether the app genuinely serves disabled users rather than whether it is dangerous, and
which carries the exemption from the autonomy prohibition in standing rule 6.

Beyond submission: Android has been testing an Advanced Protection Mode change (Canary, early
2026) that blocks accessibility permission grants to apps without the flag and revokes it from
installed ones. If it ships, **the flag is what makes the app installable for its users at
all**.

*Both claims recorded as supplied and dated 12 Aug 2026; not independently verified in this
session. Re-check before relying on either.*

**State of this repo:**

- **The flag IS present**, in `app/src/main/res/xml/autoscroll_service.xml`, with the
  reasoning inline. Added 2026-08-12 after closure — the one code change made to a closed
  spike, because these documents are shared across spikes and a decision recorded as a
  functional requirement should not be contradicted by the only implementation anyone can
  look at. API 30+; older platforms ignore it, so it is safe against `minSdk 26`.
- **There is still no CI ratchet.** This repo has no CI configuration and no test sources —
  tests were an explicit non-goal. The ratchet described below is a thing to **build**, not a
  thing to extend.

### TODO for the production repo: build the declaration ratchet

**Not built. Does not belong in this repo** — spike 1 has no CI, no test sources, and is
closed. This is a task for whichever repo ships.

A *ratchet* is a build check that makes a specific loss impossible to merge silently. It does
not test that the app works; it fails the build if an established declaration loosens.
Weakening it then requires editing the check itself, which appears in review as a deliberate
act rather than a diff nobody noticed. Each of the three declarations below is a one-line
change that looks harmless and spends something expensive.

**Read from the built artifact, never the source.** Demonstrated on this repo 2026-08-12:
the source contained `isAccessibilityTool="true"` while the APK did not, because the APK
predated the edit — a source check would have passed on a build that shipped without it.
Source grepping fails in both directions: it matches the attribute named in a comment, and
it misses anything a dependency merges into the manifest.

| Assert | Read from | How |
|---|---|---|
| `isAccessibilityTool="true"` present | `res/xml/autoscroll_service.xml` inside the APK | `aapt2 dump xmltree <apk> --file res/xml/autoscroll_service.xml` |
| `canPerformGestures=true`, `canRetrieveWindowContent=false`, and the absence of `canRequestFilterKeyEvents`, touch exploration and magnification | same file, same command | these are the *inputs* that produce `capabilities=32` |
| No `INTERNET` in the **merged** manifest | APK badging | `aapt2 dump badging <apk>` — merged, so a library's manifest can introduce one that `tools:node="remove"` in the app manifest does not catch |

**Correction worth carrying:** earlier notes said "assert `capabilities == 32` from the built
artifact". The bitmask is computed by the OS at runtime and is not in the APK. From the
artifact you assert the attribute set that yields it. Asserting the literal `32` needs a
device — `dumpsys accessibility` — which is an instrumented test requiring hardware in CI,
worth having eventually because it verifies the OS agrees with your reading of the
attributes.

**What it will not catch.** Declarations only. It would not notice the engine reaching the
node tree by some other route, or a control path letting a third party start the scroll.
Those need review and tests, not a ratchet.

### Distribution: Play as the primary channel — PAUSED

Started as reasoning, became observation. Three distinct blocks were hit sideloading onto
two devices: Android 13+ restricted settings (Moto), MIUI's "Install via USB" (Poco), and
Play Protect's verifier (Poco, stacked on the MIUI block). Each presents differently, each
needs a different remedy in a different place, none is discoverable.

**Scope it correctly:** these are *sideload* defences. A Play build faces none of them. The
finding sharpens the channel choice and says nothing about the in-app permission-grant flow.

Counterpart risk, on the developer rather than the user: Play scrutinises
`AccessibilityService` use. Needs the `isAccessibilityTool` declaration and a
prominent-disclosure statement — a per-release review risk, not a per-user wall.

**PAUSED as of 2026-08-12: the Play track waits until spike 2 delivers a control surface.**
The reasoning above still holds; the submission simply has nothing to submit until the app
has a way for a user to start and stop it that is not adb. Standing rule 6 also means the
control surface must be designed against the autonomy prohibition from the start, rather
than reviewed against it afterwards.

### Engine: never catch up for dead time

A re-grip costs ~460ms of no motion. Repaying that debt with a burst afterwards is rejected.

The evidence is direct: changing only the restart profile, with the pause length unchanged,
flipped an observer's verdict from "noticeable... could bother a different type of user" to
"an overall improvement". The pause was never the problem; the velocity discontinuity after
it was. Catching up would reintroduce that discontinuity on top of a lead-in that already
overshoots.

### Engine: stop on every cancellation

Every cancelled gesture ends the run, whatever caused it. Disambiguating a user touch from a
window change requires window-state events, which carry the foreground package name and
forfeit `eventTypes = 0` — trading a structural guarantee for a convenience.

This turned out to be a feature rather than a compromise. Touch-to-stop works on every
device, in every app, with no permissions and no UI: a physical finger cancels in ~10ms, and
a hands-off soak produced zero spurious cancellations in 9 minutes. Notifications, shade
peeks, volume overlays and screenshots provoke nothing.

The cost lands on *restart*, not stop — a user who cannot reliably touch the screen needs a
hands-free way back in.

### Engine: segment duration stays 100ms — REOPENED

**The null that justified this was measured on the wrong outcome.** The A/B/A tested
*discrimination over ~90 seconds*; the outcome that turned out to matter is *fatigue over
minutes*, which that design could not detect. The null stands for what it measured and says
nothing about the question now on the table.

Reader work since shows fatigue tracks continuous jitter rather than the re-grip, and the
segment sweep already established that jitter is fixed-cost and therefore dilutes with
duration: 12.6% of segment at 100ms, 5.3% at 250ms. That makes segment duration the tuning
lever that plausibly matters, on grounds unrelated to the original argument.

**Test run; prediction not confirmed, so the default stays at 100ms on two independent
grounds now.** 250ms cut relative jitter 2.7x (11.8% → 4.3% of segment) with re-grips
unchanged, and the reader reported *"not a significant difference that they can spot between
tests"* — plus the discrete jumps became more noticeable, the trade flagged as a risk
beforehand. A verified 2.7x mechanical improvement that nobody can perceive is not a reason
to pay the stop-latency cost.

The original reasoning below still applies to *how* it should be set if it ever changes.

### Engine: segment duration stays 100ms (original reasoning)

A blind A/B against 250ms returned a null: the observer grouped the two matching sessions
apart, and re-grip dead time is flat across both settings (7ms spread), so the one artifact
they named could not have been affected by the manipulation.

A flat raise is also the wrong shape of fix. The problem it would solve — sub-pixel segments
hitting the `MIN_SEGMENT_PX` floor — is speed- and density-dependent, binding only below
~5 dp/s on a 2.0-density screen. **Production recommendation: derive segment duration from
floor clearance**, shortest duration commanding ≥3× `MIN_SEGMENT_PX` at the current speed and
density. Keeps stop latency minimal where it can be and generalises to untested densities.

### Engine: 25%–75% band

Widened to 12%–88% mid-spike, then narrowed back when both halves of the theory failed.
Travel stopped being the binding constraint, and at 88% the finger landed directly on a
site's sticky footer — a hazard the narrower band avoids. Above ~20 dp/s travel binds again
and the narrow band costs re-grip frequency; that trade is accepted because reading speeds
are the point.

### Control surface: the receiver must not ship exported

`RECEIVER_EXPORTED` means any installed app can start and stop the scroll, with no
permission and no user interaction.

Planned fix: channel off by default plus a per-install pairing token. Rejected alternatives:
a signature-level permission (breaks third-party switch-access and AAC apps, exactly the
integrations this audience depends on) and routing control through key-event filtering
(raises the capabilities bitmask — see standing rule 4).

---

## How the beta should measure fatigue

Recorded for whoever designs it. Fatigue is now the success criterion (standing rule 2) and
this project has no instrument for it.

### Self-report is a weak instrument for fatigue

It requires recall, and it is **exactly what sensitised the existing reader**. Asking someone
whether they feel tired makes them attend to tiredness, which changes both the experience and
the next report. Four sessions produced converging, less useful answers.

Behavioural proxies are better, and are recordable without asking anyone to introspect:

- **How long a participant voluntarily keeps reading.** A session they end early is a
  stronger fatigue signal than a session they rate as tiring.
- **Whether they come back.** Return rate across days is the closest available proxy for the
  outcome that actually matters, and it requires no questions at all.

Both are closer to the real outcome than any verbal report, and neither sensitises.

### The Play testing requirement may be the instrument, not an obstacle

If a closed test with 12 testers over 14 continuous days applies, that is **precisely the
between-subjects, fresh-reader, longitudinal cohort** this question needs — the design named
in HANDOVER.md as the only one that can settle the comparative questions.

**Assign conditions across participants rather than within.** One condition per tester, never
a sequence. That removes memory, habituation, sensitisation and order effects in a single
move — every one of which corrupted the sessions run here.

What looks like a scheduling obstacle is the study design. Plan the conditions before the
closed test opens, not after, because a cohort can only be assigned once.

## Retractions

Findings recorded, then disproved. A later reader cannot re-run these and needs to know
which held up.

| Retracted claim | What was actually true |
|---|---|
| Chain-age decay: chains degrade past ~40s | Sub-pixel segments from the decel ramp. Pre-fix bounds (36s good / 48s bad) are superseded and not comparable |
| The band should be 12%–88% | Wrong in both directions; back to 25%–75% |
| App switches cancel the stroke | Inferred from a symptom. The *touch* that performs the switch cancels; a window change alone does not |
| A "zombie service" on the Moto | The app was working throughout; `logcat -G 64M` had broken the log reader |
| The lead-in is 0.7× cruise, an ease-in | The metric measured only the margin term. Corrected: 2.55× at margin 1, with the ramp carrying ~90% |
| Null `contentIntent` explains the slow start | A real defect, but not the cause — the developer confirmed pressing the action button |

---

## Open items, and what would settle each

| Item | What would settle it |
|---|---|
| Is the re-grip visible on noisy hardware? | The naive reader, 5 minutes on the SM-P200, logged. The Moto result came from the *smooth* device — zero jitter outliers vs 216/1000 |
| Is the ±35% instantaneous velocity variation visible as micro-stutter? | Same session; unprompted words, separate from the re-grip question |
| Does a Start-button press fire no broadcast? | Reproducible and unexplained. Three hypotheses proposed and each disproved. First item for the control-surface spike |
| **Does one dp/s setting work at all?** Two independent reader observations say no: images have no reading rate, and smaller fonts raise the line rate at fixed dp/s. The default was derived from WPM × line height, and the app cannot know line height — it cannot read the screen, by design | Font-size test: same device, same duration, same content at two text sizes. If fatigue tracks font rather than device, this is a speed-model gap, not a scroll artifact — a bigger result, not a smaller one |
| Should fast-forward latch or skip? | Beta, with target users. An able-bodied informant cannot settle it — and the obvious design violates standing rule 1 |
| Should the lead-in stretch? **Measured, not decided.** 375ms gives 0.67x mean / 2.0x peak against 120ms's 2.53x / 6.3x, at the cost of re-grip dead time growing ~440ms → ~695ms. The naive reader noticed neither the overshoot nor the pause at 120ms, so there may be nothing to buy | The same reader on both settings, blind. Note the clamp is device-dependent — 375ms here, 300ms on the Moto — so one shipped constant cannot be optimal on both |
| ~~Should `LEAD_IN_MS` stretch at low speeds?~~ superseded by the row above | **Margin is now measured as a weak lever**: its whole range is 3.96x → 2.05x cruise, and at margin 0 the ramp is 100% of the effect with a 5.6x peak. `LEAD_IN_MS` is the only lever reaching the dominant term — ~3× unused headroom against the long-press deadline, cost is re-grip dead time growing ~450ms → ~680ms. Untested |
| Host apps with vertical-gesture semantics | Untested: drag-to-dismiss, short-video feeds, video players mapping vertical drags to brightness/volume, nested-scroll and collapsing toolbars |
| What to do about touchless app switches | Incoming call, alarm, full-screen intent, deep link — the only remaining case where the finger keeps dragging in a window the user did not choose |
| **Declaration ratchet not built** — three declarations that are one-line changes to lose and expensive to re-establish | Build it in the shipping repo, reading from the built artifact. Spec in the TODO section above |
| Liveness when the service is not running | Six ways to be "enabled but not working", none detectable by the app because it is not running to notice. Must be observable from outside the service |
