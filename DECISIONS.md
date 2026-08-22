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

**One known unresolved conflict with this rule:** the spasm-tolerant hard stop is designed as
a hold-cover gesture, which is sustained input. It is not recorded anywhere as a deliberate
exemption. Open, and in the open items table — do not read the design's existence as the rule
having been waived.

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
`tools:node="remove"`. The OS enforces all three, and both test devices confirmed them.

**They are not all the same kind of property, and the difference is load-bearing.**
`capabilities=32` and `eventTypes = 0` are **runtime** properties — computed by the OS from
the service's declared attributes, and readable only from a running service, with
`adb shell dumpsys accessibility`. The absent INTERNET permission is an **artifact**
property — a fact about the built APK's merged manifest, readable with
`aapt2 dump badging <apk>` whether or not anything is running.

Two things follow. Verifying the set takes **two checks, not one**: no single command
answers for all three, and any claim that one does is wrong. And the ratchet must read the
built artifact rather than the source, because the artifact is where the permission question
is actually settled — a dependency can merge in a permission that `tools:node="remove"` in
the app manifest does not catch.

This is the project's strongest verifiable claim and its main trust asset. Anything that
raises that bitmask spends it. `CAPABILITY_CAN_REQUEST_FILTER_KEY_EVENTS` would move it from
32 to 40 and was rejected on those grounds alone.

### 5. Before believing a result, ask what a null instrument would have produced

Eight measurements in this spike returned clean-looking data about things the instrument could
not see. **The eighth is a different species from the other seven and the rule as stated cannot
catch it:** those are instruments that ran and misled, while `NOTES.md`'s "with a 64M logcat buffer
so nothing rolled" is a **check recorded as having been run that was not run** — the SM-P200 refuses
any buffer above 5 MiB, so `logcat -g` would have returned 5 MiB and the sentence could not have been
written. A reader applying this rule finds it already satisfied and moves on, which is how it
survived every pass over these documents. *Measured 16 Aug 2026; see the caveat at `NOTES.md`'s
segment-duration A/B/A.* Verify the instrument *before* the measurement, use a positive control from the
real modality, and repeat the baseline.

**A ninth belongs beside these eight without matching the sentence above it.** The other eight
were run inside this project; this one was not. It earns its place anyway because it is the
same failure shape as the rest, on the primitive most people reach for first: a `Messenger`
posts to a `Handler` that runs *after* the binder transaction returns, so
`Binder.getCallingUid()` inside `handleMessage` is outside any incoming transaction and returns
self-uid — a clean-looking value that reads as "no identity available" rather than "you read it
in the wrong place." **Recorded from design discussion — not measured**: reasoned from
documented Handler/IPC semantics, read from the control-path identity-primitive charter
(2026-08-21) rather than reproduced here. The charter used a raw `Binder` (`onTransact`, inside
the transaction) specifically to avoid this: the raw-Binder `onTransact` returned the caller's
real uid on both test devices, where a Messenger handler would have returned self. **Reported as
a measurement by the charter, not independently reproduced or verified here**: two test devices,
per the charter (2026-08-21); this environment has no Android device to re-run it on.

| Instrument | Looked like | Actually |
|---|---|---|
| `adb shell input tap` | touches don't cancel | synthesised via InputManager; never reaches the digitiser |
| `am start` to launcher | app switches don't cancel | changes the window *without* a touch, and only the touch cancels |
| Plain A/B, one comparison | 250ms segments are smoother | unreplicated; A/B/A showed the discrimination didn't hold |
| A primed observer across sessions | later sessions felt worse | sensitisation, not the setting |
| `logcat -G 64M` | the app is dead | the log reader silently stopped returning data |
| The `VISIBLE LURCH` log line | the lead-in is an ease-in | measured a tenth of the effect; the ramp carries ~90% |
| A recorded buffer size, never checked | the capture was complete | the device refused that size; the check was written, not run |
| A check scoped narrower than the claim | HANDOVER's basis cell was accurate on its own | the row title made it a claim about all three privacy properties, which the cell could not support |
| A `Messenger` for caller identity | no identity available | `getCallingUid()` runs in the `Handler`, after the transaction returns — reads self, not the caller |

Verify the **claiming unit** — title plus basis, heading plus body — not the fragment you
were pointed at. In that case the narrow scope came from the instruction, not from the check.

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

---

## The v1 control set

**Recorded here because it was recorded nowhere.** Until now the v1 control set lived in a
frozen Play draft and in conversation, while the rest of this repo reasoned *about* it. The
external switch is the clearest symptom: it appears only inside a rejected-alternative
argument — a signature-level permission was rejected because it "breaks third-party
switch-access and AAC apps, exactly the integrations this audience depends on" — so it was
load-bearing without ever having been scoped. Same failure as the consent line: the thing
everything else depends on is the thing nobody wrote down.

**Provenance varies inside this section and is marked per item.** Most of it rests on
measurement, or on arguments recorded elsewhere in this file. Two items — the edge slider and
tilt — come from design discussion only: never prototyped, never measured. They are marked so
they do not sit at the same apparent weight as everything around them.

### The speed premise, and why this section gives no count

**v1's premise is a set-and-forget speed:** one dp/s value, chosen once, not adjusted during a
run.

Two reader observations put that premise under pressure — images have no reading rate, and
smaller text raises the line rate at a fixed dp/s. Both are recorded in full in *Threat to the
v1 premise* above, which is where the argument lives; it is not restated here. What matters
for scope is their weight: **Medium confidence, two observations, one reader.** Standing rule
3 says no design decision may be made against a single reader.

So the budget is recorded as **two intents committed — start and stop — and two contingent on
the beta: faster and slower.** The need is real and is recorded at its actual confidence. The
count is a design decision the present evidence cannot carry, and this is the document meant
to outlive the rest, so it does not get to make one on that evidence.

**The proximity result sharpens this rather than settling it.** Fewer usable input channels on
tablet-class hardware means an expansion from two intents to four is not an abstract scope
choice — it may be undeliverable on the hardware it lands on. That is an argument for settling
the *need* before the *count*, not after.

### The controls

**Start and stop by proximity, where the hardware provides it.** Not universal: proximity is
absent on tablet-class hardware, so anything built on it must feature-detect and degrade. It
cannot be the primary hands-free control. Measured — see the sensor availability table in
[DEVICES.md](DEVICES.md), and the consequences under *Proximity cannot be the primary
hands-free control* below.

