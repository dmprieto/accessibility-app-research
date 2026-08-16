# Per-device results

One block per device. Copy the template. `PRESS` in logcat reports density, screen size
and the device's measured slop / long-press timeout — paste those in rather than
guessing.

**`dev.spike.autoscroll` in the commands here is not stale and must not be swept.** Every
command in this file was run against a build carrying that package name; rewriting them would
make the record describe a build that never existed. The production identifier is a different
string and applies to a tree that has never been built — see *applicationId and namespace* in
[DECISIONS.md](DECISIONS.md), which is where both rules live and is not repeated here.

---

## Moto G54 5G — objective measurements only

Filled in from logcat across six driven runs on 2026-08-06.

> **HISTORICAL — read the constants in this block as they were at the time.** These runs
> predate three changes: the band was 12%–88% (now 25%–75%), `MAX_PRESS_MS` was 30s (now
> 120s, and chain-age decay turned out not to exist), and the slop margin was 4dp (now
> 1dp). The measurements are real; the settings they were taken under are not current.
> Current per-device data lives in [DEVICES.md](DEVICES.md).

**Device:** Moto G54 5G (cancunf)
**Android version / API:** _____
**Screen (px) / density:** 1080x2400 / 3.08
**Measured slop (dp) / long-press timeout (ms):** 8.1dp / **400ms**
**Min fling velocity:** 50.1 dp/s (154 px/s)
**Build tested:** 0.1-spike, post age-cap/segment-floor fix
**Date:** 2026-08-06

### Band geometry

- [ ] 12% of height clears the status bar (no pull-down triggered on press)
- [ ] 88% of height clears the nav bar / gesture strip (no nav gesture triggered on lift)
- [ ] x = 50% width never triggers edge back-gesture

Band resolved to 288..2112px = 593dp of travel. Note that at 12 dp/s only 366dp of it
gets used before the 30s age cap fires, so the band could be narrower at reading speeds.

### Timing

| Metric | Value |
|---|---|
| Jitter outliers (>±16ms) over 936 segments @ 100ms | **0** |
| Seam outliers (>16ms) over 936 segments | **0** |
| Typical seam gap | 1–3 ms |
| Longest clean single press | 65 s / 759 segments (6 dp/s, cruise only) |
| Re-grip dead time @ 12 dp/s | 466 / 480 / 472 ms |
| Re-grip dead time @ 100 dp/s | 591 / 584 / 565 / 585 ms |
| Stop latency, decel skipped | 126 ms |
| Stop latency, decel run | 262 ms |
| Cancellations in 95s @ 12 dp/s after fix | **0** |

### Chain-age failure (pre-fix, for the record)

| Speed | Chain age | Segment size | Outcome |
|---|---|---|---|
| 100 dp/s | 6.6 s | 3.84 px | ok |
| 12 dp/s | 36 s | 0.46 px | ok |
| 12 dp/s | 48 s | 0.46 px | **cancelled** |
| 6 dp/s | 62 s | 0.69 px | **cancelled** |
| 6 dp/s | 65 s | 1.85 px | ok (cruise, no decel) |

### Visual verdict — 57-page PDF, 12 dp/s, 5 re-grips observed

Re-grips in the observed run: 466 / 460 / 473 / 468 / 467 ms, 0 cancellations over ~3 min.

> "the jump is not overly distracting to me but it is noticeable so it could bother a
> different type of user"

Caveats on that verdict worth carrying forward: the observer was the developer, knew when
each re-grip was due, and was evaluating rather than reading. At 30s intervals this is
**~120 interruptions per hour** of continuous reading.

Decomposition of the "jump" — two separable effects:

| Component | Size | Fixable? |
|---|---|---|
| Dead time (lift → reposition → press → cruise) | ~470 ms | Hard floor; IPC bound |
| Lurch on re-press — **originally mis-measured**, see below | 4 dp margin | Partly — `slopMarginDp` is a weak lever; `LEAD_IN_MS` is the dominant one |

### Slop-margin sweep — RESOLVED

Swept in Chrome on a Vanity Fair article, 12 dp/s, screenshots at t=2s and t=10s.

| Margin (dp) | AS LOGGED (wrong) | **MEASURED** mean | peak | ramp's share | Long-press? | Selection UI? | Scrolled? |
|---|---|---|---|---|---|---|---|
| 4 (old default) | 2.8x | **3.96x** | 8.3x | 74% | no | no | yes |
| 2 | 1.4x | 3.00x | 6.9x | 83% | no | no | yes |
| **1 (current default)** | 0.7x "ease-in" | **2.53x** | 6.3x | 90% | no | no | yes |
| 0 | 0.0x | **2.05x** | 5.6x | **100%** | no | no | yes |

Measured on the SM-P200 at 12 dp/s with the corrected metric, 2026-08-12. The first three
columns of the old sweep were judged against a figure that measured only the margin term.

**The metric this sweep was judged against was wrong.** The `VISIBLE LURCH` log line
reported only the margin term over `LEAD_IN_MS` and ignored the RAMP, which starts at
`kickPxPerS()` and carries ~90% of the visible motion at margin 1. So the sweep was tuning
against roughly a tenth of the effect.

The 4dp → 1dp change was still a real improvement — 3.96x → 2.53x measured — and the
observer verdict that followed it stands. But **the lead-in is still a ~2.5x overshoot, not
the "ease-in" it was recorded as.**

### Lead-in duration sweep — the dominant lever, measured

SM-P200, 12 dp/s, margin 1dp, long-press 500ms so the clamp sits at 375ms.

| Requested | Actual | Visible | Mean | Peak |
|---|---|---|---|---|
| 120 ms (default) | 120 | 9.7 dp / 320ms | **2.53x** | 6.3x |
| 200 ms | 200 | 6.7 dp / 400ms | 1.40x | 3.8x |
| 250 ms | 250 | 5.8 dp / 450ms | 1.07x | 3.0x |
| 300 ms | 300 | 5.2 dp / 500ms | 0.87x | 2.5x |
| 375 ms | 375 | 4.6 dp / 575ms | **0.67x** | 2.0x |
| 500 ms | **375 (clamped)** | 4.6 dp / 575ms | 0.67x | 2.0x |

