# Device measurements

Structured, one row per device, **not prose**. This exists because the chain-decay
threshold looks like a property of the OS build rather than the individual handset — so
once enough devices are measured, these become shipped starting values keyed on
`Build.MANUFACTURER` + `Build.VERSION.SDK_INT`, with calibration correcting from there
instead of discovering from scratch. That only works if the numbers stay comparable
across devices, so keep the columns fixed and leave cells blank rather than approximating.

Every field except the two subjective ones, **and except where noted**, comes straight out of
logcat:

- `DEVICE` line → manufacturer, model, device, api, release, build
- `PRESS` line → screen, density, slop, longPress, minFling
- `NOTES.md` stage 2 → highest good / lowest bad age
- `REGRIP` lines → re-grip dead time
- `CANCEL` lines → spurious cancel rate

## Identity and device constants

| # | manufacturer | model | device | api | release | screen px | density | slop dp | longPress ms | minFling dp/s |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | motorola | moto g54 5G | cancunf | 35 | 15 | 1080x2400 | 3.08 | 8.1 | **400** | 50.1 |
| 2 | samsung | SM-P200 (Tab A 8.0, tablet) | wisdomwifi | 30 | 11 | 1200x1920 | **2.00** | 8.0 | **500** | 50.0 |

`longPress` is bolded on device 1 as a reminder: it was 400ms, not the 500ms every
reference states, and it is user-configurable in accessibility settings.

## Sensor availability

**Provenance: `dumpsys sensorservice`, read-only — not logcat.** One of the two "except where
noted" cases; the other is *Accessibility shortcut surfaces* below. Measured 2026-08-13 on both
devices.

**This table does not feed the shipped-defaults model, and cannot.** Everything above is
recorded because it looks like a property of the OS build, so it can become a starting value
keyed on `Build.MANUFACTURER` + `Build.VERSION.SDK_INT`. Sensor presence is per-model hardware
inventory: it cannot be interpolated from a neighbouring device, and a matching manufacturer
and API level tell you nothing about it. It is here because it is a device constant, not
because it feeds that model.

| # | proximity | light | notes |
|---|---|---|---|
| 1 | **yes** — `stk3a5x_ps`, `android.sensor.proximity(8)`, on-change, wakeUp | yes | Motorola vendor type `prox_for_call(65585)` is also present; type 8 is the usable one |
| 2 | **no — absent in hardware** | yes — VEML3328 | Full inventory: accelerometer, light, Semtech grip, significant_motion, tilt_detector, light_cct |

**Device 2's absence is verified, not assumed.** An empty grep looks identical whether the
sensor is absent or the dump failed, so the full inventory was printed to tell those apart:
103 lines, "Total 6 h/w sensors", every sensor named. Absent in hardware — not disabled, and
not hidden behind a vendor alias. That check is standing rule 5 applied without being asked
for, and it is the reason this row can be trusted.

**Why the spec sheets disagree (documentation read 2026-09-02, not independently verified).** Web
aggregators list a proximity sensor for this device and appear to contradict the row above; they do
not. GSMArena covers `SM-P200` (Wi-Fi) and `SM-P205` (LTE) on one page — "Versions: SM-P200 (Wi-Fi);
SM-P205 (LTE)" — and lists "Sensors: Accelerometer, proximity" for the pair, i.e. the union across
variants. A proximity sensor's primary job is blanking the screen during a call, which exists only on
the cellular `SM-P205`; the `SM-P200` is Wi-Fi-only with no call path (already established for the
host-hazard incoming-call arm). PhoneDB lists the `SM-P200` on its own and gives "L sensor, Hall" —
light and hall, no proximity. So the sensor that differs across the listings is exactly the one whose
purpose exists only on the LTE model. Spec aggregators are not a source this project treats as
authoritative — rule 1 ranks a device reporting its own hardware above them — and this is recorded only
so the next person who spots the discrepancy finds it already explained, not to re-open the measurement.

**Confirmed via the app's own call (2026-09-02).** `getDefaultSensor(TYPE_PROXIMITY)` returns null on
device 2. It was inferred from the hardware inventory; now measured against the API the app actually
uses. `PairingActivity` (spike1-autoscroll, lines 91–92) calls
`getSystemService(SensorManager)?.getDefaultSensor(TYPE_PROXIMITY) != null` and, on null, disables the
proximity toggle and shows the "unavailable" note. On device 2 the toggle is greyed with that note
(developer-observed on the tablet, app code now present there) — so the call returned null. Optical
evidence, the developer's, matching the hardware inventory rather than a spec aggregator.