**The accessibility button.** Recorded as a candidate, **not as a working control.** Measured on both
devices 14–16 Aug 2026; what was found is below, and it does not make the button available.

**Three states, and they are not three orthogonal axes.** The vocabulary matters because the earlier
version of this entry asked the question in terms of services being *registered*, which is the wrong
term and would have tested a condition the effect does not require:

- **enabled** — in `enabled_accessibility_services`, the service is bound and running
- **assigned** — in `accessibility_button_targets`, the shortcut acts on it
- **flag-declared** — `flagRequestAccessibilityButton` in `accessibilityFlags`, which is what receives
  the button *callback*

Enablement does not imply assignment; **an assignment does not survive its service being disabled**;
and assignment does not require flag-declaration — this service is assigned and ticked in Settings
while declaring no `accessibilityFlags` at all, and the platform then toggles it instead of delivering
a callback. *`accessibilityFlags` is declared nowhere in this project, so nothing here establishes
what flag-declaration is independent of.*

**Whether the button presents a chooser has no single answer.** It depends on two things:

- **The invoking surface.** A single control — a nav-bar button, a gesture — must disambiguate between
  targets and raises a chooser. A floating menu (`accessibility_button_mode=1`) enumerates targets as
  persistent icons and raises none. *Demonstrated within one device with everything else held: the
  Moto's floating menu against its two-finger swipe, same day, same two targets.*
- **Whether a selection has been persisted.** The chooser is a **configuration step, not a
  per-invocation gate**: once a target is selected, later invocations go straight to it.

**So the chooser is a first-run condition, not a standing one**, and the earlier reading — that a
start through this button lands on the chooser rather than the content — is **withdrawn** as a general
claim. It holds while no selection has been persisted.

**None of this makes the button available.** It is touch-only; there is no nav-bar button at all under
gesture navigation; and it currently toggles the service rather than starting a scroll, which a
rebuild would change. *Measured on a Samsung SM-P200 (API 30, One UI, 3-button) and a Moto G54
(API 35, gesture). No cross-device same-surface comparison exists, so OEM and API level are not
excluded as factors — they are merely not needed to explain the divergence.*

**DEFERRED TO 1.1, not dropped. Decided by the developer, 16 Aug 2026**, after the measurements above.

**Three facts carry the deferral**, and all three survive this run: there is no nav-bar button under
gesture navigation; it is a touch control, so it does not serve the segment that made an external
switch mandatory; and wiring it is a rebuild, since `accessibilityFlags` is declared nowhere and the
platform therefore toggles the service instead of delivering a callback.

**Two findings landed on the other side and are recorded so the deferral is not re-argued from a
one-sided set.** On floating-menu devices the control is a persistent, user-positioned icon — a
better target for limited reach than a notification, which requires a drag from the top edge, the
exact motion this app exists to remove. And the icon **cannot** be parked in the synthetic finger's
column: measured on the Moto, it snaps to a vertical edge, 318px clear of x = 50%, so it does not
create the hazard it might have. A faded overlay also stays touchable, so it does not stop working
when it stops being obvious.

**One fact was refuted and must not be reintroduced.** *Multi-target selection is touch-and-hold*,
carried at the documentation ceiling and read as a standing-rule-1 conflict, **failed on all three
surfaces tested** — nav-bar button (tap and hold both), floating menu (neither), swipe (the swipe;
hold is not a distinct gesture). It is not a rule 1 conflict, and the set of facts against the button
is three rather than four.

**What would reopen it:** the rebuild that wires the button, which is a prerequisite for it doing
anything other than toggling the service — and which the control-path charter would have to want for
its own reasons, since the three facts above are unaffected by it.

**An external switch, via a documented entry point — required, not optional.** v1 requires a
documented external-switch entry point. **A specification now exists and is unbuilt** —
`SWITCH-CONTRACT.md` in the spike repo, written 2026-08-20 in the same pass as the receiver
fix, deliberately: closing every path a third party can use and *then* discovering the switch
needs a hole in it is the failure it exists to avoid. It proposes an exported, pairing-gated
receiver whose caller is authenticated by `PendingIntent` creator identity rather than by a
bearer token, because a `BroadcastReceiver` is never told who sent the broadcast.

**Adopting it is the control-path charter's call, and nothing here is settled by its
existence.** The consent surface is deferred inside the document itself, and its command
vocabulary deliberately adds nothing to the two intents recorded above.

*This previously read "**The contract is undefined**: which intent, what it carries, and who
documents it are all control-path charter output." The first two now have a proposed answer;
who documents it is still open.* The receiver-security half of the blocker is cleared — the
exported `CONTROL` broadcast was closed 2026-08-20 and replaced by a non-exported receiver
that reads no extras — while the pairing mechanism itself remains unbuilt. All of this is
required because of the proximity result: on a tablet, a user who cannot reliably touch the
screen has no other hands-free path.

**Identity is not authorisation, and the mechanism above already reflects that.** *Inferred.*
An attested package (or uid) is identity, not authorisation. A provider- or uid-based
control-path design must add a signature check as a second step — resolve the caller's signing
certificate (`PackageManager.GET_SIGNING_CERTIFICATES` / `hasSigningCertificate`) and compare
against the paired signer — because a package name is not a security boundary on its own.
`SWITCH-CONTRACT.md` (spike repo) already keys pairing on uid **and** signer — `getCreatorUid()`,
then the paired-set check, then `hasSigningCertificate` before granting, revoking the pairing if
the signer ever changes — so this finding aligns with the shape of the existing design. It does
not close the gap, though: the identity step that shape depends on, `getCreatorUid()` /
`getCreatorPackage()`, is exactly what `SWITCH-CONTRACT.md` §12 already flags as unexercised —
their "behaviour was not exercised", called "the weakest link in the argument for the fix" — so
this finding sharpens why that verification gap matters rather than closing it. *Checked by
reading `SWITCH-CONTRACT.md` directly, not by running it; there is no Android device in this
environment to test a real signer-mismatch case against.*