**At 375ms the lead-in is genuinely an ease-in** — 0.67x mean, 2.0x peak — which is what the
old broken metric wrongly claimed 120ms already was. `kickPxPerS()` scales inversely with
duration, so the model behind the recommendation holds.

**The cost is re-grip dead time.** Lead-in + ramp goes from 320ms to 575ms, so `deadMs`
grows from ~440ms to ~695ms — a ~58% longer pause in exchange for removing the overshoot.

**That trade is not yet decided, and the evidence points both ways.** An observer judged the
*discontinuity* rather than the pause, which argues for taking it. But the naive reader
noticed neither at 120ms — where the overshoot is 2.53x — so there may be nothing to buy.
Needs eyes, ideally the same reader on both.

The clamp is device-dependent: 0.75x the measured long-press timeout, so 375ms here and
**300ms on the Moto** (400ms timeout). A single shipped constant cannot be optimal on both.

**Margin is a weak lever, now measured rather than argued.** The entire range buys
3.96x → 2.05x, and at margin 0 the ramp *is* the whole effect (100%) with a 5.6x peak still
present. Nothing in this parameter gets near 1x cruise. `LEAD_IN_MS` sets `kickPxPerS()` and
therefore the ramp's starting velocity, so it is the only lever that reaches the dominant
term — see its doc comment for the ~3x unused headroom and the dead-time cost.

**Long-press never fired, even at margin 0.** The kick alone does not clear slop at
margin 0 — kick + ramp together cover ~16dp by 320ms against 8.1dp of slop (~2x headroom)
inside the 400ms deadline. So this parameter is a smoothness knob, not a safety one.
Caveat: one device, two host apps.

**Observer verdict at margin 1**, same PDF and same speed as the margin-4 baseline:

> "it looks better on the web browser but is not bad on the pdf, an overall improvement
> compared to previous iterations"

Re-grips at margin 1: 458 / 459 / 459 / 461 ms, 0 cancellations. Tighter than at margin 4.

### Hazard found while sweeping

At y = 88% height the virtual finger lands **directly on Vanity Fair's sticky
"GET DIGITAL ACCESS" footer**. It scrolled correctly here, but a sticky element that
consumes its own touches (carousel, bottom nav, floating button) would swallow the drag.
Second independent argument for narrowing the band back toward 25%–75%; the first is that
age binds before travel below ~20 dp/s, so the extra travel buys nothing at reading speed.

### Still unanswered — needs eyes on the screen

- Any context menu, selection handle, tooltip or stuck press state?
- Any overscroll glow or fling on lift?
- Behaviour with `--ez leadin false` (expect long-press within 400ms)?
- Same verdict in a WebView / browser article, where effective slop may exceed
  `scaledTouchSlop`?

---

## Device 2 (Samsung SM-P200) — protocol and results

Per-device numbers go in [DEVICES.md](DEVICES.md) as structured rows. This file is for
working notes and subjective observations; that one is the dataset.

**Pick device 2 for coverage, not convenience: different manufacturer and different
Android major version from the Moto (motorola / API 35).** Samsung One UI or a
Pixel/AOSP build adds the most, since those are the two gesture pipelines most likely to
differ.

Run the stages in order.

### 1. Cancellations — two halves, both required

A passive session alone can return zero, which tells you almost nothing: you cannot
distinguish "spurious cancels are rare" from "nothing happened to provoke one". Do both.

```
adb logcat -s AutoScroll:V | grep CANCEL
```

**1a. Hands-off soak.** Phone resting untouched, 10+ minutes at 12 dp/s margin 1, across
two or three host apps. Establishes the passive rate.

| Device | Host app | minutes | presses | regrips | cancels | pid survived? |
|---|---|---|---|---|---|---|
| SM-P200 | Google Drive PDF viewer (151pp) | 9 | 14 | 13 | **0** | yes (6899) |
|  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

**Passive rate so far: 0 per 9 min**, one device, one host.

**Capture note — do not repeat this mistake.** The logcat ring buffer (5MB) rolls during a
run this long, and dumping with `logcat -d` at the end loses most of it: 684 ticks were
recovered out of ~5400, and 1 of 13 `REGRIP` lines. The engine's cumulative counters on the
final `LIFT` are authoritative and survived, but per-event detail did not. **Stream to a
file during the run** (`adb logcat > file &`) or bump the buffer first — but use
`-G 16M`, **not** `-G 64M`, which broke the reader entirely on a Moto G54. **Verify with
`logcat -g` either way — and treat that as mandatory rather than advisory: the SM-P200 refuses the
raise and caps at 5 MiB, so on this device the buffer stays 5 MiB whatever you asked for** (measured
16 Aug 2026).

**1a-ter. Before any run, confirm the service is actually bound.**

`dumpsys accessibility` is not sufficient — it listed the service under both "enabled
services" and "bound services" on the SM-P200 while no process existed. Use:

```
adb shell pidof dev.spike.autoscroll
```

An empty result means nothing is running, whatever Settings or dumpsys claim. Causes seen:
master toggle off, package left in the stopped state by a reinstall, or LMKD. Cycling the
Settings toggle fixes the first two; adb writes to the secure settings do not rebind on
One UI.

**Check this before interpreting any "no cancels" result.** A run against a dead service
produces zero ticks, zero cancels, and a clean-looking pass — the null-instrument trap in
its most mundane form.

**1a-bis. Distinguish the THIRD failure mode: process death.**

A scroll can stop for three reasons, and only two of them log anything:

| Cause | `CANCEL` line? | `STOP` line? | How to tell |
|---|---|---|---|
| User touch / window change | yes | yes | normal |
| You sent stop | no | yes | normal |
| **Service process killed** | **no** | **no** | ticks simply end; pid changes |

On memory-constrained devices the low-memory killer can take the service out mid-session.
Observed on the SM-P200: LMKD thrashing killed a Samsung *system* service while Play Store
was updating, on a 2.8GB device. A doze whitelist does not protect against LMKD; nothing
does.

If this happens to a real user the scroll just stops, silently, with nothing to explain it.
Do not let it contaminate the spurious-cancel count — check the pid whenever a run ends
without a log line:

```
adb shell pidof dev.spike.autoscroll
```

Record it before the soak and compare after. A changed pid means the process was restarted;
ticks ending with no `CANCEL` and no `STOP` means it died.

