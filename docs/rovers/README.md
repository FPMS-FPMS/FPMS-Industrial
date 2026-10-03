# FPMS Ground Rovers

Two autonomous ground rovers do the physical work: navigate to a dry zone, avoid obstacles and
each other, apply targeted water, and document the heritage site with geotagged imagery.

One is a new build. The other is the robot that won 1st place at Nationals, rebuilt — it is our competition robot for the Final.

---

## Two rovers, one reborn champion

| | Rover 1 — new build | Rover 2 — champion rebuild (competition robot) |
|---|---|---|
| Role | Senses only today; joins the Final demo later | The SCORCH rover at the Final: plans, drives, sprays, refills |
| Compute | Orange Pi 5 Max 8 GB | Orange Pi 5B |
| Motion | Yahboom STM32 ROS board V3.0 | Yahboom STM32 board, behind our own ROS 2 bridge |
| Vision | Thermal Master P1 thermal camera | USB colour camera; YOLO checks and logs the target |
| LiDAR | LDROBOT LD19 | LDROBOT D500, 360°, 10 scans/s |
| Inertial | IMU (ICM-20948) + wheel odometry | IMU + wheel encoders |
| Water | — | ESP32-S3 water board: spray pump, refill pump, level probe, servo-lowered refill tube |
| Origin | Built new in 2026 | Rebuilt from the 1st-place-at-Nationals robot |
| New core-part cost | — | **$0** — computer, motors, wheels, camera all carried over |

Rebuilding the champion instead of buying a second robot means a sponsor's dollar goes twice as
far, and it gives us a true field twin for testing the peer-swarm hand-off.

## The stack

- **OS / middleware** — Ubuntu · ROS 2 Humble
- **Navigation** — our own planner (`fpms_rover2`): the shortest route with at most two turns and two straight moves; every turn measured by LiDAR scan matching (to 0.09°); position corrected against fixed landmarks
- **Perception** — YOLO on the rover; it checks and logs the target, and never decides when to spray
- **Suppression** — pump + nozzle, short targeted spray
- **Dashboard** — FastAPI · WebSocket live telemetry
- **Swarm (planned)** — the two rovers see each other over DDS as moving obstacles, so one covers a zone
  while the other refills — no gap in patrol

## The five decisions Rover 2 makes on its own

1. **Is it safe to start?** — stop signal clear, LiDAR working (median of 5 scans), IMU live and still, wheels reporting
2. **Which route?** — the shortest route that keeps the whole body at least 120 mm from the obstacle (never below 85 mm), including room to turn
3. **When is a turn finished?** — LiDAR scans before and after, to 0.09°; one strong turn that cannot overshoot
4. **When to spray and refill?** — only after a full stop (1 s, moved less than 3 mm), then a 2 s stream; a refill is judged by how fast the level rises
5. **Where am I?** — wheel odometry corrected against at least 3 agreeing landmarks, shown live on the dashboard map

## Sensors — each used only for what it is good at

LiDAR (obstacles, route, turns, position), camera + YOLO (checks and logs the target), wheel
encoders and IMU (distance and motion), and the water-level probe (spray and refill). On Rover 1, a
thermal camera.

## How the rovers relate to the rest of the system

The rovers are the **decide-and-act** tier. Upstream, the zone nodes
sense risk and alert them; downstream, they report to the cloud for
mission logging and natural-language reporting. The full sequence is in the
mission workflow.

---

## Documentation


- **[Navigation](NAVIGATION.md)** — how the rover plans a route, avoids an obstacle,
  and measures its own turns. Includes the findings that changed the design.
- **[Calibration](CALIBRATION.md)** — every measured constant, with the method used
  to obtain it and the earlier value it replaced.
- **[Dashboard](DASHBOARD.md)** — the on-vehicle operator interface, with live
  screenshots.

Source: [`software/rover2/fpms_phase6.py`](../../software/rover2/fpms_phase6.py)

## Rover 2 at a glance

| | |
|---|---|
| Compute | Single-board computer, Ubuntu 22.04, Python 3.10 |
| Motor controller | Yahboom STM32 over CH340 USB serial, Rosmaster protocol |
| Drive | Four-wheel differential, 230 mm wide, 170 mm track |
| LiDAR | LD D500, 360 one-degree bins at 10 Hz |
| Odometry | Wheel encoders at 6.00 counts/mm |
| Heading | **LiDAR scan-matching** — the IMU is logged but not trusted |
| Interface | Self-hosted dashboard on port 8085 |

## The mission it performs

Drive from a start box to a target zone in a 1000 × 1200 mm arena, avoiding an
obstacle placed between the two, hold position for two seconds, and return home.
Turns land within 1–3°. A full round trip takes 25–35 seconds.

## Three decisions that shaped this stack

**The IMU is not trusted.** The gyro on the motor board under-reads physical
rotation by 4–5×. Every turn is measured by matching the live LiDAR scan against a
snapshot taken before the turn — a measurement that shares no hardware with the
IMU. The IMU is still printed beside every turn so the discrepancy stays visible.

**Safety outranks shape.** The competition route should be a clean two-joint
diamond. The planner tries hard to produce one, but if no two-joint route clears
the obstacle by 196 mm it keeps the longer A* path instead. An ugly safe route beats
a pretty one that clips.

**Everything is measured, not assumed.** Turn rate against motor duty is 6.3×
non-linear; on-ground rotation is 16× slower than the same duty with the wheels
raised. Several confident hypotheses — a command watchdog, a weak motor, a missing
driver — were each disproved by measurement. [Calibration](CALIBRATION.md) records
the false ones alongside the true, because the false ones were plausible.