**Rewind-on-resume.** On restart, the scroll resumes **~120dp above where it stopped**, so the
reader re-reads a few lines rather than resuming mid-sentence at a point they may already have
passed. **A fixed distance, not a computed one** — the app cannot know line height, because
knowing it would mean reading the screen (standing rule 4), so it cannot resume by lines.

**The edge slider — deferred to 1.1, not dropped.** *Recorded from design discussion, not
measured.* Two independent reasons:

1. **The design that reached rule 1 was a held slider, and there is no surviving non-held
   design.** What is deferred is the idea of an on-screen edge control, not one mechanism.
2. **Play exposure.** `SYSTEM_ALERT_WINDOW` combined with an `AccessibilityService` is the
   signature of tapjacking malware, and draws scrutiny on precisely the submission this
   project cannot afford to lose. **This compounds with the window-capture finding** — an
   overlay owns a window, so it would capture the synthetic finger as well. That second
   argument arrived later and independently of the first, and is recorded because the repo
   previously carried only the rule-1 reason.

**Tilt — considered and rejected.** *Recorded from design discussion, not measured. Never
prototyped.* Two structural reasons, either sufficient:

1. **It fails entirely on a mounted device**, which is a named v1 audience.
2. **It demands controlled wrist rotation**, which is precisely the capability this app
   assumes absent.

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
  reasoning inline. Added 2026-08-13 after closure — *corrected from 2026-08-12 on 16 Aug 2026;
  commit `c6681cd` is dated 2026-08-13 11:06:50 and the file mtime 11:04:58. The wrong date misled a
  later run, which found a 12 Aug APK that looked as though it should carry the flag and did not,
  because the edit came after it* — the one code change made to a closed
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
not test that the app works; it fails the build if an established guarantee loosens.
Weakening it then requires editing the check itself, which appears in review as a deliberate
act rather than a diff nobody noticed. Each assert below is a one-line change to lose, looks
harmless in a diff, and spends something expensive.

**Scope: declarations and shipped defaults.** The first three asserts are declarations. The
fourth is a shipped default, admitted here deliberately — a guarantee that can be lost in one
line and whose only other enforcement is remembering belongs in a ratchet by the same
argument, whether or not it lives in a manifest.

**Read from the built artifact, never the source.** Demonstrated on this repo 2026-08-13:
the source contained `isAccessibilityTool="true"` while the APK did not, because the APK
predated the edit — a source check would have passed on a build that shipped without it.
Source grepping fails in both directions: it matches the attribute named in a comment, and
it misses anything a dependency merges into the manifest.

| Assert | Read from | How |
|---|---|---|
| `isAccessibilityTool="true"` present | the accessibility-service XML inside the APK — **not at a fixed path**, see below | `aapt2 dump xmltree <apk> --file <path>`, where `<path>` is found by root element, not by name |
| `canPerformGestures=true`, `canRetrieveWindowContent=false`, and the absence of `canRequestFilterKeyEvents`, touch exploration and magnification | same file, same command | these are the *inputs* that produce `capabilities=32` |
| No `INTERNET` in the **merged** manifest | APK badging | `aapt2 dump badging <apk>` — merged, so a library's manifest can introduce one that `tools:node="remove"` in the app manifest does not catch |
| **No configuration extra is reachable in a release build** — not "`repressOnCancel` cannot be enabled", see below | the merged manifest **and** the dex | two reads: nothing but the accessibility service is `exported="true"`, and no debug-channel class or `Intent` extra accessor appears in `classes*.dex`. **Mechanism decided and implemented** |

**The fourth assert used to be a different shape from the first three. It no longer is, and
it is no longer about a flag.** It previously read *"`repressOnCancel` cannot be enabled in a
release build"*, with the note that this is a code property, so asserting it means inspecting
compiled code — *"or, more cheaply, removing the broadcast extra from release builds entirely
and asserting **that**."* The cheaper option is what was built.

**Naming the flag was the bigger error, and it is why the assert is now stated as a
property.** The spec named `repressOnCancel` because it defeats touch-to-stop. Nobody named
`leadin`, which at low speed drops cruise under the long-press threshold and makes the
synthetic finger long-press the host app, navigating the reader out of it — equally a safety
property, and **not known to be one until 19 Aug 2026**. Enumerating flags keeps losing that
race. A release build that accepts no configuration at all cannot be weakened by an extra of
any name, remembered or not. The original reasoning survives unchanged: a build shipped with
these reachable is an app the user cannot stop by touching the screen, and the alternative to
a ratchet is remembering.

**Mechanism.** The debug control channel lives in a `debug` source set, so it is absent from
a release APK's merged manifest *and* its dex, and both absences are artifact reads. The
allowlists in the script are the ratchet: landing the switch contract means editing them,
which appears in review as a deliberate act rather than a diff nobody noticed.

*Implemented and verified 2026-08-20 — six assertions hold on the release APK and three of
them fail on the debug APK, which is the control that shows they discriminate.
`tools/declaration-ratchet.sh` in the spike repo, with its own "what this does not catch"
note. Recorded from running it, not sourced from this repo.*

**Correction, and the reason the first two rows changed.** This table used to name
`res/xml/autoscroll_service.xml` as both the location and the command argument. **A release
build renames the resource** — in the first release APK ever built from spike 1 it landed at
`res/2n.xml` — so `aapt2 dump xmltree <apk> --file res/xml/autoscroll_service.xml` returns
`error: failed to find file` against the exact artifact the assert exists to check. A debug
APK does keep the original path, which is why the spec read correctly for as long as nobody
ran it against a release build.

The failure mode matters more than the path. **It fails by finding nothing**, which is
indistinguishable from a clean pass unless the check treats absence as an error — standing
rule 5, inside the tool built to enforce rule 4. Locate the file by its root element
(`accessibility-service`) instead, which is stable against renaming and needs no resource-id
arithmetic, and make an unreadable APK fail rather than pass.

*Measured 2026-08-20 by running it — aapt2 37.0.0 against `app-release-unsigned.apk`, AGP
9.0.0, `isMinifyEnabled = false`. Run in a session that had the Android SDK available, unlike
the environment `CLAUDE.md` assumes; not sourced from this repo. A working implementation is
in the spike repo at `tools/declaration-ratchet.sh`.*

