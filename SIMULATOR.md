# FPMS-AS1 "Manta" — Flight Simulator

A physics-based 3D flight simulator for the Manta, written to validate the design
**before committing to a build**.

**[▶ Open the simulator](../../simulator/FPMS-AS1-Manta-Simulator.html)** — single HTML file,
no install, runs in any browser.

---

## Why we built it

This is the team's first aircraft. Rather than buy ~$1,200 of components and discover the power
budget was wrong on the first flight, we built a simulator driven by the *actual* specifications
of the *actual* parts on the bill of materials, and checked the numbers first.

It is not a game. Every constant traces to a real datasheet. Change a component and the whole
aircraft behaves differently.

## The physics

| Quantity | Model |
|---|---|
| Thrust | `T = T_max × throttle²` per motor (thrust ∝ RPM²) |
| Motor power | Momentum theory — induced velocity solved iteratively, efficiency ramped from static to forward flight |
| Lift | `L = ½ρV²S · C_L`, with `C_L = C_L0 + C_Lα·α` |
| Drag | `C_D = C_D0 + C_L²/(π·AR·e)` plus parasitic `C_D·A`, blending to flat-plate at high angle of attack |
| Battery | Li-ion discharge curve + 60 mΩ internal resistance, solving `R·I² − V_oc·I + P = 0` |
| Attitude | ArduPilot-style first-order tracking to commanded pitch/roll |
| Tilt | Servo slew rate limited to 45°/s |

Vehicle constants: 0.95 kg AUW · 12.75 N per motor · 0.18 m² wing · AR 3.56 · 74 Wh pack.

## Validation

The flight model was tested headlessly against the design targets. Nine test groups, all passing:

| Check | Simulated | Design target |
|---|---|---|
| Hover throttle | 49.4% | 41–49% |
| Hover power | 206 W | 160–220 W |
| Hover endurance | 21.6 min | ~20 min |
| Cruise power | 103 W | 70–95 W |
| Cruise endurance | 43.2 min | — |
| Stall speed | 8.8 m/s | below 15 m/s cruise |

Also verified: hover-to-cruise transition without altitude loss, tail motor shutdown in cruise,
position hold against 10 m/s wind, full autonomous mission to completion, crash detection on hard
impact, genuine battery depletion, and numerical stability from 8 ms to 50 ms timesteps.

## Features

**Flight modes** — HOVER (manual), POSITION HOLD (altitude + position), CRUISE (winged forward
flight), MISSION (fully autonomous)

**Autonomous mission** — click the map to drop waypoints; the aircraft takes off, cruises out,
transitions to hover, descends, sprays, climbs, moves to the next waypoint, then returns and lands:

```
TAKEOFF → TRANSIT → ARRIVE → DESCEND → SPRAY → CLIMB → RTH → APPROACH → LAND
```

**Environment** — 2.6 km forest, four lakes, terrain elevation, adjustable wind speed, direction
and gust intensity

**Live telemetry** — altitude, airspeed, vertical speed, attitude, nacelle tilt, per-motor thrust,
power draw, pack voltage, per-cell voltage, current, mAh and Wh consumed, endurance remaining,
water tank level

**Failure modes** — the aircraft can genuinely crash. Hard impact, battery depletion, stall
warning, throttle saturation

**Views** — follow, chase, orbit, nose camera, and an INSPECT mode that lets you zoom to 0.55 m
and scrub the nacelle tilt through its real 45°/s servo sweep

## What it caught

Two real findings that changed the design:

1. **Tank capacity.** At the pump's rated 25 mL/s, 120 mL gives only ~4.8 seconds of total spray —
   about one treatment. Found in simulation, not on the flying field.
2. **Nacelle orientation.** An inverted rotation in the 3D model showed the rotors pointing forward
   in hover and down in cruise. The *physics* was correct, but the bug would have propagated
   straight into presentation renders and the technical report.

## Controls

| Input | Action |
|---|---|
| `A` / `D` | Yaw |
| `W` / `S` | Pitch |
| `Q` / `E` | Throttle trim |
| Map click | Add waypoint |
| Drag / scroll | Orbit and zoom (ORBIT and INSPECT views) |
