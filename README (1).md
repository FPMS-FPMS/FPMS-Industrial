# FPMS — Fire Prevention & Management System

**Autonomous cultural-heritage protection through fire prevention.**

> *"Protecting the past with the power of the present."*
> **Think Globally · Act Locally · Save Culture**

[![WRO 2026](https://img.shields.io/badge/WRO%202026-Future%20Innovators-c8102e)](https://wro-association.org/)
[![Gold — Canada Nationals](https://img.shields.io/badge/Gold-WRO%20Canada%20Nationals%202026-d4af37)]()
[![World Final](https://img.shields.io/badge/World%20Final-San%20Juan%2C%20Puerto%20Rico-0057b7)]()

---

## The problem

Every mainstream wildfire tool — satellites, smoke cameras, 911 calls, air tankers —
watches for **smoke**. By the time smoke appears, the site is already burning.

For Indigenous cultural heritage, that gap is fatal. A petroglyph, a ceremonial site, or a
culturally modified tree lost to wildfire is not *damaged* — it is **erased**. It cannot be
rebuilt, reprinted, or restored.

FPMS operates in the empty window *before* ignition: it finds dangerously dry ground near
heritage sites and treats it before there is anything to report.

## What it does

FPMS is an autonomous multi-device fleet that senses pre-ignition fire risk, decides on its
own which zone to serve, navigates there, applies targeted water, documents the heritage site
it just protected, and reports — with no human in the loop.

```
SENSE  ──▶  DECIDE  ──▶  NAVIGATE  ──▶  SUPPRESS  ──▶  REPORT
 zone       closest-      LiDAR +        targeted      log, geotag,
 nodes      first,        Nav2, obstacle  water         alert, return
 (ESP-NOW   severity      avoidance                     & refill
  < 2 s)    rank
```

## System architecture

| Subsystem | Count | Role |
|---|---|---|
| Zone sensor nodes | 3 | Solar ESP32-C3; soil moisture, leaf wetness, thermal. ESP-NOW alert in under 2 s |
| Ground rovers | 2 | ROS 2 / Nav2 autonomous navigation, YOLO perception, water application, heritage documentation |
| Water stations | 2 | Autonomous refill, ArUco docking |
| Aerial scout — FPMS-AS1 "Manta" | 1 | VTOL tiltrotor; aerial survey, RTK heritage waypoint logging, precision water delivery |
| Ground HQ | 1 | ROS 2 bridge, live dashboard, SQLite mission log, TTS, alerting |

## Subsystem documentation

- **[FPMS-AS1 "Manta" — aerial platform](docs/drone/README.md)** — Y3 tiltrotor VTOL flying wing
  - [Bill of materials](docs/drone/BOM.md)
  - [Wiring and power architecture](docs/drone/WIRING.md)
  - [Flight simulator](docs/drone/SIMULATOR.md)

## Technology

**Edge (on-vehicle)** — ROS 2 Humble · Nav2 · SLAM Toolbox · YOLO on RKNN NPU ·
ArduPilot QuadPlane · EKF sensor fusion
**Cloud** — AWS IoT Core · multimodal reporting agent · SQLite mission log · FastAPI
**Comms** — ESP-NOW (zone alerts) · WiFi 6 (telemetry) · DDS peer swarm · 5G uplink (aerial)
**Sensing** — LiDAR · AI vision · thermal · inertial · environmental

### Edge vs cloud — a deliberate split

All **real-time perception and control runs locally** on the vehicles: navigation, obstacle
avoidance, object detection, sensor fusion, and the spray decision. The rovers and the aircraft
keep operating with zero connectivity.

**Non-time-critical reasoning runs in the cloud** — mission report generation and natural-language
summarisation. This is not a limitation we worked around; it is the correct architecture.
A 6 TOPS edge NPU cannot run a vision-language model at usable speed, and pretending otherwise
would mean a robot that stalls waiting for a sentence. It is the same pattern commercial fleet
robotics uses, and it degrades gracefully: if the link drops, the vehicle carries on and reports later.

## Repository layout

```
.
├── docs/
│   └── drone/            FPMS-AS1 "Manta" design documentation
├── simulator/            Physics-based flight simulator (open in any browser)
├── AI_USE.md             Disclosure of AI assistance, per WRO rules
└── LICENSE
```

## The team

**Aryan Wadhawan** and **Alex Tang** — David Leeder Middle School, Ontario, Canada.

Two students who designed, built, coded and present the entire system themselves.

- 🥇 **Gold** — WRO Canada National Final 2026, Montréal
- 🌎 **Qualified** — WRO International Final, San Juan, Puerto Rico, December 2026
- Category: Future Innovators — Junior · Season theme: *Robots Meet Culture*

## Supporting the project

FPMS is built and funded by two students and their families. Component support, technical
mentorship, and partnership enquiries are all welcome.

📧 **scorch.sentinel.fpms@gmail.com**

## Licence

FPMS mission code is released under the [MIT Licence](LICENSE).

Third-party dependencies retain their own licences. Where GPL-licensed components are used,
they are kept in separate modules with clearly documented boundaries, and are **not** linked
into MIT-licensed FPMS mission code.

## AI use disclosure

This project used AI assistance during design and development. See **[AI_USE.md](AI_USE.md)**
for a full and honest account, as required by WRO competition rules.