**Correction worth carrying:** earlier notes said "assert `capabilities == 32` from the built
artifact". The bitmask is computed by the OS at runtime and is not in the APK. From the
artifact you assert the attribute set that yields it. Asserting the literal `32` needs a
device — `dumpsys accessibility` — which is an instrumented test requiring hardware in CI,
worth having eventually because it verifies the OS agrees with your reading of the
attributes.

**Note on the duplication.** This section and standing rule 4 both state the
runtime-versus-artifact distinction. That is deliberate — the ratchet spec has to be readable
on its own by whoever builds it — but restating a definition in two places is exactly how the
"one command" claim drifted in the first place. **If it drifts again, the ratchet spec is the
one that should point at rule 4 rather than restate it.** Rule 4 is the definition; this is
its application.

**What it will not catch.** Anything that is not one of the asserts above. It would not
notice the engine reaching the node tree by some other route, or a control path letting a
third party start the scroll. Those need review and tests, not a ratchet.

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

### applicationId and namespace: `io.github.dmprieto.reading`

**Amended 14 Aug 2026.** Recorded first as `io.github.dmprieto.keepreading`, reopened, and
settled on `io.github.dmprieto.reading`. **The namespace is unchanged; only the leaf segment
moved.** The superseded candidate and the reasoning that admitted it are kept below under
*Superseded* rather than overwritten — it failed gate 4, and *how it got past gate 4* is the
more useful half of the record.

**Correction first: the naming criterion was never recorded anywhere in this repo.** It was
assumed to be in this file and it is not. Nor was `dev.spike.autoscroll` ever a rejected
candidate with recorded reasoning — it was a placeholder in a draft of the Play declaration
package, a document that is not in this repo either.

**This is the third instance of the pattern named in *The v1 control set* above**: the thing
everything else depends on is the thing nobody wrote down. The v1 control set was the first,
the consent line the second, and this is the third.

**This is a new identifier for the production tree, not a rename.** `dev.spike.autoscroll`
stays exactly as it is everywhere it appears in this repo. Every `adb` command, `pidof` call
and broadcast action recorded here was run against a build carrying that package name;
rewriting them would make the record describe a build that never existed.

**The production identifier is under the opposite rule**, and the difference is why the
amendment above could be swept and the placeholder cannot. Nothing has ever been executed
against `io.github.dmprieto.reading` or against the candidate it replaced: they exist only as
recorded intent, so they are updated wherever they appear.

**The criterion — four hard gates.** An identifier that fails any of these is out, without
judgment being involved:

1. **Derives from a namespace the developer controls** and can keep for the life of the
   listing, with the privacy policy and verification page hosted inside it.
2. **Is a legal Java package** — lowercase ASCII, no hyphens, no leading digits, no Java
   reserved word as a label. The TLD trap: `.do`, `.new` and `.int` produce illegal first
   segments; `.co`, `.dev`, `.io`, `.org` and `.app` are fine.
3. **Encodes only settled decisions.**

   *Note, added after this section was reopened.* **The leaf should be forgettable.** Anything
   meaningful in it is a decision, and decisions get revisited. This is the generalisation of
   "never encode an unsettled decision" and it is the stronger form: `reading` is settled and
   still should not have been the interesting part of the name. The evidence is this section's
   own history — closed and reopened twice, both times because the leaf carried meaning.
4. **No collision** — not an existing Play package, not a trademark, not a domain with a prior
   owner who shipped software under it.

**None of these four is satisfied by relocating the finding.** Every one of them can be dodged
the same way: a namespace-control problem redirected to "we will host it elsewhere", a legality
problem to "we will alias it", an occupancy problem to the display name. The finding moves, the
candidate survives, and the gate appears to have been respected. Gate 4 was lost exactly this
way — see *Superseded* below.

**And four judgment tests:**

5. **Must not read as automation.** The declaration package ranks "automation tool wearing an
   accessibility badge" as its highest rejection risk and bans *automate*, *automatic* and
   *macro*. `auto-` is the same pattern, and the applicationId is publicly visible — in the
   Play URL, in App info, and in `dumpsys` output.
6. **Must survive transcription.** It appears in output this project asks sceptical readers to
   type.
7. **Capability, not diagnosis.** No condition names, no clinical vocabulary, no
   "assist"/"helper" framing. The store copy defines the audience by what swiping costs them;
   the identifier must not undo that.
8. **No English wordplay.**

**Why the placeholder fails: gates 1 and 3, and test 5.** `dev.spike` is not a namespace
anyone controls, so nothing can be hosted inside it (gate 1). `spike` names a phase closed
2026-08-12, and `autoscroll` names a mechanism whose premise this file records as unresolved —
the single-speed premise is under threat, the hands-free path is PENDING (gate 3). And `auto-`
is the prefix test 5 exists to catch. Its *shape* was never the problem: it is structurally
identical to `com.google.android.marvin.talkback`.

**Refinement to gate 3.** Established accessibility apps do name mechanisms —
`com.google.android.marvin.talkback`, `com.google.android.apps.accessibility.voiceaccess`. So
the rule is not "never name a mechanism", it is **"never encode an unsettled decision"**.
Google could name theirs because theirs were settled at ship time; this file records the
single-speed premise as a threat to v1 and the hands-free path as PENDING.

TalkBack is also the cautionary case in the other direction. That identifier carries a dead
research codename (`marvin`) plus a product name that is now one of three features inside
"Android Accessibility Suite" — and the cost of the mismatch has been zero. **Identifier and
branding are permitted to diverge**; `com.letsenvision.assistant`, shipping publicly as "ally",
does it deliberately.

**What these two precedents do not support.** One diverges into a dead codename, the other into
a generic word. **Neither identifier carries another live product's brand in the same
category**, so neither licenses a leaf that does. They were cited for exactly that when
`keepreading` was admitted, and it is a proposition they cannot carry: they support the *shape*
of the divergence, not the instance.

*The third-party package names and brandings above — TalkBack, Voice Access, Envision — are
recorded as supplied, 13 Aug 2026, and not independently verified. Nothing here depends on
them being exact: they are precedent for a permitted shape, not evidence for a measurement.*

**Superseded: `io.github.dmprieto.keepreading`.** Recorded 13 Aug 2026, rejected 14 Aug 2026.
**It fails gate 4**, on a finding this section already carried: `com.emspx.keepreading` is a
live Play app branded "KeepReading" in reading assistance, and Keep Reading, S.A. de C.V. is a
company with a Play developer presence. **The argument for changing it, in the form it was
made and no stronger:** an irreversible field should not carry a string this same section says
must not be used elsewhere, when a clean alternative costs nothing. Nothing is published, so
the change is free now and unavailable after first publication.