| Run | pid before | pid after | ended with | verdict |
|---|---|---|---|---|
|  |  |  |  |  |

**1b. Provocation pass.** Deliberately fire each suspected cause while scrolling and
record whether it cancels. This is what turns a zero into a finding.

**SM-P200 / Google Drive PDF viewer / 12 dp/s — results:**

| Provocation | cancelled? | notes |
|---|---|---|
| **Physical finger on screen** | **YES, 12ms into the segment** | The positive control. Touch-to-stop is real. |
| Notification posted | no | `cmd notification post` |
| Shade expand + collapse | no | `cmd statusbar expand-notifications` / `collapse` |
| Volume key (system overlay) | no | |
| Screenshot | no | |
| **Physical HOME button press** | **YES, 9ms into the segment** | Cancels before the launcher comes forward. A real app switch is a touch. |
| App switch via `am start` (no touch) | no — finger follows into the new app | ticks continued 79 → 149. Only reachable without a touch: call, alarm, full-screen intent, deep link. |
| `adb shell input tap` | **no** | see methodology note below |
| IME appears | not tested | needs a text field; not automatable without touching |
| Incoming call | not tested | no SIM in this device |
| Screen rotation | not tested | |

### STANDING RULE: a synthetic provocation needs a positive control from the real modality, run FIRST

Before trusting any negative result from a synthesised event, prove the instrument can
produce the thing being measured — using the real modality, before the provocations, not
after.

This rule caught two separate errors on one claim:

| Instrument | Result | Verdict |
|---|---|---|
| `adb shell input tap` | no cancel | **invalid** — synthesised via InputManager from shell uid, not a digitiser touch |
| `am start` to launcher | no cancel | **invalid as an app-switch proxy** — changes the window without a touch, and only the touch cancels |
| Physical finger on screen | cancels in 12ms | valid control |
| Physical HOME button | cancels in 9ms | valid control |

Apply it to everything still untested here — rotation, IME, incoming call. Ask first
whether the adb command reproduces the user action or merely its side effect.

**Methodology note, important for anyone repeating this: `adb shell input tap` does NOT
cancel an accessibility gesture.** It is synthesised through InputManager from the shell
uid and the framework does not treat it as a finger on the digitiser. Any provocation pass
driven purely from adb will therefore report "nothing cancels" — including the cases that
genuinely would. **A physical touch is the only valid positive control**, and every
adb-only negative is uninterpretable without one.

Run the control FIRST. Five provocations were run here before the control, and all five
looked like clean negatives; they only became trustworthy once a real finger confirmed the
detector fires.

If none of these cancel, touch-to-stop is a feature **and you know why**. If some do,
those exact rows are the limitations-page entries.

**Reading the pressAge distribution:**

- Tight cluster at a consistent age, reproducible across speeds and apps, fresh press
  immediately fine afterwards → this device really does decay. Lower `MAX_PRESS_MS`.
  **Check `MIN_SEGMENT_PX` and the fling-threshold skip are in the build first** — that
  combination is what produced the false decay signal on device 1.
- Diffuse ages, or correlation with `yFrac` or host app → something else. That is a
  finding, not a tuning problem.

This decides whether touch-to-stop is a feature or a limitations-page entry.

### 2. Probe the decay threshold directly

Raise the cap and find where it breaks, if it breaks. `--ei cap N`. Use a slow speed so
travel does not end the press first: at 4 dp/s the band allows ~68s, at 6 dp/s ~54s.

Device 1 found no failure anywhere — 35 / 40 / 45 / 50s all clean, plus one 68.07s press.

| cap (ms) | speed (dp/s) | press survived? | age at failure |
|---|---|---|---|
| 35000 | 6 |  |  |
| 50000 | 6 |  |  |
| 90000 | 4 |  |  |

### 3. Delivered speed, and whether the correction settles

Delivered speed = `cum` ÷ `heldMs` from any `LIFT` line. Record with correction off and
on, at each speed. Device 1 raw: +43% at 4 dp/s, +21% at 6, +2% at 12, −13% at 100.

| dp/s | delivered, `ratecorrect false` | delivered, `ratecorrect true` | ratio settled at |
|---|---|---|---|
| 4 | not measured | not measured | |
| 6 | not measured | not measured | |
| 12 | not measured | **+0.2% / −0.1% / +0.2%** (3 presses) | 0.90–0.94, oscillating |
| 100 | not measured | not measured | |

Only 12 dp/s was measured on the SM-P200. Note the ratio **oscillates in a band** here
rather than settling to a point as it did on device 1 — that is the filter tracking a
genuinely noisy input, not failing to converge, and delivered rate stayed within 0.2%
throughout. **This device validates the decision to filter rather than use raw
last-segment elapsed:** with ±35% swings a raw estimator would have injected that noise
straight into commanded velocity.

Watch `ratio=` in the TICK lines. Settling confirms systematic bias and the fix is done;
wandering means noise is being fed into on-screen velocity and the filter is too fast.

### 4. Subjective — the one no instrumentation can produce

