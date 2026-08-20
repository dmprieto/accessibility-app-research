# Accessibility app research

Research toward an Android reading-assistance app: **continuous, slow, hands-free scrolling
in other people's apps**, for readers who cannot repeatedly swipe a screen — weakness,
tremor, spasticity, or a device mounted out of reach.

The constraint that shapes everything here is that the app must do this **without reading the
screen**. It scrolls; it does not know what it is scrolling. That is a privacy property
enforced by the operating system rather than promised by the code, and most of the design
decisions in these documents are downstream of refusing to give it up.

---

## About these notes

This repository is the working record of an **unreleased** research project — five documents,
no source code. It is public because the reasoning is more useful in the open than in a
drawer, and because the retractions are part of the record: several findings here were wrong,
are marked as wrong, and are left in place alongside whatever corrected them.

Three things follow, for anyone reading a section in isolation:

**Findings carry confidence levels, and some are weak.** The engine measurements rest on
instrumented runs across two devices and are solid. Every *perceptual* result rests on **one
reader** and is recorded as a hypothesis for a beta, not as a basis for a decision.
[HANDOVER.md](HANDOVER.md) opens with the confidence table — read it before quoting any
number from these files.

**The Play Store compliance reasoning is recorded in full**, including the alternatives that
were considered and rejected, and why. It is engineering reasoning about a policy, worked
through in the open and at length. Individual lines will misread if lifted out of it: the
conclusion in every case was to build the app so that its claims are *independently
verifiable — from a running device or from the built APK*, rather than to find a favourable
review path. The declaration the
app makes is treated throughout as a functional requirement that must stay true, not as a box
to tick. [DECISIONS.md](DECISIONS.md) carries the whole argument, and the whole argument is
the point.

**Nothing here describes a shipping product.** No build is released, no submission has been
made, and the application sources are not in this repository. Where these notes reference
source files, build steps or class names, they describe a separate working tree.

---

## How the work is organised

The project runs as **spikes**: each one asks a single question, and closes when that question
is answered rather than when the work runs out. Findings that fall outside a spike's question
get recorded and handed forward instead of chased.

| Spike | Question | Status |
|---|---|---|
| **1 — Autoscroll engine** | Can an `AccessibilityService` produce smooth, slow, continuous scrolling in arbitrary third-party apps by holding one synthetic finger down, rather than dispatching repeated swipes? | **Closed 2026-08-12 — answered yes**, on two devices five Android majors apart |
| **2 — Control surface** | How does a user start and stop it, without adb, given no control may require sustained input? | **Not started.** Spike 1 collected the evidence it should start from |
| **Beta — fatigue** | Does the scroll actually reduce reading effort, measured between-subjects on fresh readers? | **Not started.** Design recorded in [DECISIONS.md](DECISIONS.md) |

**The Play track is paused** until spike 2 delivers a control surface. There is nothing to
submit until a user can start and stop the app without a terminal.

### The documents

| File | What it carries |
|---|---|
| **[HANDOVER.md](HANDOVER.md)** | **Read this first.** Confidence levels per finding, the retractions in one table, the scope boundary, and what *not* to do next |
| **[DECISIONS.md](DECISIONS.md)** | The standing rules, product decisions with their reasoning, rejected alternatives, and open items with what would settle each. Written to outlive the code |
| **[DEVICES.md](DEVICES.md)** | One row per device, structured not prose. Built so measurements stay comparable as devices are added |
| **[NOTES.md](NOTES.md)** | Test protocols, session-by-session observations, and the raw working record |
| **[PARKED.md](PARKED.md)** | Deliberately not being worked on, with what would un-park each |
| **README.md** | This file: the programme, then spike 1 in operational detail |

One more, outside that ordering because it is about *how* the record is kept rather than what
the research found: **[RECORD-KEEPING.md](RECORD-KEEPING.md)** — the provenance vocabulary,
which decisions are never settled without the developer, and why publishing is a separate
authority from writing. Read it before editing any of the files above. It governs the research
phase and is meant to be deleted when that closes, apart from the part it marks as graduating.

---

## What the programme has established

**On the engine — high confidence, instrumented, two devices:**

- **Held-finger chaining works.** One `StrokeDescription` extended segment after segment
  produces smooth continuous scroll in third-party apps: a 68-second unbroken press, seams of
  1–3ms. The second device required **zero engine changes**.
- **The privacy properties hold, and are OS-enforced.** They take **two** checks, not one,
  and the distinction matters — one reads runtime OS behaviour, the other reads the built
  artifact:

  - `capabilities=32` (gestures only, no window-content access) and `eventTypes=0`, from
    `adb shell dumpsys accessibility` against a running device. The bitmask is computed by
    the OS from the service's declared attributes, so this is the OS agreeing with the
    declaration rather than the app reporting on itself.
  - **No `INTERNET` permission**, from `aapt2 dump badging <apk>` against the built APK. It
    has to be read from the *merged* manifest: a dependency can introduce the permission,
    and `tools:node="remove"` in the app manifest does not catch that.

  Verified on both devices, and checkable by anyone. See the ratchet spec in
  [DECISIONS.md](DECISIONS.md) for why the second one cannot be a source grep.
- **Touch-to-stop works.** A physical finger cancels within 12ms, with zero spurious
  cancellations across a 9-minute hands-off soak.
- **Re-grip is unavoidable and costs ~450ms**, bounded by travel rather than time.
- **Delivered speed lands within ~1% of commanded** across 4–100 dp/s after rate correction.

**On the product — this is where the programme turned:**

- **Visibility was the wrong success criterion; fatigue is the right one.** The engine was
  built, tuned and measured to make the re-grip *invisible*. That work is correct, and it
  optimised an artifact that turned out not to matter. Across four sessions the discrete
  re-grip was noticed once, called harmless, and never linked to fatigue. What tracked
  fatigue was a *continuous* artifact nobody was optimising.

  The lesson generalises past this project: **a criterion that is easy to measure will get
  optimised whether or not it is the one that matters.**

- **A single dp/s setting cannot serve varying content.** `dp/s` is a *distance* rate;
  reading demand is a *line* rate, related by line height — and the app cannot know line
  height, because knowing it would mean reading the screen. Two independent reader
  observations converged on this: images have no reading rate at all, and smaller text at the
  same dp/s asks the reader to read faster. This is an open threat to the v1 premise, not a
  tuning problem.

