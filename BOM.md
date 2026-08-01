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