**Correction: how gate 4 was read — not a change to gate 4.** The gate as written already
covers brand occupancy: *not an existing Play package, not a trademark, not a domain with a
prior owner who shipped software under it.* It was read down to mean package collision only,
which is narrower than what is recorded and narrower than what the gate is for. **The criterion
did not need extending; it needed applying.**

The mechanism of the error is worth keeping, because it is available to any gate. The occupancy
finding was found, understood and then **routed to the display name** instead of failing the
candidate. Gate 4 governs the identifier — the criterion is a criterion for the applicationId —
so redirecting an occupancy finding to a different field exempts a candidate from a gate while
appearing to respect it.

**Rejected alternatives:**

- **A registered custom domain.** Play requires a reachable privacy policy URL for the life of
  the listing, so the domain becomes a renewal that can never lapse — for a solo developer with
  no entity and no revenue. A lapse someone else picks up leaves the identifier reverse-DNSing
  to a stranger's site, for an app whose whole claim is that it is checkable.
  `io.github.<handle>` removes that failure mode and points at the source rather than a
  marketing site. **Residual risk accepted and named:** control is delegated to GitHub, and the
  handle is encoded permanently.
- **A product-named leaf.** The app name is not chosen, and a leaf chosen now fixes a permanent
  identifier to a name that may not survive the beta.
- **A coined or non-English word from a "helping/assisting" field.** Fails test 7 — a word
  meaning "helper" inverts the social-model framing the store copy rests on.
- **A phonetic respelling** (e.g. `kiipreading`). Likelihood-of-confusion analysis covers marks
  that sound alike, so it gives no trademark protection while potentially evidencing intent; it
  breaks transcription in the verification flow; and deliberate near-miss spellings
  pattern-match to typosquatting, in a category already under extended review.
- **`reader`.** Passes every gate and test, and was rejected on two avoidable frictions. It
  implies text-to-speech, which this app does not have and which the nearest comparable does —
  in a string that appears in the verification output a reader is asked to check. And it is the
  same leaf as `com.google.android.accessibility.reader`, which is the app the differentiation
  argument is about.
- **`longform`.** Fails gate 4. Longform.org is a long-running longform-journalism publication
  that shipped a Longform iOS app from 2012 until it was withdrawn in 2018, and an unofficial
  "Longform Reader" remains on the App Store: same category, live brand, the same failure mode
  as `keepreading`. *Source: longform.org, Longform's Medium post on withdrawing the app, and
  App Store listings, checked 14 Aug 2026. Not independently verified.*

  **Kept in the file although rejected**, because it is the positive control on the corrected
  reading of gate 4: it was the first non-generic candidate tried after that correction, and
  the check fired.

**`reading`, against the recorded criterion.** It passes every gate and every test. By the
numbering above: **gate 1** is untouched, since the namespace did not move. **Gate 2** — a legal
Java package. **Gate 3** — encodes only the settled purpose: no mechanism, no control surface,
no speed model. **Gate 4** — a generic activity noun, so there is nothing to collide with.
**Test 5** — it names an activity a person does, not a thing software does. **Test 6** — a
single common word, trivially transcribed from verification output. **Test 7** — no diagnosis,
no helper framing. **Test 8** — not wordplay.

**The gate 4 result has a different provenance from the others, and it must not be written as
though it were a search.** No check of any kind was run on `reading`, and none was needed —
because the gate's two clauses are each satisfied structurally, by different mechanisms:

- **Brand occupancy.** A generic activity noun describing the category it appears in is not
  ownable as a brand within that category, so there is no occupancy to find. The provenance
  here is the reasoning, not a search, and it is unverifiable by construction rather than by
  omission.
- **Package collision.** `io.github.dmprieto.*` is namespaced to a handle nobody else holds, so
  an exact-string collision is not something another developer can plausibly produce. Nothing
  further is available now in any case: *Permanence* below records that the only authoritative
  collision check is Play Console at first bundle upload.

So gate 4 is satisfied twice over by structure, and this records which structure does which.
That is a narrower and more durable claim than "gate 4 passed".

**That is the main reason for this choice, and it is structural rather than aesthetic. A
generic leaf makes gate 4 self-satisfying** — it removes the failure mode instead of clearing
it once. No occupancy check has to be repeated on the next candidate, in a year's time, or by
the next reader who reopens this section.

**Display name — separate, and deferred.** `com.emspx.keepreading` is a live Play app branded
"KeepReading" in reading assistance, and Keep Reading, S.A. de C.V. is a company in Guadalajara,
Mexico with a Play developer presence. **The word is occupied in this category, so it is
avoided in both fields** — it failed gate 4 for the identifier, as recorded above, and
**`keepreading` remains unusable as the display name.** The display name itself is still an
open decision, deferred, and nothing here settles it. *Play listings, checked 13 Aug 2026;
read from the listings, not independently verified.*

**Permanence.** Play package names are globally unique. A deleted app with zero lifetime
installs frees its name; with any lifetime installs it is locked out permanently, including for
the account that created it. The only authoritative collision check is Play Console at first
bundle upload. **The identifier is irreversible at first publication, not first upload.**
*support.google.com/googleplay/android-developer/answer/16483176, checked 13 Aug 2026, not
independently verified.*

### Play developer account: personal, and creation deferred

**Personal account, not organisation.** The organisation route needs a registered business and
a D-U-N-S number. Google's own pages disagree on timing — the verification FAQ says up to 28
days, Play Console help says up to 30; **the disagreement is recorded rather than resolved**,
since picking one would be inventing a number. Either way it is weeks plus an entity that does
not exist, to avoid a closed test that should run anyway. *Both figures as supplied and dated
13 Aug 2026, neither independently verified — the disagreement is the recorded fact, so neither
number is treated as authoritative.*

**What creating the account actually starts.** *All three items recorded as supplied and dated
13 Aug 2026; not independently verified in this session. The third carries its own source.*

- **Identity verification is not a deferred deadline for new accounts** — it happens at
  onboarding. The widely-circulated material about choosing a deadline, completing it 60 days
  early and requesting a 90-day extension applies to accounts created **before September
  2023**. It does not describe this path and will mislead anyone planning around it.
