# FPMS-AS1 "Manta" — Flight Simulator

Before committing to the aircraft build, the design was validated against a
physics simulator driven by the real motor, propeller and battery specifications.

The simulator is a self-contained web application — open
[`simulator.html`](simulator.html) in any browser, no install required.

## What it validates

| Metric | Simulated | Design target | Method |
|---|---|---|---|
| Hover power | 206 W | 160-220 W | Momentum theory |
| Hover throttle | 49% | 41-49% | `T = T_max · throttle²` |
| Hover endurance | 21.6 min | ~20 min | 74 Wh / blended power |
| Cruise endurance | 43.2 min | — | L/D at 18 m/s |
| Stall speed | 8.8 m/s | below 15 m/s cruise | `sqrt(2W / rho·S·C_Lmax)` |

It also verifies the hover-to-cruise transition without altitude loss, tail-motor
shutdown in cruise, position hold against 10 m/s wind, a full autonomous mission
to completion, and numerical stability across frame rates.

See also: [aircraft overview](README.md) · [bill of materials](BOM.md) ·
[wiring & power](WIRING.md)