> **RULES FOR RECORDING OBSERVERS — read before filling in any table below.**
>
> Git history is permanent. Anything written here cannot be withdrawn later without
> rewriting history or abandoning the repo, so the convention has to hold from the first
> participant session, not be applied retroactively.
>
> 1. **Pseudonymous IDs only — `Observer A`, `Observer B`.** No names, and no initials:
>    in a group this small, initials identify.
> 2. **Never record impairment, diagnosis, age, assistive-tech setup, or anything about
>    the person's body or health.** If that context is needed to interpret a result, keep
>    it out of git entirely. For this project it is special-category data under GDPR
>    Art. 9, and a repo cannot honour a later erasure request.
> 3. **Verbatim quotes only when the person knows they are being written down.** Say so
>    out loud before the session; it costs one sentence.
> 4. **Distinguish self from participant.** The developer's own quotes are self-disclosure
>    and carry none of this. A recruited observer's are not, and do.
> 5. **Recruiting actual target users is human-subjects work, not a NOTES.md exercise.**
>    Consent, purpose, retention and withdrawal need deciding before the session, not
>    improvised in a table afterwards.
>
> **Consent record — Observer A.** Every verbatim quote attributed to Observer A in this
> file — the unprompted and prompted reports from the SM-P200 session, and the mixed-media
> feedback below — was recorded after they were told out loud, before the session, what
> would happen to their words: that they would be written down verbatim, that these notes
> would be **shared as a public repository for review**, and that **nothing about their
> health, body or identity would be recorded alongside them**.
>
> That last undertaking is what rule 2 exists to keep, and it is checkable rather than
> promised: the only things recorded about Observer A anywhere in this repo are a pseudonym
> and their relation to the target population. No impairment, diagnosis, age, assistive-tech
> setup, or health data. Their quotes describe the experience of reading a scrolling screen,
> which is the object of study, and nothing else about them.
>
> **Re-confirmed after the fact.** Observer A was asked again once the repository was
> actually public, and agreed to the quotes appearing in it on those terms. The pre-session
> disclosure and the confirmation are recorded separately on purpose: agreeing to be written
> down and agreeing to be published are different undertakings, and only the second one can
> be given by someone who knows what the repository turned out to be.
>
> Quotes of the developer's own are self-disclosure and carry none of this, per rule 4;
> where both appear, the attribution says which is which.
>
> **Update this line whenever a quote is added**, and do not let it fall behind the tables
> again. It said "the two quotes currently in this file are the developer's own" well after
> the file had acquired three of Observer A's — the one line carrying the consent record was
> the one line that had gone stale.

Does the eased re-grip read as a blink or a glitch on hardware it was not tuned on?

**Get a second pair of eyes if at all possible.** Every subjective judgement on this
project so far comes from one observer who knew when each re-grip was due and was
evaluating rather than reading. Someone who does not know when to look, handed the phone
to read something they actually want to read, is worth more than any further logging.

Three things make this less soft than "one person's impression":

**4-pre. LOG THE SESSION. Capture the re-grip count before anything else.**

An unprompted null only carries weight if there were events to miss. *"No disturbance
reported"* and *"no disturbance reported across 9 re-grips"* are different claims, and only
the second survives someone asking how you know. The Moto session was not logged and the
count had to be derived from duration afterwards — do not repeat that.

Before handing the device over:

```
adb shell pidof dev.spike.autoscroll          # confirm it is actually running
adb logcat -G 16M                             # NOT 64M - that broke the reader on the Moto
                                              # SM-P200 refuses this and caps at 5 MiB - check -g
adb logcat -g                                 # verify the buffer took
adb logcat -c
```

Start the scroll from adb, note the wall-clock time, and afterwards:

```
adb logcat -d | grep -c REGRIP
```

Record: session length, re-grip count, cancellations, whether the pid survived, **and the
apparent text size / zoom level**. The last one was missed on the first two sessions and
became a confound the moment the reader raised it unprompted. Nothing in logcat captures
it — it has to be written down at the time.

A session that ended early because the reader touched the screen is a different result from
one that ran clean, and only the log distinguishes them.

**Target for the tablet session: 5 minutes.** At 12 dp/s the SM-P200 band gives ~39s per
press, so 5 minutes is ~8–9 re-grips — enough that an unprompted null is a strong claim
rather than a thin one. The Moto's 2-minute session yielded only ~3–4, and derived at that.

**4a. Mark-and-correlate.** Have the observer mark the moment — a tap on the desk, a word,
anything you can timestamp — whenever they notice a disturbance or lose their place. Then
check the marks against `REGRIP` timestamps in logcat. If reported disturbances do not
cluster on re-grips, the re-grip is not what is bothering them whatever they say it is.
**Nothing reported across a session containing twenty re-grips is a stronger result than
any verdict they could give.**

| Session | re-grips in session | marks reported | marks within ±1s of a REGRIP | attributable? |
|---|---|---|---|---|
|  |  |  |  |  |

**4b. Do not ask a leading question.** "Did the re-grip read as a blink or a glitch" tells
a primed observer exactly what to find. Have them read for five minutes and report
anything at all about the experience, unprompted. Only then ask directly, and record both
answers separately — the gap between them is itself informative.

| Observer (pseudonym) | device | session | unprompted report | prompted report | primed? |
|---|---|---|---|---|---|
| Observer A | Moto G54 (smooth) | 2 min, ~3–4 re-grips (derived) | velocity fine; **no mention of any bump, no loss of place** | not separately recorded | no |
| Observer A | SM-P200 (noisy) | **6 min, 9 re-grips (counted)** | **"a very small interference, kind of the text getting stuck for a brief moment... made me feel tired"** | speed fine; didn't get lost; "I could still read ok but made me more tired" | no |

### SM-P200 session — the important negative result

Verbatim, unprompted: *"At the beginning I felt comfortable reading but after a while there
seemed to be a very small interference kind of the text getting stuck for a brief moment and
that made me feel tired."* Prompted follow-ups: speed fine, did not get lost, *"not sure how
to describe it, I could still read ok but made me more tired."*

Telemetry: 10 presses, **9 re-grips** (440/432/428/428/431/436/428/428/428ms, mean 431,
spread 12ms), 4006 ticks, 4424dp, one cancellation at the end when the reader touched the
screen. Jitter this session: mean 12.6ms, **36% of segments over ±16ms** — noisier than the
21.6% recorded in DEVICES.md, so this reader got a harder version of the hardware than the
dataset describes.

**Three things this changes:**

1. **Fatigue is a distinct axis from visibility, and no instrument measures it.** Every
   question asked so far was "is it visible". This answer is: barely, survivable, *and it
   accumulates*. For an app used to read for long periods that may matter more than
   visibility. Nothing in logcat would ever surface it.
2. **It is time-dependent** — "at the beginning I felt comfortable... after a while". The
   Moto session was 2 minutes and **could not have detected this at any hardware quality**.
   The Moto's clean verdict is therefore not evidence against this result, and device and
   duration are currently confounded.
3. **The artifact is unidentified.** "Text getting stuck for a brief moment" could be the
   9 discrete ~431ms re-grips or the continuous jitter. The phrasing leans discrete, but
   "very small" fits a 431ms pause poorly.