- **The 12-tester requirement applies, and cannot be banked.** It applies because this will be
  a personal account created well after 13 November 2023. It is a state you must be *in* when
  you apply for production access: at least 12 testers continuously opted in for the previous
  14 days. Creating the account early buys nothing. **This is the definition of the
  requirement**; *The Play testing requirement is the instrument, not an obstacle* below
  draws the scheduling consequence and points here rather than restating it.
- **Inactivity closure is the real clock.** An account created more than a year ago that has
  never submitted an app for review is closed, and the fee is not refunded. The documented
  remedy is to verify account email and phone and publish something — internal app sharing,
  internal testing or closed testing are explicitly sufficient. **At least one recent developer
  report describes an account flagged despite a closed-testing release**, so the remedy is not
  bulletproof. *support.google.com/googleplay/android-developer/answer/11605267, checked
  13 Aug 2026, not independently verified.*

**Android developer verification: Colombia is not in the first wave.** Enforcement begins
30 September 2026 for users in Brazil, Indonesia, Singapore and Thailand across seven
participating stores; global expansion is stated for 2027 with no announced date. Enforcement
is device-side, so it concerns **where the app is installed, not where the developer is** — and
publishing through Play satisfies it, since the programme targets sideloading and third-party
stores. *support.google.com/android-developer-console/answer/16561738, checked 13 Aug 2026,
not independently verified.*

**No consequence for the beta — recorded so nobody re-examines it.** Closed testing distributes
through Play, Play is a participating store, and a developer publishing through Play is verified
by that route. A tester inside one of the four launch countries after 30 September 2026 is
unaffected. There is no scope implication and nothing to plan around before the 2027 expansion,
which has no announced date. An unanswered adjacency here reads like an unexamined risk, which
is why the negative is written down rather than left implicit.

**Trigger for creation:** spike 2 has delivered a control surface **and** the exported-receiver
fix has landed **in full — receiver closed *and* pairing built.** The receiver was closed
2026-08-20; **the pairing mechanism is unbuilt, so this trigger is not yet met** — see
*Control surface: the receiver must not ship exported — CLOSED*, which records the developer's
decision that the complete fix, not the closure alone, is what these triggers require.

**Worth doing now, ahead of the trigger:** make the identity document, address and payment card
carry the same name in the same form. Mismatched or unsupported documents are the primary
documented cause of verification failure.

### Upload key: decided, generation deferred

**This is a decision with a trigger, and it does not belong in [PARKED.md](PARKED.md).** Parked
items wait for a reason that may never arrive. This waits on a known prerequisite and is
certain to happen.

**Correction to a prior assumption.** The upload key is **recoverable**: if lost, the account
owner requests a reset through the Play Console help form and supplies a new
`upload_certificate.pem`, which affects neither the app signing key nor existing users. The
unrecoverable key is the **app signing key** — and by default the app is enrolled at first
upload in quantum-ready hybrid signing with Google-generated keys, so Google holds it and it
cannot be lost. Two standing instructions follow:

- **Do not opt out of the default.** Play Console offers "provide a copy of your app signing
  key". Taking it moves the unrecoverable key onto a developer laptop and creates a
  catastrophic scenario that otherwise does not exist.
- **The real safeguard is the Google account**, since the reset path is account-based. Enforce
  2-Step Verification. **Not deferred — do this immediately**, independent of everything else
  here.

*support.google.com/googleplay/android-developer/answer/9842756, checked 13 Aug 2026, not
independently verified.*

**The command:**

```
keytool -genkeypair -v \
  -keystore reading-upload.jks \
  -storetype PKCS12 \
  -alias upload \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -dname "CN=<name or project>, O=reading, C=CO"
```

- **`-storepass` is omitted deliberately** so keytool prompts. A password on the command line
  lands in shell history.
- **PKCS12** is keytool's default since JDK 9; JKS is legacy. PKCS12 uses one password for
  store and key and will not accept separate ones.
- **2048** is Google's documented minimum and what the reset flow expects. Larger buys nothing,
  since Google re-signs with its own key.
- **10000 days ≈ 27.4 years.** Android's signing guide recommends at least 25 years and
  requires a validity period ending after 22 October 2033 for Play.
  *developer.android.com/studio/publish/app-signing, checked 13 Aug 2026.*
- **`-dname` is baked into the certificate permanently.** **Corrected 14 Aug 2026: it is not
  publicly readable.** It goes into the *upload* certificate, and under Play App Signing that
  certificate is stripped and replaced before anything reaches a user — what users can inspect
  is Google's generated app signing key, per the standing instruction above not to opt out. So
  the DN is visible to Google and Play Console, not to the public. *Reasoning from what this
  file already records about Play App Signing; not independently verified, and it does not
  hold if the default is ever opted out of.* The advice is unchanged and rests on permanence
  rather than exposure: no street address, no phone number.

**The keystore filename and the `-dname` organisation followed the identifier when it was
amended**, on 14 Aug 2026: both read `keepreading` until then. Generation is deferred, and the
13 Aug flag-set run left no keystore on disk — *confirmed by the developer, 14 Aug 2026*. So no
keystore carrying either string exists: they are recorded intent, and the rule stated under
*applicationId and namespace* puts recorded intent on the update side. `dev.spike.autoscroll`
stays because builds carrying it actually ran; nothing carrying this ever will.

*The flag set was executed against OpenJDK 21 on 13 Aug 2026 and produced a `PrivateKeyEntry`,
`SHA384withRSA`, expiry December 2053. Run by the developer and recorded as supplied — there is
no JDK, keystore or Android toolchain in the environment these documents are edited from, so it
was not re-run here. **That run confirms the flag set**, which is a fact about `keytool`'s
arguments and survives the leaf changing; it does not attest to the two values amended above.*

**Backup verification is the deliverable, not the generation.** Seven steps, and the scenario
they are testing is "the laptop is gone":

1. **Two locations in different failure domains.** Not two folders on one laptop; not cloud
   storage plus a password manager behind the same account.
2. **Password in a password manager** whose own recovery kit lives somewhere neither location
   is.
3. **Record the SHA-256** from `keytool -list -v -keystore <file> -alias upload`. The
   fingerprint is public and safe to write down.