**The feature-detect-and-degrade path works — first evidence (2026-09-02, measured).** The same
observation carries a second finding, previously untested: the greyed toggle with its "unavailable"
note is `ProximityControl`'s designed **degradation running correctly on hardware with no proximity
sensor**. That was the requirement set on 13 August, when the sensor check first established proximity
could not be universal — anything built on it must feature-detect and degrade. This is the first
evidence the requirement holds. It exists only because the **shipped path** (`PairingActivity`'s own
`getDefaultSensor` check) was used to settle the null, rather than a `run-as` instrument: a `run-as`
write would have confirmed the null and shown nothing about the app's behaviour. So the disabled toggle
on device 2 is two results at once — the null return, and the degradation working.

## Accessibility shortcut surfaces

**Provenance: `settings get secure`, `dumpsys accessibility` and `dumpsys input`, read-only — not
logcat.** A second "except where noted" case. Measured 2026-08-14 (device 2) and 2026-08-16
(device 1), through the Settings UI throughout; no `settings put secure` was used, because Settings
was the modality under test and a written value would have created a state no user can reach.

**Like sensor availability, this does not feed the shipped-defaults model.** It is per-model and
per-OEM inventory: a matching manufacturer and API level tell you nothing about it. **The two devices
disagree on almost every row**, which is the useful part.

| # | nav mode | `accessibility_button_mode` | floating menu offered? | surfaces available | chooser class |
|---|---|---|---|---|---|
| 1 | gesture (`navigation_mode=2`) | **1** (floating menu) | **yes** — `accessibility_floating_menu_size` also present | floating button; two-finger swipe. **No nav-bar button exists** | `AccessibilityButtonChooserActivity` |
| 2 | 3-button (`navigation_mode=0`) | **absent** | **no** — the Settings page offers no location option, no size or transparency settings | nav-bar button only | `AccessibilitySamsungShortcutChooserActivity` |

**Whether a chooser appears is not a device property.** It depends on the invoking surface and on
whether a selection has been persisted — see *The v1 control set* in [DECISIONS.md](DECISIONS.md),
which is where that argument lives and is not repeated here. The classes above are what each device
shows *when* one appears; Samsung ships its own variant.

**Device 1's floating overlay cannot occupy the synthetic finger's column.** Released at three
positions it snapped to a vertical edge every time — touchable region `x 0..222` or `x 858..1080` on
a 1080px-wide screen, never including **x = 540**, where the engine puts the finger. 222px wide
against 318px of clearance. Vertically it is free, spanning y 166–2277. *Read from `dumpsys input`'s
touchable region rather than the window frame, which is full-screen and says nothing; each reading
taken twice. The overlay is **faded when idle and still takes touches**, so fade is visual only.*

**Device 2's absence of a floating menu is the developer's read of the Settings page**, corroborated
by five unset `accessibility_floating_menu_*` keys — weaker than device 1's positive result, since
unset is not the same as absent. **The consequence is scope, not tuning:** the overlay-in-drag-path
question can only ever be answered on device 1, so the two-device agreement that would close it is
unavailable by construction.

## Chain decay

`highest good` = oldest press age observed to lift cleanly. `lowest bad` = youngest age
observed to cancel during teardown. The gap between them is the sampling uncertainty; a
wide gap means stage 2 probing was coarse, not that the device is unstable.

| # | highest good (s) | lowest bad (s) | gap (s) | shipped cap used (s) | probe steps run |
|---|---|---|---|---|---|
| 1 | **68.07** | none found | unbounded | 120 (backstop, never fires) | 35 / 40 / 45 / 50 / 55 / 90 — all clean |
| 2 | 38.9 (travel-bound) | none found | unbounded | 120 (backstop, never fires) | none — travel binds at 40s, so ages beyond that need a slower speed |

Device 1's pre-fix bounds were 36s good / 48s bad, and those are **superseded, not
comparable**. They were measured before sub-pixel decel segments were eliminated and
reflect that bug rather than any property of the device. Do not carry them into the
dataset. If a device 2 run reproduces something like them, check `MIN_SEGMENT_PX` and the
fling-threshold skip are actually in the build before concluding the device decays.

## Behaviour

| # | re-grip dead time @12 dp/s (ms) | jitter outliers / 1k segments | typical seam (ms) | longest clean press (s) | spurious cancels / 10 min |
|---|---|---|---|---|---|
| 1 | 458 / 459 / 459 / 461 | 0 | 1–3 | **68.07** | not measured |
| 2 | 427–446 (n=7, independent of segmentMs) | **216 / 1000** | 2.0 (max 10) | 38.9 (travel-bound) | **0 per 9 min** |