- **The control surface, not the engine, is where the risk lives.** Every finding after the
  second device concerned how a user *tells* the engine what to do — and that is also where
  the entire Play-policy exposure sits, since the engine cannot observe state and therefore
  cannot decide on it.

## What the programme has *not* established

- **Every perceptual result comes from one reader**, naive at the start and heavily primed by
  the end. That is the binding limitation on all of it, and more sessions with the same
  person cannot fix it.
- **Whether jitter reduction is perceptible is unresolved, not disproven.** The comparison was
  serial and memory-based, which is itself a null instrument for subtle continuous
  differences.
- **No control surface exists.** The app cannot currently be started or stopped by its
  intended user without a terminal.

## Standing rules

These outrank individual decisions: a proposal that violates one is wrong regardless of how
good it looks in isolation. Stated in full, with the reasoning that produced each, in
[DECISIONS.md](DECISIONS.md).

1. **No control may require sustained input.** Sustained contact is exactly the effort this
   app exists to remove. Reached twice independently, from different directions.
2. **Fatigue, not visibility, is the success criterion.**
3. **No design decision may be made against a single reader.** Anything comparative needs
   between-subjects: fresh readers, one condition each.
4. **The privacy properties are structural, not conventional.** Anything that raises the
   capabilities bitmask spends the project's strongest verifiable claim.
5. **Before believing a result, ask what a null instrument would have produced.**
6. **The app may not initiate, plan, or sequence anything.** The engine satisfies this by
   construction; the exposure is entirely in the control surface.

## One method rule, learned seven times over

**Before believing a result, ask what a null instrument would have produced.** Six separate
measurements returned clean-looking data from instruments that could not measure the thing in
question:

| Instrument | Looked like | Actually |
|---|---|---|
| `adb shell input tap` | "touches don't cancel" | synthesised via InputManager, never reaches the digitiser |
| `am start` to launcher | "app switches don't cancel" | changes the window *without* a touch, and only the touch cancels |
| Plain A/B, one comparison | "250ms segments are smoother" | unreplicated; the A/B/A control showed the discrimination didn't hold |
| A primed observer across sessions | "later sessions felt worse" | sensitisation — the observer got more critical, not the setting worse |
| `logcat -G 64M` | "the app is dead" | the log reader silently stopped returning data |
| A recorded buffer size, never checked | "the capture was complete" | the device refused that size; the check was written, not run |
| The `VISIBLE LURCH` log line | "the lead-in is an ease-in" | it measured only the margin term; the ramp carries ~90% |

Each returns something that looks like data. The check is cheap: run a positive control from
the real modality *first*, repeat the baseline, and ask whether the instrument reproduces the
user action or only its side effect. Verify the instrument **before** the measurement, not
after.

---
---

# Spike 1 — autoscroll engine

**Closed 2026-08-12.** Everything below is spike 1 in operational detail: what it built, how
to run it, what it measured, and what it handed forward.

Tests one assumption: *can an AccessibilityService produce smooth, slow, continuous scrolling
in arbitrary third-party apps by holding a single synthetic "virtual finger" down and moving
it incrementally, rather than dispatching repeated discrete swipes?*

**Answer: yes.** Verified on a **Moto G54** (Android 15, phone) and a **Samsung SM-P200**
(Android 11, tablet) — five Android majors apart, two manufacturers, both form factors, both
navigation modes. Full per-device data in [DEVICES.md](DEVICES.md); protocols and subjective
notes in [NOTES.md](NOTES.md).

**Scope note: spike 1 is closed for engine work.** Everything found after device 2 landed
concerns the *control surface*, which was an explicit non-goal — see
[Handover to a control-surface spike](#handover-to-a-control-surface-spike) at the end.

## Findings

**Settled:**

- **The re-grip is invisible to an unprimed reader in a short session on good hardware** —
  2 minutes, ~3–4 re-grips, Moto G54: speed comfortable, no bump mentioned, no loss of place.

- **The re-grip is not what matters.** Four reader sessions, 9 re-grips in each: the
  discrete "jump" was noticed in one session, called harmless, and never linked to fatigue.
  **A continuous artifact — the reader's "small interference" — is what varied**, and it
  tracks **text size**, not jitter magnitude. Two mechanically matched sessions differing
  only in font scale produced interference at the small size and none at the large one.
  Cutting relative jitter 2.7× at matched text size (11.8% → 4.3% of segment, via 250ms
  segments) produced no difference the reader could *report* — but that comparison was
  serial and memory-based, on their fourth repetition of the task, and the one non-memory
  measure (fatigue) moved in the predicted direction. **Unresolved, not disproven.** See
  NOTES.md for why the instrument, not the artifact, is the limiting factor.

  The engine was built, tuned and measured around making the re-grip invisible. That work is
  correct and it optimised the artifact that does not matter.

- Held-finger chaining produces smooth continuous scroll. 68s unbroken press; seams 1–3ms.
- Re-grip costs ~450ms and is unavoidable — travel-bound, ~32s on a phone, ~39s on the tablet.
- Delivered speed within ~1% of commanded at 4–100 dp/s, after rate correction.
- Privacy properties hold on both devices: `capabilities=32`, `eventTypes=0`, no content access.
- **Touch-to-stop works.** A physical finger cancels within 12ms; zero spurious
  cancellations in 9 minutes hands-off; notifications, shade peeks, volume overlays and
  screenshots provoke nothing.

**NOT settled, and the headline open problem:**

- **Over 6 minutes on the noisy device, the same reader reported fatigue.** Unprompted:
  *"after a while there seemed to be a very small interference kind of the text getting stuck
  for a brief moment and that made me feel tired."* Speed fine, did not get lost, *"I could
  still read ok but made me more tired."* Counted: 9 re-grips, 36% jitter outliers.

  This introduces an axis nothing here measures — **fatigue, as distinct from visibility**.

  **Attribution is weak, and should not be carried forward as "the artifact causes
  fatigue."** Three confounds: the device (noisy vs smooth), the duration (6 min vs 2, and
  the effect is explicitly time-dependent), and — volunteered by the reader — **the font was
  smaller**. The artifact was never identified either: 9 discrete ~431ms pauses, or
  continuous micro-stutter at 36% outliers.

  The font point may be the whole finding. **12 dp/s is a distance rate; reading demand is a
  line rate**, related by line height. Smaller text at the same dp/s asks the reader to read
  faster, and fatigue is what a slightly-too-fast scroll produces.