4. **A THIRD confound, volunteered by the reader: the font was smaller.** *"the font size
   seems smaller on this example and they are not sure if that had to do with it feeling
   different."*

   This may be the whole finding. **12 dp/s is a distance rate; reading demand is a line
   rate**, and the two are related by line height. Smaller text at the same dp/s means more
   lines per second — the reader was being asked to read faster, and fatigue is exactly what
   a slightly-too-fast scroll produces. Smaller text is also more tiring to read on its own
   terms, independent of scrolling.

   The devices make this worse rather than better: density 2.00 on the tablet against 3.08
   on the Moto, and an 8" panel against a 6.5" one. The same PDF at the same zoom does not
   present at the same physical text size on both.

   **Structural consequence, and the same gap as the images feedback:** the 12 dp/s default
   was derived from words per minute x line height. The app cannot know line height — it
   cannot read the screen, by design. So one dp/s setting cannot be correct across font
   sizes any more than it can across mixed media. Two independent observations now point at
   the same missing input.

**Attribution is now weak.** Three confounds — device, duration, font size — and the
artifact was never identified. Do not carry "the scroll artifact causes fatigue" forward as
a finding; the honest statement is *"a 6-minute session on smaller text on the noisy device
produced reported fatigue, cause unresolved."*

### Font-size test, session 1 — the reader separated the artifacts unprompted

Wikipedia prose, Chrome, `font_scale=1.3` (large), 12 dp/s, 5–6 min. Telemetry: 10 presses,
**9 re-grips** (436–469ms), **0 cancellations**, 4662dp, jitter 20% outliers (mean 11.9ms).

Verbatim: *"I feel less fatigued not sure if it is related to the text size, I didn't feel
the small interference and the continuous scrolling of the page seemed smoother. I did
notice 2 or 3 times where there was a small jump but I could continue to read without
issues."*

**This is the discrimination 4a was written to produce, and it arrived unprompted.** The
reader distinguished two artifacts by their character:

| Their words | Character | Maps to | This session |
|---|---|---|---|
| "the small interference" | continuous | jitter / micro-stutter | **absent** (20% outliers vs 36%) |
| "a small jump", 2–3 times | discrete | the re-grip | present, noticed ~3 of 9, **"could continue to read without issues"** |

**If this holds it inverts the spike's priorities.** The re-grip — the artifact this engine
was built around, tuned for, and measured to ~431ms — is noticed occasionally and rated
harmless. The fatigue tracked the *continuous* artifact, not the discrete one.

**Do not conclude yet: this comparison is confounded four ways**, because the previous
session differed in more than font size.

| | Previous (fatiguing) | This one (comfortable) |
|---|---|---|
| Font | smaller | larger (1.3) |
| Jitter outliers | 36% | 20% |
| Content | PDF with figures | Wikipedia prose |
| App | Drive PDF viewer | Chrome |

Session 2 fixes three of those four: same device, same app, same content type, same duration,
**only the font scale changes**. That is the comparison that can actually attribute.

### RESOLVED: fatigue tracks the continuous artifact, not the re-grip

Four sessions, same reader, same device, 12 dp/s, ~5-6 min each, 9 re-grips in every one.

| Session | Text | Jitter outliers | "Interference" (continuous) | "Jumps" (discrete) | Fatigue |
|---|---|---|---|---|---|
| PDF, original | small | **36%** | present | — | **most tired** |
| Font S1 | large (1.3) | 20% | **absent** | noticed 2-3 of 9, harmless | least |
| Font S2 | small (0.8) | 19% | "minimum but noticeable" | **not noticed** | slight, manageable |

Verbatim, S2: *"there was minimum interference but noticeable compared to the previous
tests, didn't notice the jumps"* and *"slightly more fatigued than the previous test but was
manageable"*. On the original PDF session: *"felt more tired with the pdf smaller text"*.

**Fatigue tracks the interference in perfect rank order across all three, and never tracks
the jumps.**

**Conclusion 1 — the re-grip is not the problem.** It was noticed in one session of three,
called harmless, and never linked to fatigue. The engine was built, tuned and measured
around making it invisible: the lead-in profile, the margin sweep, the catch-up invariant,
~431ms of dead time characterised to a 12ms spread. That work is correct and it optimised
the artifact that does not matter.

**Conclusion 2 — jitter is the problem, and text size scales its cost.** S1 and S2 were
mechanically matched (9 re-grips each, 20% vs 19% outliers, same app, content and duration)
and differed only in font scale, yet the interference appeared only at the smaller size. A
fixed dp deviation is a larger fraction of a 17dp line than a 24dp one, so the same wobble
is more salient against smaller text. The original session's severity is explained by having
both: small text *and* nearly double the jitter.

Note the artifacts trade off in opposite directions with text size — the discrete jump was
noticed only at *large* text, presumably because there is less to read per screen and a
pause is less easily absorbed. Neither was disruptive to reading in any session.

### The segment-duration null was measured on the wrong outcome

The A/B/A that returned "not reliably discriminable" tested **discrimination over ~90
seconds**. Fatigue is a different outcome and takes minutes to appear — that design could
not have detected it. The null stands for what it measured and says nothing about this.

And the sweep already showed the relevant lever: jitter is fixed-cost, so it dilutes with
segment duration — **12.6% of segment at 100ms, 5.3% at 250ms**.

**Test run, prediction NOT confirmed.** Small text (0.8), 250ms segments, 5 min, same
reader. The mechanical change landed exactly as forecast — relative jitter fell from 11.8%
to **4.3% of segment**, a 2.7x reduction, with re-grips unchanged at 9 and dead times
unchanged at 441–465ms. The perceptual result did not follow.

| | S2 (100ms) | S3 (250ms) |
|---|---|---|
| Jitter, % of segment | 11.8% | **4.3%** |
| Re-grips | 9 | 9 |
| "Interference" | "minimum but noticeable" | "little" |
| "Jumps" | not noticed | **"a few"** |
| Fatigue | slight, manageable | none |

Verbatim: *"little interference and noticed a few jumps, reading went well and they didn't
feel tired. Not a significant difference that they can spot between tests."*

**Do NOT read this as "the jitter reduction is imperceptible".** That was the first
conclusion drawn here and it was too strong. Three reasons:

1. **The comparison was serial and memory-based.** S3 was judged against a recollection of
   S2 from ~30 minutes and one session earlier. *"Not a significant difference that they can
   spot between tests"* is a statement about their **ability to compare**, not about the
   artifact. Memory for a subtle continuous quality degrades fast, and this was the fourth
   repetition of the same task.
