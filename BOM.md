# FPMS-AS1 "Manta" — Bill of Materials

All prices in **USD** unless marked CAD. Prices are indicative and were current at time of
specification; verify before ordering.

---

## Airframe

| Component | Qty | Price | Source |
|---|---|---|---|
| EPP foam hot-wire CNC cutting (from our DXF) | 1 | ~$50–150 CAD | [Malton Best, Mississauga](https://maltonbest.com/fabrication-services/cnc-foam-cutting/) |
| EPP foam sheet stock | — | ~$15–25 CAD | local supplier |
| Carbon rod spar + reinforcement tape | 1 set | ~$15 | local hobby supplier |

## Propulsion

| Component | Qty | Price | Notes |
|---|---|---|---|
| T-Motor F80 1900KV | 3 | ~$84 | 1900KV is the long-range/efficiency variant, correct for 4S |
| HQProp 7×4.5 propeller | 3 (+spares) | ~$8 | Ships with 5 mm bore; needs the included 4 mm reducer ring |
| JHEMCU EM-45A 4-in-1 ESC | 1 | ~$52 | 45 A/ch, DShot, 3 of 4 channels used |
| HEEWING T1 VTOL tilt servo | 2 | ~$36 | Metal gear; stock part on the reference aircraft |

## Flight control & navigation

| Component | Qty | Price | Notes |
|---|---|---|---|
| Pixhawk 6C | 1 | ~$166 | ArduPilot QuadPlane; JST-GH connector standard throughout |
| Holybro PM06 V2 power module | 1 | ~$20 | ⚠️ See compatibility note below |
| CubePilot Here3+ RTK GNSS | 1 | ~$180 | ⚠️ **Must connect to CAN1** — CAN2 unsupported in current firmware |
| SiK-class telemetry radio (pair) | 1 | ~$40 | Powered from TELEM1 — single connector for power and data |

## Companion computing & uplink

| Component | Qty | Price | Notes |
|---|---|---|---|
| Radxa Cubie A7S (4 GB) | 1 | ~$25 | Octa-core, 3 TOPS NPU, 51 × 51 mm |
| Radxa Camera 13M 214 | 1 | ~$20 | Official module for this board; MIPI-CSI 31-pin FPC |
| USB 5G module | 1 | ~$35 | Bus-powered via USB 3.0 — no separate power rail |

## Power

| Component | Qty | Price | Notes |
|---|---|---|---|
| 4S Li-ion pack, ~5000 mAh (21700 cells) | 1 | ~$60 | **Pre-built** — avoids spot welding |

## Payload

| Component | Qty | Price | Notes |
|---|---|---|---|
| 3D-printed 120 mL tank + tapered outlet | 1 | ~$5 | Printed in-house; no off-the-shelf tank exists at this size |
| 3–6 V micro submersible pump | 1 | ~$3 | 80–120 L/h; runs on the 5 V peripheral rail |

## Connectors & cabling

| Component | Qty | Price | Notes |
|---|---|---|---|
| XT60 connector pair | 1 | ~$1 | Battery → PM06 V2 |
| 3.5 mm bullet connector set | 3 pair | ~$5 | ESC → motors; rated 40 A vs our 30 A peak |
| JST-GH → USB-C adapter cable | 1 | ~$5 | PM06 V2 5 V output → Cubie A7S |
| JST-GH → Dupont cable | 1 | ~$3 | Pixhawk TELEM2 → Cubie A7S MAVLink |

---

## Total

| | |
|---|---|
| **Electronics and hardware** | **~$755 USD** |
| **Airframe (CAD)** | **~$80–190 CAD** |
| **Approximate total** | **~$1,150–1,250 CAD** |

---

## ⚠️ Compatibility notes — read before ordering

**Holybro PM03D is NOT compatible with Pixhawk 6C.** The PM03D uses a digital I²C power
protocol; the Pixhawk 6C requires an **analog** power module. Holybro's own compatibility
documentation lists Pixhawk 6C as unsupported for PM03D. We specified PM03D at one stage and
caught this during a wiring audit before ordering. **Use PM06 V2.**

**Here3+ must be wired to CAN1.** CAN2 is not supported in current Here3+ firmware. Wiring to
CAN2 produces a silent failure — the GPS simply never appears.

**The FMU PWM rail needs external 5 V.** The Pixhawk 6C does not power the servo rail itself.
The PM06 V2 5 V BEC output must be connected to the FMU PWM power pin, or the tilt servos will
not move. This is easy to miss during assembly.

A full 14-connection compatibility audit covering voltage, connector type, protocol and required
ArduPilot parameters is maintained alongside this repository.