- **A single dp/s setting cannot serve varying line height — the same gap the images
  feedback found.** The 12 dp/s default came from words per minute × line height, and the
  app cannot know line height because it cannot read the screen, by design. Two independent
  reader observations now point at the same missing input. See NOTES.md for the tests.

**Open, and not answerable from logs:**

- Whether the ~450ms re-grip is visible **on the noisy device**. It is now confirmed
  invisible to an unprimed reader on the Moto G54 — which is the *smooth* device, zero
  jitter outliers in 936 segments. The SM-P200 has 216/1000 outliers and ±35% instantaneous
  velocity variation and has had no eyes on it at all. Same reader, other hardware, is the
  one remaining perceptual question.
- Whether the tablet's ±35% instantaneous velocity variation is visible as micro-stutter.
- What production should do about touchless app switches — incoming call, alarm,
  full-screen intent, deep link — which are the only remaining case where the finger keeps
  dragging in a window the user did not choose.

**Known wrong turns, documented so they are not repeated:** chain-age decay (was sub-pixel
segments), the band widening (wrong in both directions), and the app-switch cancellation
claim (was the touch, not the window change). The instrument failures behind several of them
are in [the null-instrument table](#one-method-rule-learned-six-times-over) above.

---

## What it does

Opens one `StrokeDescription` with `willContinue = true` and extends it with
`continueStroke()` segment after segment. The finger starts at x = 50% width,
y = 75% height and travels upward (so content scrolls down, reading direction) to
y = 25% height, then re-grips: decelerate → hold still → lift → reposition → press →
lead-in kick → ramp → cruise.

**Travel, not duration, is what bounds a press.** Segments are sub-second, so
`GestureDescription.getMaxGestureDuration()` (60s) is never approached, and a single
press has been held unbroken for 68 seconds. The finger runs out of the 25–75% band
after 0.5 × screen height — roughly every 32s at 12 dp/s, every 4s at 100 dp/s. That
re-grip is the thing this spike exists to measure.

**Velocity is a profile, not a constant.** At 12 dp/s a finger covers ~5dp inside the
long-press window (**400ms** on the Moto G54, **500ms** on the Samsung tablet — read it at
runtime, the 500ms every reference quotes is not universal), which is under touch slop
(~8dp) — so a constant-velocity slow drag fires the host
app's long-press handler before it scrolls anything. Every press therefore opens with
a fast lead-in kick, then ramps down to target over 200ms. This costs nothing: the
host app discards the slop distance either way. `scaledTouchSlop`,
`getLongPressTimeout()` and `scaledMinimumFlingVelocity` are all read at runtime,
never hardcoded.

## What it deliberately doesn't do

- **Never touches the node tree.** The service declares
  `canRetrieveWindowContent="false"`, so `getRootInActiveWindow()` returns null no
  matter what the code asks for — the privacy property is OS-enforced, not a promise
  to remember. See the comments in `AndroidManifest.xml` and
  `res/xml/autoscroll_service.xml`.
- **Subscribes to no accessibility events.** `eventTypes = 0`, set both by omission in
  the XML and explicitly in `onServiceConnected()`. `onAccessibilityEvent()` is dead code.
- **No INTERNET permission**, asserted with `tools:node="remove"` so no future
  dependency can merge one in.
- **Declares `isAccessibilityTool="true"`.** A functional requirement rather than a
  submission tactic — it carries the exemption from Play's autonomy prohibition, and a
  reported Advanced Protection Mode change may make it necessary for the app to be grantable
  accessibility permission at all. See [DECISIONS.md](DECISIONS.md).
- No settings UI, no persistence, no sensors, no tile, no reverse scroll, no
  onboarding, no tests. One notification and one broadcast, that's the whole surface.
- **No self-calibration.** There was a subsystem tracking known-good/known-bad press
  ages with confirmation and probing; it was deleted once chain-age decay turned out
  not to exist. A fixed 60s backstop plus `CANCEL` logging replaces it.

---

## Reproducing the measurements

**The application sources are not in this repository** — this repo is the research record.
What follows is what a reader needs to interpret the numbers in [DEVICES.md](DEVICES.md) and
reproduce them against the spike's working tree. Full operating instructions belong with the
code.

The spike builds against `compileSdk`/`targetSdk` 37, `minSdk` 26, **with no dependencies
beyond the Kotlin stdlib** — a choice that began as an APK-size preference and became a
reliability requirement once LMKD kills turned out to matter (see the memory-footprint
finding below). No Gradle wrapper is committed to the source tree; opening the project in
Android Studio once materialises it.

The app has **no launcher activity and no UI**, so it is enabled and driven entirely from
adb: grant `POST_NOTIFICATIONS`, clear the Android 13+ restricted-settings block with
`appops set … ACCESS_RESTRICTED_SETTINGS allow`, then enable the service in
**Settings → Accessibility** — or write `enabled_accessibility_services` and
`accessibility_enabled` directly, which overwrites any other enabled services.

### Onboarding evidence: every device blocked the sideload somewhere

Each device presented differently, and none of these apply to a Play-installed app. This is
onboarding evidence, not a build problem:

| Device | Barrier | Presents as | Cleared by |
|---|---|---|---|
| Moto G54 / Android 15 | Restricted settings (Android 13+) | Accessibility toggle greyed out, no explanation | `appops set … ACCESS_RESTRICTED_SETTINGS allow`, or App info → ⋮ → Allow restricted settings |
| Poco F1 / MIUI 12 | MIUI "Install via USB" | `INSTALL_FAILED_USER_RESTRICTED` on adb install **and** on a manual tap, because MIUI routes any adb-delivered APK through `AdbInstallActivity` | Mi account sign-in; or deliver the APK by a non-adb route |
| Poco F1 / MIUI 12 | Play Protect verifier | `INSTALL_FAILED_VERIFICATION_FAILURE` (-22), scan "cancelled" rather than a malware verdict | Disable Play Protect scanning for the install |

Three distinct blocks across two devices, two of them stacked on the same handset so
clearing the first only revealed the second. For a user with motor impairments, a flow
that requires finding a hidden overflow menu, signing into a vendor account, or toggling
off a security feature is not friction — it is a wall.

**Scope this correctly: these are sideload defences, not install defences.** A
Play-delivered build faces none of them — restricted settings exempts Play installs, MIUI's
gate targets unknown sources, and Play Protect's verifier exists to check unverified APKs.
The finding sharpens the *channel* decision. It says nothing about the in-app
permission-grant flow, and should not be read as evidence that onboarding is hostile in
general.

The counterpart risk on the Play path is different in kind and lands on the developer
rather than the user: Play scrutinises `AccessibilityService` use, and an app using it for
anything other than accessibility gets rejected. This one qualifies genuinely, but it will
need the `isAccessibilityTool` declaration and a prominent-disclosure statement, and that
is a per-release review risk rather than a per-user wall. Worth planning for, not worth
conflating with the table above.

### The measurement surface

Every parameter is settable at runtime by broadcast, which is what made the sweeps possible
without a rebuild:

```bash
adb shell am broadcast -a dev.spike.autoscroll.CONTROL -p dev.spike.autoscroll --ei speed 25
```

**`dev.spike.autoscroll` is the spike's package name and stays.** It is not the production
identifier and must not be swept to match it: this command was run against a build carrying it.
See *applicationId and namespace* in [DECISIONS.md](DECISIONS.md).

| Extra | Sets | Swept over |
|---|---|---|
| `--es cmd start` / `stop` / `toggle` | Run control; also available from the notification | — |
| `--ei speed` | Cruise speed, dp/s | 6 / 12 / 25 / 50 / 100 |
| `--ei segment` | Segment duration, ms | 32 / 50 / 100 / 150 / 250 |
| `--ei margin` | Lead-in margin, dp — the visible lurch after every re-grip | 4 / 2 / 1 / 0 |
| `--ei leadinms` | Lead-in duration, ms — the dominant lever on re-grip smoothness. Clamped per device at 0.75× the measured long-press timeout; a request above that is honoured to the limit and logged | 120 / 200 / 250 / 300 |
| `--ei cap` | Press age cap, ms — probes a device's chain-decay threshold directly | 30000 → 120000 |
| `--ez leadin false` | Disables the lead-in kick, to observe the long-press failure mode on that device | — |
| `--ez ratecorrect false` | Disables rate correction, to reproduce the raw delivered-speed bias or A/B the smoothness | — |
| `--ez repress true` | **Measurement mode only.** Re-presses through cancellations, so touching the screen does *not* stop the scroll. Never ship it on — its removal is a recorded build-check requirement, not a thing to remember | — |

Extras combine in one broadcast. Speed and segment take effect at the next segment
boundary; `leadin` applies at the next press. Nothing needs a restart or a rebuild.

---

## Reading the logs

```bash
adb logcat -c
```

```bash
adb logcat -s AutoScroll:V
```

Every line is `EVENT  key=value ...`, so `adb logcat -s AutoScroll:V | grep REGRIP`
gets you one event type.

| Event | Says |
|---|---|
| `SVC` | Service connected. Confirms `canRetrieveWindowContent=false`, `eventTypes=0`, and the platform's actual max gesture duration. |
| `CFG` | Current speed / segment / lead-in. Logged at boot and on every broadcast. |
| `PRESS` | A finger went down. Reports screen size, density, the band in px, **travel budget in dp and seconds**, and this device's measured slop and long-press timeout. |
| `LEADIN` | The kick finished. Shows dp commanded vs dp the host app discards to slop, against the long-press deadline. |
| `TICK` | One segment completed. `target` vs `actual` ms, signed `jitter`, current velocity, and **cumulative commanded displacement in dp**. Lines over ±16ms are tagged `<-- JITTER`. |
| `SEAM` | Dead time between one segment completing and the next being dispatched — the IPC round trip that a continued stroke cannot avoid. Over 16ms is tagged `<-- SEAM`. This is the number that decides whether chaining is viable. |
| `DECEL` | Motion is being wound down, and why (`travel` = out of band, `stop` = you asked). |
| `REGRIP` | **The primary output.** `deadMs` is the full disturbance, decel through to cruise velocity in the new press. Also reports dp lost to slop on the re-press and the next press's budget in seconds. |
| `LIFT` | Finger up, with how long it was held and running press/re-grip/cancel counts. |
| `CANCEL` | One line per cancellation carrying `pressAge`, `phase`, `segElapsed` vs `segTarget`, `y` and `yFrac`, the current cap, and the full config. Cluster the `pressAge` values: a tight cluster at a consistent age, stable across speeds and apps, means the cap is wrong for that device. Diffuse ages, or correlation with `yFrac` or host app, means something else. |
| `DEVICE` | Manufacturer, model, API level, build. Logged once at connect. Copy into [DEVICES.md](DEVICES.md). |
| `ratio=` in `TICK` | Filtered actual/nominal segment duration — the rate-correction factor. Should **settle** within a couple of seconds and then sit still. A wandering ratio means there is noise rather than systematic bias in the timing, and `RATE_FILTER_ALPHA` needs to be much slower. |
| `GEST onCancelled` | A segment was cancelled — you touched the screen, another gesture pre-empted us, or the window went away. Carries the running cancellation count. The chain re-presses after 250ms, otherwise the count could never exceed 1. |
| `STOP` | Requested, then completed with measured latency (remaining segment + 100ms decel + 100ms hold). |
| `SEG` / `ABORT` | Warnings: clamped durations, degenerate sub-pixel paths, a refused dispatch. |

### What good looks like

- `SEAM gap` consistently small and consistent. A *variable* gap will read worse than a
  large steady one.
- `TICK jitter` small relative to segment duration. Expect this to get worse as segment
  duration goes down — at 32ms the IPC round trip is a large fraction of the segment.
- `REGRIP deadMs` around 350–400ms. Whether that reads as a blink or a glitch every
  ~26s is not in the logs.
- Cumulative dp advancing linearly. Stair-stepping or stalling at 6 dp/s means the
  Double accumulator got quantised somewhere.

Logs looking clean is **not** a pass. A context menu, a text-selection handle or a
tooltip appearing at press or re-grip is a fail regardless of what logcat says.

---

## Measured results — Moto G54 5G (Android 15, 1080x2400, density 3.08)

*Device 2 (Samsung SM-P200, Android 11, tablet) is in [DEVICES.md](DEVICES.md), including
where its numbers differ — notably 20x worse segment-timing jitter.*

Slop 8.1dp, long-press timeout **400ms** (not the 500ms textbook figure — read it at
runtime, always). Travel band 288..2112px = 593dp.

- **Cruise is solid.** 65 seconds of continuous held-finger scroll at 6 dp/s, 759
  segments, zero cancellations. Jitter inside ±14ms on 100ms segments, seams 1–3ms.
- **Re-grip costs ~473ms at 12 dp/s** (466 / 480 / 472) and **~580ms at 100 dp/s**
  (591 / 584 / 565 / 585). The difference is the decel ramp, which is skipped below the
  fling threshold. Both are worse than the 350–400ms estimate. At 12 dp/s with a 30s age
  cap that is ~1.6% of wall-clock time disturbed.
- **Below ~20 dp/s, age binds before travel does.** At 12 dp/s a press uses only 366dp of
  the 593dp band before `MAX_PRESS_MS` fires. Crossover on this device is
  593dp / 30s = 19.8 dp/s. Widening the band from 25%–75% to 12%–88% therefore buys
  nothing at reading speeds and only pays above ~20 dp/s — worth reconsidering, since the
  wider band is the one running closest to the gesture zones.
- **Sub-pixel segments get cancelled. Chain age does not matter.**

  This was originally recorded here as age-related decay, and that was wrong — worth
  stating plainly because the wrong version was in this file for a while. Cancellations
  appeared at 48s and 62s of chain age against clean behaviour at 36s, which looked like
  decay. It wasn't. The decel ramp at low speed commanded 0.46px and 0.69px paths, and
  long chains were merely where slow ramps produced them.

  Two fixes, both in `ScrollEngine`: the decel ramp is skipped entirely when velocity is
  already below `scaledMinimumFlingVelocity` (no fling to prevent, and the ramp was what
  generated the sub-pixel segments), and `MIN_SEGMENT_PX` floors commanded displacement
  at 1px by lengthening the segment rather than rounding the accumulator.

  Controlled probing afterwards found no decay at all: clean age-triggered teardowns at
  35 / 40 / 45 / 50s, and **a single unbroken 68.07s press** ending on travel with zero
  cancellations. `MAX_PRESS_MS` is now a 60s safety net rather than a 30s routine
  constraint — it was costing a re-grip every 30s for nothing.

- **Delivered speed does not match requested speed.** Displacement per segment is computed
  from the *nominal* duration while the loop advances at the *actual* completion time, so
  delivered = requested x (nominal / actual):

  | Requested | Delivered | Error |
  |---|---|---|
  | 4 dp/s | ~5.7 dp/s | +43% |
  | 6 dp/s | ~7.25 dp/s | +21% |
  | 12 dp/s | ~12.2 dp/s | +2% |
  | 100 dp/s | ~87.3 dp/s | −13% |

  Short paths complete early, long ones run over. The `jitter` column was never cosmetic —
  it is a direct speed bias, and it is worst exactly where this product lives. Not fixed:
  the fix is to compute displacement from measured elapsed time, which is feedback and
  sits close enough to the catch-up invariant to deserve its own before/after.

- **The re-grip restart profile matters more than the pause length.** The lead-in margin
  was originally 4dp, which moved content at 3.98x cruise across the lead-in right after the
  pause — read as a jump. At 1dp it is 2.55x, and the same observer on the
  same document called it "an overall improvement." The pause itself was unchanged at
  ~460ms. Default is now 1dp.
- **The lead-in kick is not what clears touch slop** — kick + ramp together are. They
  cover ~16dp by 320ms against 8.1dp of slop, so no long-press fired at *any* margin
  including 0, in both a WebView article and a PDF preview.
- **At y = 88% the finger can land on a sticky footer.** On a Vanity Fair article it
  landed on the subscription CTA. It scrolled anyway, but an element that consumes its own
  touches would swallow the drag.
- **Starting from the notification scrolls the notification shade, not the app.**
  Reproduced on the SM-P200 and independently observed on the Moto G54, so this is platform
  behaviour rather than an OEM quirk. With the shade open, `mCurrentFocus` stays
  `NotificationShade` for the whole run — 86 ticks, no cancel — and the finger drags the
  shade.

  Cause: a notification **action button** does not collapse the shade; only tapping the
  notification body does. So the shade is still topmost when the press lands, and the
  finger goes wherever the topmost window is.

  **This is the same root behaviour as the finger following you into another app**, and it
  generalises past the notification: *any control surface that is itself a window will
  capture the finger*. A quick-settings tile has the identical problem, because the QS panel
  is a window. Only Stop is safe — stopping places no finger.

  It is also an argument for the planned proximity-wave control that was not part of the
  original reasoning: it is the only planned control with no panel of its own, so it is the
  only one immune to this **where the hardware provides it**.

  **Narrowed 2026-08-13.** This originally read "the only one immune by construction",
  which was true of the mechanism and false as a statement about availability: proximity is
  absent in hardware on the SM-P200, so on tablet-class devices there is no immune control
  at all. See DECISIONS.md.

  **Fixed.** A start dismisses system panels first, then waits `PANEL_COLLAPSE_MS` (500ms)
  for the collapse animation before pressing.
  `performGlobalAction(GLOBAL_ACTION_DISMISS_NOTIFICATION_SHADE)` from API 31, falling back
  to `GLOBAL_ACTION_BACK` below it. `performGlobalAction` needs no window content, so
  `eventTypes = 0` and `canRetrieveWindowContent="false"` are untouched.

  **The dismissal is gated on the start coming from the notification**, because that is the
  only path where a panel is open by construction. On API < 31 the fallback is BACK, and
  firing BACK with no panel open would send it to the foreground app and navigate away or
  close the document — a worse bug than the one being fixed. An adb start therefore presses
  immediately, with no dismissal and no delay, which also leaves the measurement harness
  timing unchanged.

  **Verified on the SM-P200 (API 30, `GLOBAL_ACTION_BACK` path):**
  `CTRL start requested (notification) - dismissing system panels first (action=BACK
  accepted=true), pressing in 500ms`, followed by a clean 30.9s press. The finger landed on
  the app rather than the shade, confirmed by the developer.

  **Verified on the Moto G54 (API 35, `GLOBAL_ACTION_DISMISS_NOTIFICATION_SHADE` path):**
  `action=DISMISS_SHADE accepted=true`, then 3 presses / 2 re-grips / 907dp with the finger
  on the app. Both branches of the fix are now exercised on the device each applies to.

- **Tapping the notification body does nothing — `contentIntent` is null.** Only the small
  "Start" action button is live. A tap on the body is swallowed with no broadcast, no log
  and no feedback of any kind; `dumpsys notification` shows `contentIntent=null` where every
  other app's notification has one.

  **This is NOT the explanation for the "takes near a minute to start" report** — that was
  proposed here and then ruled out: the developer confirmed pressing the action button, not
  the body. The defect is real and verified from `dumpsys`; the causal claim was wrong.

  **Still unexplained:** a confirmed press of the Start *button* produced no
  `Broadcasting: Intent` line from ActivityManager at all, in a log window verified to cover
  the press (16:00:59–16:01:43, 1845 lines, nothing rolled). So the action button's
  PendingIntent did not fire either. Open question, not chased further — see below.

  **For this audience the target sizes are exactly backwards.** The whole notification is a
  large, easy target that is inert, and the only working control is a small button requiring
  precision. Someone with a tremor or limited dexterity will hit the dead area repeatedly
  and conclude the app is broken — which is precisely what happened to the developer.

  Fix (not implemented, control-surface design): set a `contentIntent` so the body toggles
  start/stop, making the entire notification the target. Costs nothing and removes the
  precision requirement.

- **The 500ms delay reads as failure, because nothing acknowledges the press.** On first
  use the developer pressed Start four times and reported "it closes the menu but it doesn't
  scroll" — the log shows four `start requested` lines and a perfectly healthy run.

  From pressing Start: ~500ms of nothing while the panel collapses, then the press lands,
  then the host app discards the first ~8dp to touch slop, then motion begins at 12 dp/s —
  which is deliberately subtle. First unambiguous movement is close to a second after the
  button, with zero feedback in between.

  **This is the only hands-free control the app has**, and the person who built it concluded
  it was broken. A user who cannot easily press the button again has less patience and far
  less context.

  The delay is doing real work — it is what stops the finger landing on the shade — so the
  fix is feedback during it, not removal of it. Cheapest version: flip the notification to a
  "starting" state the instant the press is received, before the dismiss. Not implemented;
  it is control-surface design rather than engine work.

- **"Enabled but not running" has at least three distinct causes, and they are
  indistinguishable from outside.** All three present the same way: Settings shows the app
  enabled, nothing scrolls, and the app cannot report the problem because it is not running
  to notice it.

  | Cause | State it leaves behind | Observed |
  |---|---|---|
  | Master accessibility toggle off | `enabled_accessibility_services` still lists the service, but `accessibility_enabled = 0` | SM-P200, cause unknown |
  | Package in the stopped state | Both settings correct, `dumpsys accessibility` lists it under *bound services*, but no process exists | SM-P200, after every reinstall |
  | Killed by LMKD | Settings unchanged, ticks simply stop mid-run | SM-P200, 2.8GB device under memory pressure |

  Note the second one specifically: `dumpsys accessibility` reported the service under both
  "enabled services" and "bound services" while `pidof` returned nothing. **The framework's
  own bookkeeping said bound when nothing was running**, so that dumpsys field is not a
  reliable liveness check — use `pidof`, or the `SVC connected` log line.

  Recovery for the stopped state is cycling the Settings toggle; writing the secure settings
  over adb does not rebind on One UI. There is no in-app recovery for any of the three,
  because none of them leave anything of ours running.

  For production this argues for liveness being observable from *outside* the service —
  something that notices its absence — rather than the service reporting its own health,
  which is impossible in all three cases.

- **RETRACTED: the "Moto G54 zombie service" did not exist.** For roughly half an hour the
  service appeared enabled, bound, and alive while logging nothing and answering no
  broadcasts. It was diagnosed as a zombie, force-stopped, re-enabled three times, and the
  phone was rebooted.

  **The app was working the entire time.** A before/after screenshot showed the screen
  scrolling normally. What had failed was `adb logcat`: an earlier `logcat -G 64M` — added
  to stop long runs losing telemetry to buffer roll-over — left the reader returning almost
  nothing (`64 MiB, 16 MiB consumed, 2 MiB readable`). Resetting to `-G 256K` restored every
  line immediately.

  So a fix for one instrument silently disabled another, and the absence of log lines was
  read as absence of behaviour. **Never infer app state from missing logs alone** — confirm
  with something outside the logging path, which here was one screenshot comparison.

  Genuinely established during that episode, and unaffected by the retraction, because both
  were read from settings values rather than logs:

  - **Force-stop silently disables the accessibility service.** It cleared both
    `enabled_accessibility_services` and `accessibility_enabled`, requiring a manual
    re-enable. OEM battery managers and task killers force-stop background apps routinely,
    so this is a live path to the app quietly ceasing to work.
  - **`dumpsys accessibility` is not a liveness check.** It reported the service as bound on
    the SM-P200 when no process existed at all.

- **Silent process death is the failure mode with no log line.** On the SM-P200 (2.8GB
  RAM, ~880MB free) the low-memory killer was observed thrashing and killing a Samsung
  *system* service while Play Store updated. Nothing prevents it — a doze whitelist does
  not help, and an accessibility service is not exempt. If it happens, the scroll stops
  with no `CANCEL`, no `STOP`, nothing on screen, and no way for the user to tell it apart
  from the app deciding to quit. For someone who cannot easily reach the device to restart
  it, that is worse than a visible error.

  **This turns the memory-footprint budget into a reliability requirement, not a courtesy
  one.** LMKD kills by an OOM-adjusted score in which resident size matters: the smaller
  the footprint, the further down the kill list. That is now the strongest argument for
  the Views-not-Compose and zero-dependency choices — stronger than the APK-size reasoning
  they were originally made for, because it converts a size preference into an
  availability property on exactly the cheap hardware this audience is most likely to own.

  Measured baseline: **31MB total PSS** for this spike, against 200MB for Play Services
  and 209MB for `system` on the same device. Worth treating as a budget to defend.

- **A window change alone does not cancel the stroke — the finger stays put and keeps
  dragging in the new window.** Measured on the SM-P200: ticks continued 79 → 149 across an
  `am start` to the launcher, with the stroke still cruising afterwards.

  **But a user-initiated app switch DOES cancel, and that is what matters.** Measured on
  the SM-P200: pressing the physical home button cancelled at `segElapsed=9ms`, before the
  launcher even came forward. A real switch is a finger on the digitiser — a gesture-nav
  home swipe or a software home button — and fingers cancel in ~10ms.

  So the adb result and the user result differ because `am start` changes the window
  *without* a touch, and only the touch cancels. The hazard is therefore confined to
  **touchless** switches: incoming call, alarm, full-screen intent, deep link fired by
  another app. Those are a limitations-page entry, not a design gap.

  Worth keeping as a methodology lesson: this was asserted wrongly once (as "app switch
  cancels", inferred from a symptom), then measured wrongly once (as "app switch does not
  cancel", using adb), before a finger settled it.

  This corrects an earlier entry that claimed the opposite. That claim was an *inference*
  from an observed symptom ("it stops when I go to another app"), never a measurement, and
  the real cause was almost certainly the touch required to *perform* the switch, not the
  window change. Same observation, wrong mechanism.

  It also re-explains the original report that it "only works on the phone's menu": the
  finger was dragging the launcher, because it stays where it was put regardless of what
  is in front of it.

  That behaviour is arguably worse than cancelling. An auto-scroll that keeps dragging
  inside whatever app the user opens next is surprising at best, and the target user
  cannot quickly stop it. Production needs a deliberate decision here — and note that
  detecting the window change requires window-state events, which costs `eventTypes = 0`.

## Known deviations from spec

- `stop` is not instantaneous. A gesture in flight cannot be cancelled, so hard-stop
  latency is (remaining segment + 200ms). At the 250ms segment setting that's up to
  450ms. Logged as `STOP complete latencyMs=`.
- **Touching the screen stops the run.** A physical touch cancels the stroke, and the
  engine treats that as the user saying stop. It originally re-pressed 250ms later so
  that the cancellation count could exceed 1 — which meant the app fought the user for
  control of the screen and could not be stopped by touching it. `--ez repress true`
  restores the old behaviour for measurement runs only.

  **Never ship it on, and do not rely on remembering that.** A shipped build with this flag
  reachable is an app the user cannot stop by touching the screen — the failure mode the
  engine's whole cancellation design exists to prevent. Its removal belongs in the CI
  ratchet, asserted against the built artifact like the declarations are. **Recorded as a
  requirement, not as done:** the ratchet does not exist yet, and the spec in
  [DECISIONS.md](DECISIONS.md) currently scopes it to declarations only, so covering this
  means widening that spec rather than adding a row to it.
- The band is **25%–75%**, matching the original spec. It was widened to 12%–88%
  mid-spike on a theory that turned out to be wrong twice over, then narrowed back — see
  the `BAND_TOP_FRAC` doc comment for both reasons. **Verify per device** that 25% clears
  the status bar and 75% clears the nav/gesture strip, and record it in NOTES.md. The
  SM-P200 is the first device tested with 3-button navigation, where a persistent nav bar
  makes this check matter more than it did on gesture-nav devices.

---

## Handover to a control-surface spike

Spike 1 answered its question: a held virtual finger scrolls third-party apps smoothly, on
two devices five Android majors apart, two manufacturers, both form factors, both
navigation modes. **The engine is not the problem and needs no further work here.**

Everything below was found while testing device 2 and is about how a user *tells the engine
what to do*. None of it touches the scroll engine. It is listed so a control-surface spike
starts from evidence rather than a blank page.

### One reproducible open bug

**A confirmed press of the notification's Start action button fires no broadcast.**
ActivityManager logs `Broadcasting: Intent` for every adb-sent broadcast but logged nothing
for the button press, in a log window verified to cover it (16:00:59–16:01:43, 1845 lines,
nothing rolled, reader verified working beforehand). The developer confirmed pressing the
button, not the body.

Three explanations were proposed and each disproved by the next check: broadcast deferral
for background apps (ruled out — the process was at visible priority, and adb broadcasts to
the same receiver arrived instantly); a zombie service process (ruled out — the app was
scrolling the whole time, see the logcat retraction above); and a tap on the inert
notification body (ruled out — the developer confirmed the button). **Unexplained. Start
here.**

### STANDING DESIGN RULE: no control may require sustained input

**Any control the surface offers must be a discrete action, not a held one.** Sustained
contact is precisely the effort this app exists to remove — someone with weakness, tremor
or spasticity may not be able to hold anything steadily for the seconds a held control
would need.

This correction has now been reached twice from different directions: once on the edge
slider, and again on fast-forward (below), where the obvious design — hold to boost — is
what an able-bodied person reaches for and what a target user may be unable to do. Written
down here so it is not rediscovered a third time. It applies to every control the
control-surface spike proposes.

Latching (tap on, tap off) or a discrete fixed-distance action both satisfy it. Momentary
hold does not.

### Mixed media breaks the single-speed premise

Reported unprompted by a naive reader: a single speed works for text, but content with
images needs some control over it — something like fast-forward.

The 12 dp/s default was derived from reading rate, words per minute against line height.
**Images have no reading rate.** A 300dp figure at 12 dp/s takes 25 seconds to pass with
nothing to read. v1 was scoped around long-form articles and ebooks, which contain pictures,
so the model justifying a single set-and-forget speed did not account for its own content.

**Fast-forward is a transient, not a setting** — and that distinction is most of the cost.
A velocity change during CRUISE is nearly free in the engine: no new phase, no change to
the re-grip, and no breach of the catch-up invariant, because it is a new commanded velocity
rather than repayment of a debt. A *speed setting* is far more expensive — persistent state,
a UI to set it, and it reintroduces the continuous-control problem the design deliberately
removed.

Open, and not settleable by an informant outside the target population: whether it should
be **latching fast mode** (tap to enter, tap to leave — lets the user watch and stop when
the image clears,
suits someone who sees the screen well) or a **discrete skip** (one action, fixed distance,
the mirror of rewind-on-resume — needs no visual feedback loop, suits a mounted device).
That is a beta question. Note the input budget conflict either way: momentary boost maps
naturally onto proximity's hold-cover gesture, which is already allocated to the
spasm-tolerant hard stop.

**Both halves of that conflict are qualified as of 2026-08-13**, and it survives only if both
hold. The gesture may not exist: proximity is absent in hardware on the SM-P200, so on
tablet-class devices there is nothing to contend for. And the allocation it assumes is itself
unresolved: hold-cover is sustained input, which standing rule 1 forbids, and it is recorded
nowhere as a deliberate exemption. Both are open items in [DECISIONS.md](DECISIONS.md).

### Security: the control receiver was exported — CLOSED 2026-08-20

`registerReceiver(..., RECEIVER_EXPORTED)` in `AutoScrollService` meant **any installed app
could start and stop the scroll**, with no permission and no user interaction. Fine for a
spike driven from adb; not shippable. Two manifest-declared receivers replaced it — a
non-exported one that reads no extras, and a debug channel absent from release builds
entirely.

**The pairing mechanism is still unbuilt**, so v1 still has no third-party entry point. The
deadline that applied to the exported receiver now applies to that: before any build reaches
a device we do not control, including the closed beta.

**[DECISIONS.md](DECISIONS.md) → *Control surface: the receiver must not ship exported* is
the definition** — what closed it, what remains, the two rejected alternatives, and the bound
the privacy architecture already placed on the exposure. Not restated here, per rule 2 of
[RECORD-KEEPING.md](RECORD-KEEPING.md).

### Interstitial ads end the session, and the user may not be able to clear them

Observed while trying to run a reading session on a Chrome article: an ad appeared, the
reader touched the screen to dismiss it, and the touch cancelled the scroll — 5.7s into the
press, 59 ticks, run over.

Both halves matter. The ad **interrupts** reading, and dismissing it **stops the scroll**,
because touch-to-stop cannot distinguish "I want to close this ad" from "I want to stop
scrolling". For a user who cannot reliably touch the screen, an interstitial is worse than
an interruption: it is a modal they may not be able to clear, sitting on top of the content,
with the scroll already stopped.

This is not fixable inside the engine — dismissing a dialog *is* a touch, and treating
touches as anything other than stop was rejected for good reasons (see `repressOnCancel`).
It belongs on the limitations page and in the control-surface spike's restart problem.

Practical consequence for testing: **use ad-free content**. Wikipedia works well — long-form,
reflows with the system text scale, no interstitials.

### Host-app compatibility: what the vertical-only drag rules out

The finger only ever moves vertically, at x = 50% width, which removes several hazard
classes **structurally** rather than by luck:

- **Swipe-to-delete / swipe-to-archive** — horizontal gestures; unreachable.
- **Launcher icon pickup and drag** — needs a long-press first; ruled out empirically, see
  below.
- **Edge back-gesture** — activates within ~20–30dp of the left/right edges; a finger at
  50% width never approaches one.

Ruled out **empirically**: long-press fired at no margin value including 0, across a WebView
article and a PDF viewer, because kick + ramp clear slop well inside the long-press window.

Tested **accidentally**: the One UI launcher, for ~70 ticks during the `am start` run — it
scrolled normally and nothing was picked up or launched.

**Residual cases for the limitations page**, none tested:

| Case | Why it is a risk |
|---|---|
| Vertical drag-to-dismiss | Bottom sheets, some dialogs, Now Playing cards — a slow downward drag may dismiss rather than scroll |
| Short-video feeds | Vertical swipe means "next video"; a slow drag may page unpredictably |
| Video players | Many map vertical drags to brightness (left half) or volume (right half); at x = 50% the behaviour is undefined |
| Nested scroll / collapsing toolbars | The drag may collapse the toolbar and then stall, or hand off between scrolling containers mid-press |

The first three would be *wrong behaviour*, not merely ugly. Worth a deliberate pass before
any beta.

### Control-surface defects, each independently verified

| Finding | Evidence |
|---|---|
| Starting from the notification scrolled the shade, not the app | `mCurrentFocus` stayed `NotificationShade` for 86 ticks. Fixed; both API branches verified |
| Any control that is its own window captures the finger | Same root cause. A QS tile would behave identically. Proximity wave is immune, **but only where the hardware provides it** — absent on the SM-P200, so tablets have none. See DECISIONS |
| Notification body is inert — `contentIntent=null` | `dumpsys notification`. Large easy target dead, small button live: target sizing inverted for this audience |
| No feedback between press and visible motion | ~500ms panel wait + slop consumed by the host + 12 dp/s cruise ≈ a second before anything is unambiguous |
| Force-stop silently disables the service | Clears `enabled_accessibility_services` and `accessibility_enabled`. OEM battery managers force-stop routinely |
| Six ways to be "enabled but not working" | Master toggle off; package stopped after install; LMKD kill; force-stop; plus two instrument failures that mimicked them |
| `dumpsys accessibility` is not a liveness check | Reported "bound" on the SM-P200 with no process at all. Use `pidof` plus the `SVC connected` log line |

The unifying property: **the app cannot detect or report any of these, because in every one
of them it is not running to notice.** Liveness has to be observable from outside the
service. That is a design constraint for the control surface, not a bug to fix in it.

### Method notes worth carrying over

Four measurement failures in this spike returned clean-looking data about things the
instrument could not see — `input tap`, `am start`, an unreplicated A/B, and a sensitising
observer. A fifth was self-inflicted: `logcat -G 64M` broke logcat's reader on a Moto G54,
and the missing log lines were read as the app being dead, costing a force-stop, three
re-enables and a phone reboot before a screenshot showed it had been scrolling throughout.

**Before believing a result, ask what a null instrument would have produced** — and verify
the instrument before the measurement, not after.