## Speed accuracy

**Measure cruise, not the press average.** `cum` ÷ `heldMs` from a `LIFT` line includes
the lead-in kick and ramp — a fixed ~17dp per press that is deliberately not
rate-corrected, and which is +14% on its own over a 25s press at 4 dp/s. A press average
therefore cannot converge to zero however good the correction is. Reconstruct cruise-only
elapsed from `TICK actual` + `SEAM gap` instead.

Device 1, cruise-only, `ratecorrect` off → on (correction fed segment completion time):

| # | 4 dp/s | 6 dp/s | 12 dp/s | 100 dp/s |
|---|---|---|---|---|
| 1 | +29.6% → −3.7% | +14.8% → −1.1% | −0.2% → −2.3% | −13.3% → −7.7% |
| 2 |  |  |  |  |

Two things that read out of those numbers:

- **12 dp/s needed no correction at all** — cruise was already within 0.2%. The press
  average said +4.0%, which was entirely the lead-in. Beware the metric.
- **The residual is the seam.** At 4 / 6 / 12 dp/s the corrected error is almost exactly
  −(seam ÷ cycle): −3.7% against ~3ms on a ~78ms segment, −1.1%, −2.3%. The loop's true
  period is dispatch-to-dispatch, so feeding the filter segment completion time alone
  makes every correction a seam too small. Now fixed by feeding `actual + gap`; the
  confirming run belongs in the row below.

| # | 4 dp/s | 6 dp/s | 12 dp/s | 100 dp/s |
|---|---|---|---|---|
| 1, cycle-based ratio | **+0.5%** | **+1.5%** | **−0.0%** | −4.5% |
| 2, cycle-based ratio | not measured | not measured | **+0.2% / −0.1% / +0.2%** | not measured |

Confirmed: the seam was the whole missing term at reading speeds. Feeding the filter
dispatch-to-dispatch time instead of segment completion time took 12 dp/s from −2.3% to
zero and 4 dp/s from −3.7% to +0.5%.

The ratio **settles** at every speed — 0.991–1.009 around 1.000 at 12 dp/s, and at
100 dp/s it climbs to ~1.107 within 60 segments then holds flat. No wandering, so
`RATE_FILTER_ALPHA = 0.05` is not too fast, and no runaway despite the correction feeding
back into segment duration.

The 100 dp/s −4.5% turned out to be **entirely the first press of the run**, not a
residual mechanism. Per-press at 100 dp/s: press 1 −4.5%, press 2 −0.9%, press 3 −0.2%,
presses 4–6 within 0.1%. `timingRatio` already persists across re-grips (it resets in
`start()`, not `beginPress()`), so only the very first press of a session is mis-rated
while the filter climbs from 1.0.

A warm-up gain — `alpha = max(RATE_FILTER_ALPHA, 1/(n+1))` — closes that transient.
Verified: the ratio converges in **2 segments (~0.2s)** rather than ~20, climbing from
below with ±0.01 wobble and no overshoot. First-press error at 100 dp/s went −4.5% →
−0.7%; presses 2–6 stayed within 0.4%; 12 dp/s press 1 was +0.4%, undisturbed. The
mis-rated window is now far below anything a reader could notice, so cold start needs no
further handling.

The sample counter starts at 1, so the first sample gets gain 0.5 rather than 1.0. At
gain 1.0 the filter is not filtering at the one moment it is least entitled to trust its
input — the first cruise cycle sits immediately after the transition out of RAMP, the
segment most likely to be atypical — and the 0.60–1.50 clamp bounds that only to a
visible speed error, not a safe one. Costs ~0.1s of convergence.

General form, checked against the rest of the engine: **a filter that adopts its first
sample wholesale is not a filter yet.** `timingRatio` is the only filter here; everything
else is a direct device read, a timestamp, or an exact accumulator. `yPos` in particular
must stay unfiltered — per-tick rounding is the original stall bug.

The gain also re-opens on a **speed change**, because the ratio is speed-dependent: ~1.005
at 12 dp/s against ~1.11 at 100 dp/s on the same hardware. Any future shipped per-device
defaults would therefore need a curve over speed rather than one number — academic for
set-and-forget at a fixed speed, not academic if a slider ever ships.

**Measure within a single press.** At 100 dp/s travel exhausts every ~4s, so a 25s window
holds seven presses and six re-grips; spanning them charges ~460ms of dead time each to
elapsed with no matching `cum`. That alone accounted for 1.5 of the 6.0 points.

