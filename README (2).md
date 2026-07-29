# FPMS-AS1 "Manta" — Aerial Platform

**Compact Y3 tiltrotor VTOL delta flying wing.**
Vertical takeoff, winged cruise range, precision water delivery.

---

## Why a VTOL, and why this configuration

The aerial scout has to do two things that fight each other: **hover precisely** over a heritage
site to document and treat it, and **cover ground efficiently** between zones. A multirotor does
the first and burns its battery doing the second. A fixed-wing does the second and cannot do the
first at all.

A tiltrotor VTOL does both — but the configuration matters. We did not invent one. The **Y3
pattern** (two tilting front rotors, one fixed tail lift motor, tailless delta body) is a proven,
in-production layout used by aircraft including the **FIMI Manta** and the **Foxtech Saber**.
We scaled that established pattern down to our payload rather than designing something novel and
unproven for a first-ever aircraft build.

### Configuration

| Phase | T1 (front-left) | T2 (front-right) | M3 (tail) | Lift from |
|---|---|---|---|---|
| **Hover / VTOL** | vertical | vertical | vertical | all 3 rotors |
| **Transition** | tilting 0→88° | tilting 0→88° | fading out | rotors → wing |
| **Cruise** | forward thrust | forward thrust | **off** | wing |

Servo slew is limited to **45°/s**, so a full transition takes about two seconds.

## Specification

| | |
|---|---|
| Configuration | Y3 tiltrotor (2 × tilting front, 1 × fixed tail) |
| Wingspan | 800 mm |
| Length | 490 mm |
| Height | 95 mm |
| All-up weight | ~950 g |
| Front propellers | 8 in (Ø203 mm) |
| Tail lift propeller | 7 in (Ø178 mm) |
| Front motor spacing | 560 mm |
| Cruise speed | 15–22 m/s |
| Max speed | 25–30 m/s |
| Stall speed | ~8.8 m/s |
| Hover power | ~200 W |
| Cruise power | ~100 W |
| Endurance | ~21 min hover · ~43 min cruise |
| Power system | 4S Li-ion (21700), ~5000 mAh, 74 Wh |
| Frame | EPP foam + carbon spar reinforcement |
| Control surfaces | Dual elevons |
| Payload | 120 mL water, tapered-tube stream outlet |

Endurance and power figures are **not** vendor estimates — they are outputs of our own
physics simulation, derived from the real motor, propeller and battery specifications.
See [SIMULATOR.md](SIMULATOR.md).

## Avionics

| Function | Part |
|---|---|
| Flight controller | Pixhawk 6C — ArduPilot QuadPlane (tiltrotor) |
| Positioning | CubePilot Here3+ RTK GNSS — centimetre-level heritage waypoints |
| Power management | Holybro PM06 V2 (power module + ESC distribution) |
| Propulsion | 3 × T-Motor F80 1900KV |
| Speed control | JHEMCU EM-45A 4-in-1 ESC (3 of 4 channels used) |
| Tilt actuation | 2 × HEEWING T1 VTOL tilt servos |
| Companion computer | Radxa Cubie A7S — onboard vision, mission processing |
| Camera | Radxa Camera 13M 214 (13 MP) — heritage documentation |
| Uplink | WiFi 6 + USB 5G module |
| Telemetry | SiK-class radio (redundant MAVLink link) |
| Payload | 3D-printed 120 mL tank + 3–6 V micro submersible pump |

Full part numbers, prices and purchase links: **[BOM.md](BOM.md)**
Wiring, connectors and power rails: **[WIRING.md](WIRING.md)**

## Airframe manufacturing

The delta body is **hot-wire CNC cut from EPP foam** — the same construction method used by the
production aircraft this design is based on. EPP is light, crash-resistant, and repairable in the
field, which matters for a first-build aircraft that will be flown by students.

Cutting is done from our CAD/DXF by **Malton Best**, Mississauga, Ontario. A carbon rod spar is
bonded into a routed channel after cutting to provide bending stiffness.

## Water payload

A 120 mL 3D-printed tank feeds a 3–6 V micro submersible pump through a **tapered tube outlet**
producing a focused stream — deliberately **not** a mist.

Two reasons. First, WRO rules require a clear water stream rather than a mist. Second, our pump
develops only 40–110 mm of lift; an atomising nozzle at that pressure would dribble or clog
rather than atomise. The simple tapered tip is both the compliant choice and the one that
actually works with this pump.

> **Known constraint, surfaced by simulation:** at the pump's rated ~25 mL/s, a full 120 mL tank
> supports only about **4.8 seconds of total spray** — roughly one full treatment. This is a real
> design limit we found before building, not after. Mitigations under consideration: a larger tank,
> a flow-restricted outlet, or accepting one treatment per sortie.

## Regulatory note

Under **WRO Rule 5.9**, drones may not be flown at the competition venue. The Manta appears in
pre-recorded media only. All aerial development, testing and flight operations take place outside
the venue in compliance with Transport Canada RPAS regulations.

## Design history

This aircraft went through a genuine elimination process rather than arriving fully formed.
Configurations evaluated and rejected, with reasons:

| Configuration | Why rejected |
|---|---|
| Quad tiltrotor (4 tilt servos) | 4 servos = 4× the mass and failure points; *lowest* endurance of all candidates despite the best wing |
| Hexacopter lift+cruise | Real motor-out redundancy, but 7 motors and 7 ESCs exceeded budget |
| Tri-lift + pusher (4 fixed motors) | Excellent endurance and zero transition risk, but 4 motors over budget |
| Quad / tri tailsitter | Best raw endurance and zero servos, but tail-standing launch was judged too unstable for a student-operated first build |
| 2 fixed lift + 2 pusher | **Geometrically impossible** — two lift points give roll authority but no pitch authority. No amount of thrust fixes this |
| Pure fixed wing | Cannot hover; cannot treat a site |

The Y3 survived because it is the only configuration that gives full attitude authority in hover
(three non-collinear lift points), efficient winged cruise, belly-down landing stability, and a
motor count within budget — **and** because it is already flying commercially, so we are not the
first to prove it works.