2. **The one non-memory signal moved the right way.** Fatigue went from "slightly more
   fatigued, manageable" (S2) to "didn't feel tired" (S3) — in a *later* session, when
   accumulation should have pushed the opposite way. That is the only measure not requiring
   recall, and it favours the reduction.
3. **A coping reader does not introspect.** "Reading went well" is satisficing. Once the
   experience is comfortable there is no reason to hunt for detail, so absence of reported
   detail is not absence of difference.

**Honest status: the jitter reduction is unresolved, not disproven.** The text-size effect
(S1 vs S2) remains the better-supported result because those arms were closer together and
the difference was reported spontaneously rather than on request.

**The instrument is the problem, and it is another null-instrument case.** Serial
self-comparison across repeated sessions cannot detect a subtle continuous difference —
memory and habituation both work against it. To settle this properly:

- **Between-subjects: one session per reader, fresh readers, one condition each.** No memory,
  no habituation, no order effects. Costs more readers; it is the only design that answers
  the question cleanly.
- Or **within-session switching** — change segment duration mid-read without telling them,
  and ask at the switch points, so the comparison is immediate rather than recalled.

**Confound to weigh against the fatigue improvement:** S3 was this reader's fourth session
of the day. Fatigue accumulates, so reporting *less* tiredness in a later session is
notable — but habituation cuts the other way, and their reports have converged toward "it's
fine" across the sequence. **This reader is now heavily primed and practised; further
sessions with them have low value.** A fresh naive reader is needed to take any of this
further.

**Honest state of the question:** the re-grip is confirmed unimportant across four sessions.
The continuous "interference" is real, tracks text size, and is *not* explained by jitter
magnitude. Its mechanism is unidentified.

**Two tests separate the confounds, in this order:**

- **Duration vs device — and control the font.** The same reader, 6 minutes, on the
  **Moto**, with the text at the *same apparent size* as the tablet session. Without that
  control the comparison cannot separate hardware from font size, and will produce another
  uninterpretable result. Match apparent size by eye and record the zoom level.
- **Font size alone:** same device, same duration, same content at two text sizes. If
  fatigue tracks font size rather than device, the finding is a speed-model gap and not a
  scroll artifact at all — which would make it a *bigger* result, not a smaller one.
- **Which artifact:** run **4a mark-and-correlate** — have them mark the moment they notice
  anything, and check the marks against `REGRIP` timestamps. Clustering on re-grips means
  the pause; diffuse marks mean the micro-stutter. This is exactly what that protocol was
  written for.

**Do not tune anything until the artifact is identified.** The lead-in lever (0.67x at
375ms) addresses the re-grip restart; it does nothing for jitter. Picking a fix before
knowing which artifact is responsible is the same shape as tuning the margin against a
broken metric.

**RESULT — the finding this spike existed to produce.** Moto G54, PDF with figures,
12 dp/s. An unprimed reader reported the speed comfortable and did not mention the re-grip
or losing their place at any point, unprompted.

Set against the primed observer on the same hardware calling it "noticeable", this is
independent support for the sensitisation hypothesis from the A/B/A — the disturbance is
detectable when you are looking for it and not otherwise.

**Caveats, which must travel with the number:**

- **n = 1, one session.** Do not quote it as settled.
- **Re-grip count is DERIVED, not logged: ~3–4 across a 2-minute session.** The session
  was not logged; the reader read for about 2 minutes, and at 12 dp/s on this device travel
  binds every ~32.5s, giving roughly 3–4 re-grips. So the claim is *"no disturbance reported
  across ~3–4 re-grips"* — real, but thin. Treat it as supporting evidence rather than the
  citable result, and see step 4-pre: **log the tablet session.**
- **Not a member of the target population.** Fine for the perceptual question — perception
  transfers — but see the fast-forward note below, where it stops being fine.
- **This was the smooth device.** The Moto had zero jitter outliers in 936 segments. The
  SM-P200 has 216/1000 and ±35% instantaneous velocity variation, and has still had no eyes
  on it at all. **That is now the only open perceptual question.**

### Additional feedback from Observer A: mixed media breaks the single-speed model

> having just one speed works fine just for text but if the text has images then it would be
> useful to have some control on the speed like a fast forward functionality

This is content-driven and does not depend on the observer's ability, so it stands on its
own. The 12 dp/s default was derived from reading rate — words per minute against line
height. **Images have no reading rate.** A 300dp figure at 12 dp/s takes 25 seconds to pass
with nothing to read. Long-form articles and ebooks were the scoped v1 content, and they
contain pictures; the model that justified a single set-and-forget speed did not account
for that.

Framing that matters for cost: **fast-forward is a transient, not a setting.** A velocity
change during CRUISE is nearly free in the engine — no new phase, no change to the re-grip,
and it does not breach the catch-up invariant because it is a new commanded velocity rather
than repayment of a debt. A *speed setting* is far more expensive: persistent state, a UI to
set it, and it reintroduces the continuous-control problem the design deliberately removed.

**4c. Blind A/B — `--ei segment 100` vs `--ei segment 250`.** Two sessions, order
randomised, without telling them what differs or that anything does.

Segment duration is chosen over slop margin deliberately: margin was already judged an
improvement on device 1 and has no open decision attached, whereas segment duration has a
live decision, a measured predicted difference on this hardware (12.6% → 5.3% mean
instantaneous velocity variation), and an observer who can settle it. If 100ms already
reads as smooth to someone unprimed, the question is moot.

| Observer (pseudonym) | order run | identified a difference? | which preferred? | correct? |
|---|---|---|---|---|
| developer (blind to assignment) | 250 / 100 / 250 | claimed yes, but **grouped the two different settings together** | n/a | **no** |

**RESULT: not reliably discriminable.** Verbatim, in order —

- 250ms: *"feels smooth, didn't lose place and seems ok in general"*
- 100ms: *"scroll seems to have a slight lag and the bump is more noticeable but it is still usable"*
- 250ms: *"to me 2 and 3 seem similar"*