Correction is applied to CRUISE only, so no measurement here should include lead-in,
ramp, decel, hold, or a re-grip.

## Host apps and subjective verdict

The two columns no log can produce. `blink / glitch` is the whole question.

| # | hosts tested | long-press at margin 0? | selection UI? | re-grip reads as | verbatim |
|---|---|---|---|---|---|
| 1 | Chrome (WebView article), Files PDF preview, PDF w/ figures | no | no | **invisible to an unprimed reader** | primed: "noticeable... could bother a different type of user" / **unprimed: no mention of any bump or lost place** |
| 2 | Google Drive PDF viewer (151pp), Samsung Settings (RecyclerView), One UI launcher | not tested | not tested | **survivable but tiring over 6 min / 9 re-grips** | "a very small interference kind of the text getting stuck for a brief moment... made me feel tired"; "I could still read ok but made me more tired" |

## Coverage gaps

Pick device 2 to maximise what a single device adds: **different manufacturer and
different Android major version** from device 1.

| Axis | Covered | Missing |
|---|---|---|
| Manufacturer | motorola, samsung | Pixel/AOSP — the cleanest control against an OEM layer |
| Android major | 15 (API 35), 11 (API 30) | 12, 13, 14, 16+ |
| Density | 3.08, **2.00** | below 2.0; very high DPI |
| Form factor | phone, **tablet** | — |
| Nav mode | gesture, **3-button** | — |
| Long-press timeout | 400ms, **500ms** | a user-*shortened* one — the only risk case |
| Host surface | WebView, PDF, RecyclerView | Compose, nested-scroll toolbar, OEM system app |
| Spurious cancel rate | **0 per 9 min hands-off (SM-P200), plus an 8-item provocation pass** — notifications, shade, volume, screenshot and touchless app switch provoke nothing; a physical finger cancels in ~10ms | Untested provocations: IME appearance, screen rotation, incoming call. Not measured at all on the Moto |
| Subjective verdict | Naive reader on **both** devices; 4 logged sessions; text-size and segment-duration arms on the SM-P200 | **All perceptual evidence comes from one reader**, sensitised across four sessions. No second person has ever seen this scroll. That is the gap, not device coverage |


## Segment duration vs jitter (SM-P200)

Jitter is **fixed-cost** (11–14ms regardless of segment length), so longer segments dilute
it: 12.6% of segment at 100ms → 5.3% at 250ms. See NOTES.md for the full table. The
stop-latency cost applies only to deliberate stops; a physical touch cancels mid-segment.

## Device 2 — what it changed

**Constants mostly transfer.** Touch slop (8.0 vs 8.1 dp) and min fling velocity (50.0 vs
50.1 dp/s) are effectively identical, so those look like stable platform defaults rather
than per-device values. Re-grip cost transfers too: 446 / 434 ms against the Moto's ~458,
so ~450ms is a property of the platform, not of one handset.

**The Moto's 400ms long-press timeout was the outlier, in the safe direction.** The tablet
reports the textbook 500ms, so the lead-in kick has *more* headroom here than on the device
it was tuned against. A device reporting *less* than 400ms is the risk case, and neither
device tested is one.

**Density 2.00 exactly** — the first standard bucket tested, after 3.08 and 2.88. Band,
travel budget and lead-in all computed correctly, so nothing in the engine assumes a
density.

**Segment timing is far noisier here, and that is the real finding.** Seams stay tight
(mean 2.0ms, max 10ms) so IPC is healthy — but segment *execution* swings between 69ms and
107ms for a 100ms request. Mean |jitter| 12.1ms vs the Moto's ~0, with 21% of segments over
16ms and 8% over 30ms. `timingRatio` correspondingly oscillates in a 0.90–0.94 band rather
than settling to a point, which is the filter tracking a noisy input, not failing.

Delivered rate stays correct (+0.2% / −0.1% / +0.2%) because the filter averages it out.
**But that is an average.** A segment executing in 70ms instead of 100ms runs at 1.4x
target for its duration, and the next at 0.93x. Instantaneous velocity varies ~±35% here
against near-zero on the Moto.

Not a host confound: device 1's smooth numbers and device 2's noisy ones were both taken
in PDF viewers (Files preview and Google Drive respectively). The difference is the
hardware.

Whether that reads as micro-stutter at 12 dp/s is a question no log answers, and it makes
this the more important device for the subjective test — the Moto's smoothness may simply
not be representative of budget hardware. Likely cause is CPU scheduling on a 2019 midrange
tablet, which is closer to what many users will actually have.
