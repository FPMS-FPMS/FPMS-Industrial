# FPMS Ground Rovers

Two autonomous ground rovers do the physical work: navigate to a dry zone, avoid obstacles and
each other, apply targeted water, and document the heritage site with geotagged imagery.

One is a new build. The other is the robot that won Gold at Nationals, rebuilt.

---

## Two rovers, one reborn champion

| | Rover 1 — new build | Rover 2 — champion rebuild |
|---|---|---|
| Role | Patrols zones 1–2, water refill | Patrols zone 3 |
| Compute | Orange Pi 5B 16 GB (RK3588S, 6 TOPS NPU) | Identical stack |
| Motion | Yahboom V3.0 + STM32F103 | Identical |
| Vision | AI camera 30 fps (YOLO) | Identical |
| LiDAR | D500, 360° / 12 m | Identical |
| Thermal | Yes | Yes |
| Inertial | 9-axis IMU + wheel odometry | Identical |
| Origin | Built new in 2026 | Rebuilt from the Gold-at-Nationals robot |
| New core-part cost | — | **$0** — computer, motors, wheels, camera all carried over |

Rebuilding the champion instead of buying a second robot means a sponsor's dollar goes twice as
far, and it gives us a true field twin for testing the peer-swarm hand-off.

## The stack

- **OS / middleware** — Ubuntu · ROS 2 Humble
- **Navigation** — Nav2 · SLAM Toolbox · EKF sensor fusion (LiDAR + IMU + odometry)
- **Perception** — YOLO running on the RK3588 RKNN NPU (on-device, no laptop)
- **Suppression** — pump + nozzle, short targeted spray
- **Dashboard** — FastAPI · WebSocket live telemetry
- **Swarm** — the two rovers see each other over DDS as moving obstacles, so one covers a zone
  while the other refills — no gap in patrol

## The five decisions each rover makes on its own

1. **Closest first** — which zone to serve next
2. **Gap-finding** — how to steer around obstacles (LiDAR)
3. **Spray or not** — a YOLO confidence gate
4. **Severity rank** — how urgent a zone is, from sensor magnitude
5. **Return home** — when all zones are clear, go back and refill

## Sensor fusion — five modalities

Spatial awareness (LiDAR), AI vision (camera + YOLO), thermal (hotspot confirmation),
inertial (IMU + encoders), and environmental (the zone nodes' soil / leaf / air readings)
are fused so no single sensor has to be right on its own.

## How the rovers relate to the rest of the system

The rovers are the **decide-and-act** tier. Upstream, the [zone nodes](../system/ARCHITECTURE.md)
sense risk and alert them; downstream, they report to the [cloud](../system/INTELLIGENCE.md) for
mission logging and natural-language reporting. The full sequence is in the
[mission workflow](../system/MISSION.md).

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