Sessions 1 and 3 were the same setting; the observer grouped 3 with 2. Measured jitter as a
fraction of segment was 5.1% / 11.5% / 5.6% — a real 2x difference that did not survive
replication perceptually. Most likely sensitisation: by session 3 the observer was hunting
for artifacts they had not been hunting for in session 1.

**Independent confirmation that the reports were not tracking the variable.** The observer's
one concrete complaint in session 2 was *"the bump is more noticeable"* — the bump being the
re-grip. But re-grip dead time cannot depend on `segmentMs`: decel, hold, kick and ramp all
run on `PROFILE_SEGMENT_MS` / `HOLD_MS` / `LEAD_IN_MS`, and only cruise uses the configurable
value. Measured directly, with what was recorded at the time as a 64M logcat buffer:

> **CAVEAT ADDED 2026-08-16 — the buffer was not 64M and the capture-completeness claim has no
> basis as written.** This measurement is in the SM-P200 section, and **that device refuses any
> buffer above 5 MiB**, replying *"MAX log buffer size is 5 MiB. So set it to 5 MiB"* — measured
> 16 Aug 2026 on two invocation forms and at 8M as well. So `logcat -g` would have returned 5 MiB,
> and the words "so nothing rolled" could not have been written by anyone who ran it.
>
> **This is not a retraction of the numbers.** 5 MiB may well have been ample for four `REGRIP`
> lines, and the mechanism argument above — re-grip dead time cannot depend on `segmentMs`, since
> decel, hold, kick and ramp run on other constants — stands independently of the capture. **What is
> withdrawn is the stated reason for believing the capture was complete.** Whether anything rolled is
> now unverified rather than disproved.
>
> Note the tension with the same passage below: it records that *every* A/B and sweep session lost
> most of its telemetry to roll. **Re-derive the figure, or cite it with this caveat attached.**
> Recorded as the eighth entry in standing rule 5's table in `DECISIONS.md`, where it is also the
> reason that rule cannot catch this species — the check was written, not run.

| Segment | REGRIP deadMs |
|---|---|
| 100 ms | 434, 429 |
| 250 ms | 427, 434 |

7ms of spread across a 2.5x change in segment duration. The named artifact was identical in
both arms. This takes the null from "did not replicate" to **"the reports were not tracking
the manipulated variable"**, which is a much stronger statement.

**The A/B/A design is the whole reason any of this is known.** A plain A/B of sessions 1
and 2 would have read as "250ms is clearly better" and shipped a default change on one
unreplicated comparison. Always repeat the baseline.

**Capture lesson, and its own trap:** every A/B and sweep session lost most of its
telemetry to ring-buffer roll (28-79 ticks captured of ~900). Raising the buffer fixes
that — but **`-G 64M` broke logcat entirely on a Moto G54**: the buffer reported
`64 MiB, 16 MiB consumed, 2 MiB readable` and `logcat -d` returned almost nothing, which
was then misread as the app being dead. It cost a force-stop, three re-enables and a phone
reboot before a screenshot showed the app had been scrolling the whole time.

Use a modest increase — `-G 16M` **takes on the Moto G54; the SM-P200 refuses it and caps at
5 MiB**, replying "MAX log buffer size is 5 MiB. So set it to 5 MiB". *Corrected 16 Aug 2026: this
line read "worked on both devices", measured on two invocation forms and at 8M. On the tablet the
raise is unavailable and telemetry must be streamed to a file during the run instead.* Verify it with
`logcat -g` afterwards, and **never conclude anything from missing log lines without a
check outside the logging path.**

Caveats: one observer, one device, three short sessions, and the observer was deeply
primed about the system (blind to the assignment, but not naive). Telemetry per session was
thin — 28–79 cruise ticks captured — because the ring buffer rolled, though the applied
settings are confirmed from `target=` in the ticks.

Run it as A/B/A if the observer is willing — three sessions, the outer two the same
setting — so a "difference" claim has to be consistent rather than a coin flip.

```bash
adb shell am broadcast -a dev.spike.autoscroll.CONTROL -p dev.spike.autoscroll --ei segment 100
```

```bash
adb shell am broadcast -a dev.spike.autoscroll.CONTROL -p dev.spike.autoscroll --ei segment 250
```

**Keep the assignment away from whoever is in the room.** If the person running the test
knows which session is which, they will cue it. Ideally a third party sets the order, or
it is driven from the host machine and revealed only afterwards.

### Pending decision: raise default `segmentMs` from 100 to 250?

**Still not changed. 4c came back "not reliably discriminable"** — see above. So there is
no perceptual argument either way, and the decision falls back to the mechanical ones:
retiring the widen loop at low speeds, and 2.5x fewer IPC round trips. Both real, neither
urgent, and neither visible to a user.

**DECISION: leave the spike at 100ms.** A flat raise is the wrong shape of fix.

The widen-loop problem is speed- and density-dependent, not a property of 100ms. Against
`MIN_SEGMENT_PX = 1.0`:

| speed | density | 100ms commands | 250ms commands |
|---|---|---|---|
| 12 dp/s | 2.00 | 2.4px — clears comfortably | 6px |
| 4 dp/s | 2.00 | **0.8px — floored** | 2px |
| 4 dp/s | 3.08 | 1.23px — marginal | 3.1px |

At the 12 dp/s default 100ms already clears the floor, so 250ms buys nothing mechanical
there. The problem is confined to the slow end, and a flat raise fixes it by overshooting
everywhere else — paying stop latency at every speed to solve a problem that only exists
below ~5 dp/s.

**Production recommendation (not spike work): derive segment duration from floor
clearance.** Pick the shortest duration that commands at least ~3x `MIN_SEGMENT_PX` at the
current speed and density. That lands near 100ms at reading speed and ~375ms at 4 dp/s,
keeps stop latency minimal where it can be, makes the widen loop *unreachable* rather than
merely avoided, and generalises to densities below 2.0 — untested here, and exactly where a
hardcoded 250 would start failing again.

The IPC-count argument (2.5x fewer round trips) survives independently but is small, and
belongs against a measured battery figure rather than reasoning.

Arguments for, in order of strength:

1. **It looks Pareto, not a trade.** 250ms more than halves instantaneous velocity
   variation on the SM-P200 (12.6% → 5.3% of segment), and costs nothing measurable on the
   Moto, where jitter was already 0 outliers in 936 ticks — nothing to dilute, nothing to
   lose. This is structurally unlike the band widening, which was a genuine trade decided
   on one device's evidence while the other device paid for it.