4. **Restore from each location independently** into a clean directory, ideally on another
   machine, retrieving the password from the manager rather than from memory. Compare SHA-256.
5. **Prove the private key is usable**, not merely that the file opens:
   `keytool -certreq -alias upload -keystore <file> -file /tmp/check.csr`. Delete the `.csr`.
6. **Repeat for the second location.** Two backups where only one was tested is one backup and
   one hope.
7. **If any step needed something that only existed on the laptop, the test failed** regardless
   of what the fingerprint said.

The certificate for a reset request comes from
`keytool -exportcert -rfc -alias upload -keystore <file> -file upload_certificate.pem`.

**The written note is the artifact.** Contents: purpose and applicationId; creation date; alias,
format, algorithm, key size, expiry; SHA-256 fingerprint; both keystore locations and how to
reach each; where the password lives and how to recover access to *that*; the two verification
commands and what output proves success; a "last verified" date with a tickbox per location;
and the reset path if lost. **The password never appears in the note.** Re-verify periodically —
cloud accounts close and drives die, and a backup verified once and never again is one that
fails when it matters.

**Why this surfaces late.** The upload key must exist **before the first bundle upload**,
because Play binds the upload certificate at that moment. The keystore must never be committed:
`.gitignore` it, and keep the password out of any `gradle.properties` inside a repository.

**Trigger: the same as the account** — spike 2's control surface and the exported-receiver fix.

### The nearest comparable app, and what it does that this one cannot

Recorded here because this is where the distribution and differentiation reasoning lives; the
store copy itself is in the Play declaration package, outside this repo.

Google ships **Reading mode** (`com.google.android.accessibility.reader`), an
`AccessibilityService` for people with low vision, blindness and dyslexia: a decluttered reading
view, text-to-speech, control over contrast, font and spacing, quick-settings integration, and
stated TalkBack compatibility. It is the nearest comparable, and it comes from Google itself.

**Its Play listing's permissions notice states that, as an accessibility service, it can observe
the user's actions and window content.**

**The differentiator: the closest comparable accessibility reading app, shipped by Google, reads
the screen. This one structurally cannot** — `canRetrieveWindowContent="false"` and
`eventTypes = 0`, yielding `capabilities=32`, confirmed on both test devices. That is sharper
and more concrete than anything currently in the drafted store copy, and it belongs in the
listing. **Standing rule 4 is the definition of that property set** — including why verifying it
takes two checks rather than one — and is not restated here.

**Point at rule 4; do not restate the set.** This is the permanent shape, not caution for one
pass. An earlier draft of this section listed the properties and claimed they were verified in
one command, which is the exact claim rule 4 exists to kill and the one this repo has already
had to clear three times. Even a *correct* list of the runtime-only properties invites someone
to append "and no INTERNET" later, which is how it drifted the first time: **a list wants
completing, and a pointer does not.**

**Provenance ceiling on the comparison — record it and do not exceed it.** The claim about
Reading mode is a **self-description**, read from that app's own Play listing on 13 Aug 2026. It
is not a runtime property read from `dumpsys` and not an artifact property read from
`aapt2 dump badging`, so under standing rule 4 it is weaker than either. A listing's permission
notice is written by the developer and describes what the service *may* do, which is not what
its declared attributes *grant*.

**It must be verified from the artifact or the running service before this comparison appears
in the store listing or anywhere else public.** Until then it is a listing quote, and the
listing is the one place it must not be used unverified. In the open items table below, with
the checks that would settle it.

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

### Proximity cannot be the primary hands-free control

Measured 2026-08-13, both devices — see the sensor availability table in
[DEVICES.md](DEVICES.md). Proximity is present on the Moto G54 and **absent in hardware on the
SM-P200**. It fails on tablet-class hardware, so it cannot carry the hands-free path alone.

Consequences, all of them scope rather than tuning:

- **Anything built on proximity must feature-detect and degrade.** Not an implementation
  detail — a control that silently does nothing on half the hardware is worse than one that is
  absent, because the user cannot tell which they have.
- **The external switch moves from optional to required.** Recorded in *The v1 control set*
  above, with its contract still undefined.
- **The notification control is now load-bearing for a group it was not before**, which raises
  the priority of its two recorded defects: the unexplained no-broadcast bug on the Start
  action button, and the inverted target sizing — large inert body, small live button, exactly
  backwards for this audience.

**The gap, precisely.** Users who *can* touch the screen but for whom repeated swiping hurts
remain served on any hardware by the notification control. Users who cannot reach or reliably
touch the screen — mounted device, severe weakness, spasticity — are served on a phone by
proximity, and on a tablet **by an external switch only**. Mounted tablets are common in
wheelchair setups, so this lands on a real segment rather than a hypothetical one.

### PENDING — which hands-free path v1 takes

**Undecided. Recorded, not resolved, and deliberately without a recommendation.** Two options:

1. **Proximity on phones, external switch elsewhere.** Tablet users without a switch are
   served by touch controls only, documented as a limitation.
2. **Pursue the light sensor first**, as a windowless control present on both devices.

This is a scope decision, not a measurement. What is known about option 2 — and what would
have to be true for it to work — is in the open items table.

### Control surface: the receiver must not ship exported — CLOSED 2026-08-20

**This entry is the definition of the receiver's status. Every other mention points here**
— rule 2 — because two entries in this file disagreed about it for the length of one commit,
and a stated resolution is what stops that recurring.

**What it was.** `RECEIVER_EXPORTED` meant any installed app could start and stop the scroll,
with no permission and no user interaction. The extras were not merely configuration:
`repress=true` defeats touch-to-stop, and `leadin=false` with a low speed makes the synthetic
finger long-press the host app and navigate the reader out of it. Both measured on hardware.

**What closed it.** The runtime-registered receiver is gone. Two manifest-declared receivers
replace it: a non-exported one that reads **no extras at all** — the command rides in the
action — driven only by the app's own notification, and a debug channel carrying every tuning
extra, in a `debug` source set and therefore absent from a release APK's merged manifest *and*
its dex. *Verified from the built artifact 2026-08-20 and on two devices; the ratchet asserts
both absences. Spike repo, committed and pushed.*

