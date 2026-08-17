# Rover navigation — how Rover 2 gets there and back

Rover 2 runs a single self-contained Python program (`software/rover2/fpms_phase6.py`)
that owns the LiDAR, the motor board, the planner and the operator dashboard. One
process, one web server, no ROS dependency on the vehicle.

This document describes what the rover actually does, and — just as importantly —
the things we measured that turned out to be false.

---

## The mission

From a start box, drive to a target zone, hold position for two seconds, and return
home, avoiding an obstacle placed anywhere between the two. The arena is
**1000 × 1200 mm**. The rover is 230 mm wide.

| Point | Coordinate |
|---|---|
| HOME / start box | (842.5, 157.5) |
| Zone A (M1) | (90, 1110) |
| Zone B (M2) | (910, 1110) |
| Dock aim for M2 | (880, 1075) |

The dock aim is deliberately **not** the zone centre. A 230 mm-wide rover cannot
centre on (910, 1110): `910 + 115 = 1025`, which is outside a 1000 mm arena. The
zone is ~100 mm across, so (880, 1075) sits inside it and is a pose the chassis can
physically occupy. Aiming at a coordinate the robot cannot reach produces a rover
that drives into a wall trying.

---

## Sensing

| Sensor | Device | Used for |
|---|---|---|
| 360° LiDAR | LD D500, 10 Hz, 360 one-degree bins | Obstacle map, **turn measurement**, front safety stop |
| Wheel encoders | Yahboom STM32 (Rosmaster) | Distance travelled, heading trim |
| IMU (gyro) | On the motor board | Diagnostic witness only — see below |

### The IMU is not trusted

This is the single most important engineering finding in the rover stack.

Measured against LiDAR scan-matching across many turns, the board's gyro
**under-reads physical rotation by a factor of 4–5**:

| Commanded | LiDAR measured | IMU reported |
|---|---|---|
| −31° | −29° | −12° |
| +80° | +83° | +35° |
| −83° | −82° | −41° |
| +29° | +26° | +8° |

A control loop that closes a turn on the integrated gyro rate therefore keeps
turning long past the target — it waits for a number that arrives at a fifth of the
true rate. Worse, the pose estimator was integrating heading from the same signal,
and since position integrates *along* heading, the rover's belief about where it
stood drifted faster than its heading error alone would suggest.

**Heading now changes only when the LiDAR measures a turn.** The gyro is still read
and logged beside every turn, purely so the discrepancy stays visible.

---

## Turning: LiDAR scan-matching as ground truth

A turn is executed as *pulse → stop → settle → measure → repeat*, and the
measurement is a brute-force rotational alignment of the current 360-bin range
vector against a snapshot taken before the turn:

```
if the chassis rotated by +s, a feature previously at bin a is now at bin a-s,
so r1[a] ≈ r0[a+s]
```

The search is **windowed** to the band between zero and the commanded angle plus a
pad. Searching all 360° lets a distant wall in a near-rectangular room fit slightly
better than the truth and veto good data — an aliasing failure we hit and fixed.

Scan-matching shares no hardware with the IMU, which is the whole point: two
self-consistent but wrong sensor loops had already hidden problems from us once.

Turns land within **1–3° in 2–4 pulses**.

### Turn power is wildly non-linear

Measured on the ground, both directions averaged, on a fresh 12.3 V pack:

| Duty | Turn rate | Time for a 90° turn |
|---|---|---|
| 30 | 16.2 °/s | 5.5 s |
| 40 | 36.7 °/s | 2.5 s |
| **50** | **65.4 °/s** | **1.4 s** |
| 60 | 101.7 °/s | 0.9 s |

Doubling duty from 30 to 60 gives **6.3×** the rotation, because below roughly duty
40 nearly all torque is consumed by skid-steer scrub. With the wheels off the
ground, duty 30 spins the equivalent of 265 °/s — on the ground it manages 16. **Never
infer on-ground turn performance from a wheels-up test.**