2. **It retires the widen loop at reading speeds.** The floor bites when
   `speed_dp x density x seg_s < 1.0px`, i.e. below `1 / (density x seg_s)` dp/s. On this
   2.0-density tablet that is **5 dp/s at 100ms segments, but only 2 dp/s at 250ms**. Sub-pixel
   segments are the machinery that produced the false chain-decay signal in the first
   place; longer segments avoid that regime structurally at exactly the low speeds this
   product targets. Note this is density-dependent — on the Moto (3.08) the floor does not
   bite until ~3.2 dp/s even at 100ms.
3. **2.5x fewer IPC round trips per second**, feeding the CPU/battery budget and, via
   resident footprint, the LMKD survival argument.

Costs:

- **Deliberate-stop latency** rises to ~350ms worst case (segment + 100ms hold; the decel
  ramp is skipped below the fling threshold, so the +200ms figure does not apply at reading
  speeds). At 12 dp/s that is ~4dp — under a fifth of a line. Touch-to-stop is unaffected
  entirely: a finger cancels mid-segment, measured at 9–12ms. The deliberate path still
  matters because proximity wave and a QS tile are the controls for users who cannot
  reliably touch.

  **CORRECTED 2026-08-13 — the sentence above names two controls that group does not have
  on a tablet.** The QS tile was already excluded: a QS panel is a window, and any control
  owning a window captures the synthetic finger. And proximity is now measured **absent in
  hardware on the SM-P200** (see DEVICES.md), so on tablet-class devices the user who cannot
  reliably touch has neither. The deliberate path still matters — that part holds — but it
  matters because of the *notification control and an external switch*, not because of these
  two. Left in place because the reasoning that followed from it is still readable.
- **Rate-filter convergence** stretches from ~2s to ~5s at steady-state gain, since there
  are 2.5x fewer samples per second. The warm-up gain already collapses this to ~3 samples,
  and it only affects the first press of a run.

---

## Template

**Device:**
**Android version / API:**
**Screen (px) / density:**
**Measured slop (dp) / long-press timeout (ms):**  <!-- from the PRESS log line -->
**Build tested:**
**Date:**

### Band geometry

- [ ] 25% of height clears the status bar (no pull-down triggered on press)
- [ ] 75% of height clears the nav bar / gesture strip (no nav gesture triggered on lift)
- [ ] x = 50% width never triggers edge back-gesture
- Adjusted band if any of the above failed: ______

### Per app

| App | Speed (dp/s) | Segment (ms) | Smooth? | Re-grip visible? | Long-press fired? | Fling on lift? | Pass/Fail |
|---|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |

Suggested apps: a `RecyclerView` list, a `WebView` (browser article), a nested-scroll
app (collapsing toolbar), a `ViewPager`/pager-based app, a Compose app, a PDF reader,
one OEM system app.

### Speed sweep — 6 / 12 / 25 / 50 / 100 dp/s

| dp/s | Stalls or stair-steps? | Re-grip interval (s) | Fling on lift? | Notes |
|---|---|---|---|---|
| 6 |  |  |  |  |
| 12 |  |  |  |  |
| 25 |  |  |  |  |
| 50 |  |  |  |  |
| 100 |  |  |  |  |

### Segment sweep — RESOLVED on SM-P200 (12 dp/s)

**Jitter is fixed-cost scheduling latency, not proportional.** Absolute mean |jitter| sits
at 11–14ms across a 4x range of segment durations, so longer segments dilute it:

| Requested | Actual target | mean \|jitter\| | % of segment | max |
|---|---|---|---|---|
| 32 ms | **64 ms** (widened) | 13.2 ms | 20.6% | 28 ms |
| 50 ms | **74 ms** (partly widened) | 11.0 ms | 15.0% | 18 ms |
| 100 ms | 100 ms | 12.6 ms | 12.6% | 34 ms |
| 150 ms | 150 ms | 14.4 ms | 9.6% | 36 ms |
| 250 ms | 250 ms | 13.4 ms | **5.3%** | 38 ms |

**32ms is unreachable at reading speeds on a 2.0-density screen** — it commands 0.77px,
under `MIN_SEGMENT_PX`, so the widen loop doubles it. The floor and this sweep interact.

**The stop-latency cost is smaller than assumed.** A physical touch cancels *mid-segment*
(observed: 12ms into a 100ms segment), so segment duration does not gate the touch-to-stop
safety path at all. Longer segments only slow the *deliberate* stop — notification button
or broadcast — to (segment + 200ms), i.e. ~450ms at 250ms segments.

**Not yet decided:** whether to raise the default from 100ms. It buys real smoothness on
noisy hardware and nothing on device 1, where jitter was already 0 outliers in 936 ticks.
A per-device choice driven by measured jitter would be the principled answer and is
over-engineering for a spike.

**Caveat on n:** the logcat ring buffer rolled during each 25s run, so these means come
from the tail of each run (n = 28–428, not the full ~250–780). The constancy of absolute
jitter across five segment sizes is the robust part; treat individual means as ±2ms.

### Re-grip — the primary finding

- Median `REGRIP deadMs`: ______
- Reads as: ( ) invisible  ( ) a blink  ( ) a noticeable glitch  ( ) breaks reading
- Does the host app scroll-position jump, bounce or settle oddly across a re-grip?
- Does the re-grip get worse at higher speeds (more frequent) or is frequency irrelevant?

### Visual check — a fail here overrides clean logs

- [ ] No context menu at press
- [ ] No context menu at re-grip
- [ ] No text-selection handles or magnifier
- [ ] No tooltip or long-press popup
- [ ] No item highlighted / pressed-state stuck
- [ ] No fling or overscroll glow on lift
- With `--ez leadin false`, does long-press fire? (expected: yes — confirms the kick is load-bearing on this device)

### Cancellations

- `GEST onCancelled` count over a 5-minute run: ______
- Anything that reliably causes one (notification shade, IME, screen-off, app switch)?

### Verdict

( ) Pass — held-finger chaining is viable here
( ) Pass with caveats: ______
( ) Fail: ______

### Observations

