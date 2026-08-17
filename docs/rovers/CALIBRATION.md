# Rover 2 — measured constants

Every number here was measured on the vehicle, not taken from a datasheet or
inferred from a specification. Where a figure replaced an earlier assumption, the
old value and the reason are given, because the wrong values were plausible.

Conditions unless stated: fresh pack at 12.2–12.4 V, rover on the competition
floor, LiDAR scan-matching as rotation ground truth.

---

## Drivetrain

| Quantity | Value | How it was measured |
|---|---|---|
| Encoder scale | **6.00 counts/mm** (0.1666 mm/count) | `π · 70 mm / 1320` wheel geometry, cross-checked against odometry to 2.5% |
| Track (wheel centres) | 170 mm | Chassis measurement |
| Body width | 230 mm | Chassis measurement |
| Half-width (inscribed radius) | 115 mm | — |
| **Circumscribed radius** | **156 mm** | `√(115² + 105²)` — what the chassis presents on a diagonal |
| Kinetic deadzone | ~duty 22 | Duty sweep; below this the wheels do not turn reliably |

## Straight-line speed

| Duty | Speed |
|---|---|
| 18 | ~137 mm/s |
| 24 | ~219 mm/s |
| 26 | ~237 mm/s |

Cruise duty is **18**. It was 26 in the build that won nationals; 26 crosses a
1 m arena in about four seconds, which is too fast to correct.

Free-running (wheels off the ground) the motors deliver ~394 mm/s of wheel travel
at duty 30, dead linear for the full 2.5 s of a test pulse. There is no power
shortage in this drivetrain.

## Turn rate versus duty

Measured on the ground, both directions averaged:

| Duty | Turn rate | 90° turn takes |
|---|---|---|
| 30 | 16.2 °/s | 5.5 s |
| 40 | 36.7 °/s | 2.5 s |
| **50** ← in use | **65.4 °/s** | **1.4 s** |
| 60 | 101.7 °/s | 0.9 s |

**Strongly non-linear: 6.3× more rotation for 2× the duty.** Skid-steer scrub
consumes nearly all torque below about duty 40. The same duty 30 that gives 16 °/s
on the floor spins the equivalent of **265 °/s** with the wheels raised — a factor of
16. On-ground turn performance cannot be inferred from a bench test.

## IMU error

| Commanded | LiDAR measured | IMU reported | Ratio |
|---|---|---|---|
| −31° | −29° | −12° | 2.4× |
| +80° | +83° | +35° | 2.4× |
| −83° | −82° | −41° | 2.0× |
| +29° | +26° | +8° | 3.3× |

The gyro under-reads by roughly **4–5×** in mission conditions and is excluded from
both turn termination and pose estimation. Parked, it correctly reads ~0 within a
±0.01 rad/s deadband, so it is not faulty in an obvious way — which is exactly what
made this expensive to find.

The motor board reports firmware 3.6, car type 1, motion PID [0.8, 0.06, 0.5], and
emits ICM-format IMU frames (function ID `0x0E`).

## Motor board behaviour

| Question | Answer | Evidence |
|---|---|---|
| Does `set_motor` need refreshing? | **No** | Single call vs refresh every 50 ms: 5906 vs 6204 counts over 2.5 s — a 1.05× difference |
| Is one side weaker? | **No** | CW favours M1+M4, CCW favours M2+M3, mirrored — normal diagonal spin loading |
| Motor mapping | M1, M2 = LEFT · M3, M4 = RIGHT | Verified by per-wheel encoder deltas |

## LiDAR

| Quantity | Value |
|---|---|
| Device | LD D500, 230400 baud |
| Rate | ~10 Hz |
| Coverage | **358 of 360** one-degree bins filled indoors |
| Max usable range | 2600 mm |

Bin coverage matters: the scan-matcher needs at least 40 overlapping pairs between
two scans, and 358 filled bins provides 179 samples at a stride of two. An earlier
figure of "22 points" came from a display filter, not the data the matcher reads.

## Planner

| Constant | Value | Why |
|---|---|---|
| Grid resolution | 50 mm | At 100 mm, a 215 mm inflation rounds to 300 mm and closes the only corridor in a 1 m arena |
| Inflation radius | 215 mm | 115 mm robot + 100 mm clearance; must exceed the 156 mm circumscribed radius |
| Two-joint acceptance | 196 mm | 156 mm body + 40 mm margin |
| Goal standoff | 120 mm | Was 300 mm, which parked the rover 300 mm short of the zone by design |
| Zone hold | 2.0 s | Mission requirement |

## Power

The pack is a genuine variable, not a footnote.

- Turn accuracy degrades measurably as voltage sags.
- The single-board computer browns out and reboots near **11.0 V**.
- Over one session the pack fell 11.7 → 11.1 V and the rover rebooted mid-test.

**Do not tune against a discharging pack.** Numbers taken below ~11.5 V were not
reproducible and cost us a session's work before we recognised the pattern.