**What is still open, and it is the reason this heading is not simply deleted.** The recorded
plan was *"channel off by default plus a per-install pairing token"*. Off-by-default and the
pairing mechanism are **unbuilt** — what shipped closes the hole rather than opening a
controlled door through it. The door is specified in *The v1 control set* above; a v1 with no
third-party entry point is not a v1.

The two rejected alternatives stand unchanged: a signature-level permission breaks
third-party switch-access and AAC apps, exactly the integrations this audience depends on, and
routing control through key-event filtering raises the capabilities bitmask — standing rule 4.

**The deadline that applied to the exported receiver now applies to the pairing mechanism**:
before any build reaches a device we do not control, including the closed beta. Testers are not
a safe interval, and a beta build is a shipped build for this purpose.

**Developer decision, 2026-08-20 — the Play-track pause holds until the pairing mechanism is
built, not merely until the receiver was closed.** Every trigger recorded as "control surface
plus the exported-receiver fix" (the pause itself in `HANDOVER.md`, and the account and upload-key
triggers above) reads "the exported-receiver fix" as the **complete** fix — receiver closed *and*
pairing built — so all of them remain unmet while pairing is unbuilt. This is the reading this
entry already implied; it is now settled rather than left to a reader. Reserved under rule 4 and
made by the developer, not inferred.

*Previously recorded here: "`RECEIVER_EXPORTED` means any installed app can start and stop the
scroll… Planned fix: channel off by default plus a per-install pairing token", with the note
that the exposure's "ceiling is denial-of-function rather than data access: a hostile app could
start and stop scrolling and nothing more, because `capabilities=32`, `eventTypes=0` and the
absent INTERNET permission apply to the whole process." That bound was correct and still holds
for the pairing gap.*

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

### The Play testing requirement is the instrument, not an obstacle

**The requirement applies** — 12 testers continuously opted in for the 14 days before applying
for production access. It is defined under *Play developer account* above and not restated
here; this section is what follows from it. It was previously written as a conditional, from a
point where nobody had checked whether it applied.

That cohort is **precisely the between-subjects, fresh-reader, longitudinal design** this
question needs — the one named in HANDOVER.md as the only design that can settle the
comparative questions.

**Assign conditions across participants rather than within.** One condition per tester, never
a sequence. That removes memory, habituation, sensitisation and order effects in a single
move — every one of which corrupted the sessions run here.

**A cohort can only be assigned once, and the requirement is a state rather than a total.**
Those two facts compose into a scheduling constraint rather than an observation: **the fatigue
study and the Play production-access gate should be the same cohort, run once, immediately
before applying.** Running them separately means recruiting twelve disabled testers twice, for
no gain — and the 14-day window has to be the last 14 days before the application either way.

Plan the conditions before the closed test opens, not after.

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
| Should fast-forward latch or skip? | Beta, with target users. An informant outside the target population cannot settle it — and the obvious design violates standing rule 1 |
| Should the lead-in stretch? **Measured, not decided.** 375ms gives 0.67x mean / 2.0x peak against 120ms's 2.53x / 6.3x, at the cost of re-grip dead time growing ~440ms → ~695ms. The naive reader noticed neither the overshoot nor the pause at 120ms, so there may be nothing to buy | The same reader on both settings, blind. Note the clamp is device-dependent — 375ms here, 300ms on the Moto — so one shipped constant cannot be optimal on both |
| ~~Should `LEAD_IN_MS` stretch at low speeds?~~ superseded by the row above | **Margin is now measured as a weak lever**: its whole range is 3.96x → 2.05x cruise, and at margin 0 the ramp is 100% of the effect with a 5.6x peak. `LEAD_IN_MS` is the only lever reaching the dominant term — ~3× unused headroom against the long-press deadline, cost is re-grip dead time growing ~450ms → ~680ms. Untested |
| Host apps with vertical-gesture semantics | Untested: drag-to-dismiss, short-video feeds, video players mapping vertical drags to brightness/volume, nested-scroll and collapsing toolbars |
| What to do about touchless app switches | Incoming call, alarm, full-screen intent, deep link — the only remaining case where the finger keeps dragging in a window the user did not choose |
| **Ratchet not built** — four asserts, each a one-line change to lose and expensive to re-establish | Build it in the shipping repo, reading from the built artifact. Spec in the TODO section above, including the fourth assert whose mechanism is still undecided |
| ~~Does the nav-bar accessibility button show a chooser when more than one accessibility service is registered?~~ **Premise corrected and question answered, 16 Aug 2026 — see *The v1 control set*.** The term was wrong: **assignment** governs, not registration. And the answer has two conditions — the invoking surface, and whether a selection has been persisted — so it cannot be stated as one fact. A chooser appeared with a *single* registered service, and did not appear with two registered and one assigned on a surface proven to raise one. **Still open:** the button remains unavailable for this app's purpose on other grounds — touch-only, absent under gesture nav, and it toggles the service rather than starting a scroll | Cross-device replication of the sufficiency arm on the SM-P200, which never had a second service enabled |
| Can the light sensor serve as a windowless hands-free control? | Prototype occlusion detection on both devices — both have one. Needs no permission and does not touch the capabilities bitmask, so it costs nothing under standing rule 4. **Hypothesis only**, and possibly fatal on ambient variation, dim-room mounting, passing shadows, or on-change latency |
| Does the spasm-tolerant hard stop's hold-cover gesture violate standing rule 1? | The control-path charter. Pre-existing, not caused by the sensor result: hold-cover is sustained input, which rule 1 forbids, and it is nowhere recorded as a deliberate exemption. **Open design question — not resolved here.** Note the asymmetry runs backwards for a stop: a false positive is cheap (the scroll stops, the user restarts) and a false negative is the dangerous one, because the person needing a hands-free stop is by definition the person who cannot reach the screen |
| Does Reading mode (`com.google.android.accessibility.reader`) actually read window content, as the differentiation argument assumes? | `aapt2 dump badging` or `aapt2 dump xmltree` against its APK, or `dumpsys accessibility` against it running. **Currently a self-description read from its Play listing on 13 Aug 2026** — weaker than either check under standing rule 4, because a listing notice says what a service *may* do rather than what its declared attributes *grant*. **Must be settled before the comparison appears in the store listing or anywhere public** |
| Liveness when the service is not running | Six ways to be "enabled but not working", none detectable by the app because it is not running to notice. Must be observable from outside the service |