Running at duty 30 meant a 90° turn needed 5.5 s of *continuous* drive, which the
pulse-and-measure controller never delivers in one go; it would exhaust its pass
budget 6–9° short every time. Duty 50 is not arbitrary — it is the smallest value
that makes the geometry work.

---

## Driving straight: encoder differential, boost-only

Straight legs hold heading on the **encoder differential**, not the gyro:

```
err   = |left counts travelled| − |right counts travelled|
boost = min(8, |err| × 0.05)          # duty
```

At 6.00 encoder counts per mm, one count is 0.167 mm — about 0.04° of heading across
the 230 mm track. There is no drift term and no deadband blind spot.

The correction **boosts the lagging side only and never subtracts.** The earlier
form applied `power − c` to one side and `power + c` to the other; at a cruise duty
of 18 a 6-duty correction dropped one side to 12, below the motor deadzone, so that
side stalled and a heading correction became a swerve.

---

## Obstacle avoidance

1. Every LiDAR return is transformed into the **arena frame** using the live pose and
   kept if it falls inside the arena.
2. Surviving points are inflated into an occupancy grid at 50 mm resolution.
3. A* plans a route from the rover to a goal 120 mm short of the dock aim.
4. The route is reduced to a **two-joint shape** — start → apex → goal.
5. Every waypoint is clamped at least 156 mm inside every wall.

### Inflation must exceed the circumscribed radius

The rover's half-width is 115 mm, but on a 45° diagonal leg it presents its
**corner**: `√(115² + 105²) = 156 mm`. Inflating on 115 mm under-protects a diagonal
leg by 41 mm — enough to clip an obstacle the planner believed it had cleared.
Inflation is `115 + 100 = 215 mm`.

### Two joints, but never at the cost of safety

The competition route should be a clean diamond: two joints out, two back. A* plus
Douglas–Peucker simplification produces a staircase, and at loose tolerances it
collapses a genuine detour back into a straight line.

So the route is explicitly reduced to `start → apex → goal`. The apex is chosen from
every interior A* point plus an 84-point synthetic grid, scored by the **worst
clearance either leg achieves** against any obstacle point. The reduction is accepted
only if that clearance exceeds 196 mm (156 mm of body plus 40 mm of margin);
otherwise the full A* path is kept.

The rover will drive an ugly six-waypoint path rather than a pretty two-joint one
that clips. Shape is never traded for safety.

### Two bugs worth recording

Both were caught by previewing plans in **arena coordinates** rather than
robot-relative ones, and neither is visible in robot-relative numbers:

- The synthetic apex grid was not bounds-checked, so it once proposed an apex
  785 mm to the right — arena x = 1627, in a 1000 mm arena. It scored 210 mm of
  "clearance" purely because no obstacle points exist outside the arena to be near.
- A*'s `nearest_free()` snaps a blocked goal cell toward open space, and pushed the
  goal to arena x = 942 — 57 mm from the wall, for a rover with a 115 mm half-width.

Every waypoint is now transformed to the arena frame and clamped as the final step
before the rover is told to drive anywhere.

---

## Safety

- **Front stop** — any LiDAR return closer than 120 mm halts the current leg.
- **Per-leg verification** — the corridor ahead is re-checked after each turn, before
  committing to the leg.
- **Replan on block** — a blocked leg re-plans from the rover's actual position, up to
  four times, rather than skipping ahead and executing the next turn from the wrong
  place.
- **Refuse rather than guess** — if no route clears, the rover stops and says so.
- **STOP** is a top-level dashboard control and is honoured mid-leg.

---

## Things we measured that turned out to be false

Recorded because each cost us real time:

- **"The motor board has a command watchdog truncating our pulses."** It does not. A
  single `set_motor` call latches for its full duration; refreshing it every 50 ms
  gains 1.05×.
- **"A wheel is weak — the right pair stalls."** No. During a spin, clockwise favours
  motors 1 and 4 and counter-clockwise favours 2 and 3, mirrored exactly. That is
  normal diagonal loading. Always test both directions before condemning a motor.
- **"A driver or calibration file went missing."** Nothing is missing. The motor
  library is bit-identical to the version installed before nationals, verified
  against the package manager's own checksum manifest.
