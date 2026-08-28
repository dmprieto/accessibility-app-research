# Parked

Things deliberately not being worked on, kept because losing them would cost more than
carrying them.

**What un-parks an item:** a charter's question makes it relevant, or a decision comes to
depend on it. **Not a review schedule, and not elapsed time.** A parking lot with a review
cadence is a backlog, and a backlog is the thing this file exists to avoid — items here are
not waiting for attention, they are waiting for a reason.

Each entry says what it is, why it is parked, and what would un-park it. An entry nobody can
name an un-parking condition for does not belong here; it belongs deleted.

---

## Proximity: binary vs ranged on the Moto G54

**What.** Whether the Moto's proximity sensor reports a true distance or only near/far.

**Why parked.** Android 15's `dumpsys sensorservice` no longer prints `maxRange`, so answering
it needs `getMaximumRange()` from app code rather than a read-only dump — and it now governs
only a secondary control, since proximity cannot be the primary hands-free path on hardware
where it is absent entirely. The cost went up and the stakes went down in the same pass.

**What un-parks it.** A control-path charter choosing a proximity design whose behaviour
differs between binary and ranged — a graduated gesture, for instance, rather than
cover/uncover. Binary is sufficient for cover/uncover, so a charter that only needs that
leaves this parked permanently.

**UN-PARKED 2026-08-28 (design discussion; developer-relayed).** The proximity control-path charter
— reprioritised to run next after the Option D reachability finding — includes **wave recognition**,
not only cover/uncover, so it needs a design whose behaviour differs between binary and ranged: the
un-parking condition above. Answer it **first**, the way the 2026-08-13 sensor-availability check
preceded everything — via `getMaximumRange()` from app code (dumpsys no longer prints `maxRange`).
If the charter's design settles to cover/uncover only, this re-parks (binary suffices).

## Carry the sensor inventory in the `DEVICE` log line

**What.** Have the service log its sensor inventory at connect, alongside manufacturer, model,
API and build, so sensor availability arrives through logcat like every other field in
DEVICES.md.

**Why parked.** It is the right long-term answer — free on every future device, and it keeps
DEVICES.md's single-provenance discipline intact instead of carrying a `dumpsys`-sourced
exception. But it is app work that does not exist, and adopting it now would leave the cells
blank until it is built. The `dumpsys sensorservice` route fills them today.

**What un-parks it.** Enough devices measured that reading `dumpsys` per device becomes the
bottleneck, or any app-side sensor work that makes logging the inventory nearly free.
