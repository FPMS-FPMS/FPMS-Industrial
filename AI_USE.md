# AI Use Disclosure

**Team FPMS · WRO 2026 Future Innovators**

WRO rules require teams to disclose their use of AI tools. This document is a complete and
honest account. We would rather over-disclose than have a judge discover something we left out.

---

## Tools used

| Tool | Used for |
|---|---|
| Anthropic Claude | Design research, engineering calculations, code review, documentation drafting, debugging |
| OpenAI ChatGPT | Diagram and infographic image generation from our written specifications |

## Where AI genuinely helped

**Component research.** Roughly 25 single-board computers and a dozen aircraft configurations
were surveyed and eliminated far faster than we could have managed manually. The elimination
*criteria* were ours; the search was accelerated.

**Catching our mistakes.** Three examples, all real:
- We specified a **Holybro PM03D** power module. An AI-assisted compatibility audit found that
  it uses a digital I²C protocol incompatible with the Pixhawk 6C's analog requirement. We
  changed to PM06 V2 **before ordering**.
- We proposed a **2 lift + 2 pusher** motor layout. It was pointed out that two lift points give
  roll authority but no pitch authority — geometrically impossible, regardless of motor power.
- Our 3D model had the **nacelle tilt inverted**. Aryan spotted the visual error himself; the
  root cause was traced with AI assistance.

**Documentation and code.** The flight simulator, this repository's documentation, and our
spreadsheets were drafted with AI assistance and then reviewed, corrected and verified by us.

## Where AI did *not* make the decisions

Every design decision was made by the team. Several times we rejected AI recommendations:

- AI recommended a **tailsitter** configuration on the strength of its endurance numbers. We
  rejected it — a tail-standing launch was too unstable for a student-operated first build.
- AI initially proposed several **original airframe designs**. We rejected all of them and
  required a configuration already proven in production aircraft.
- AI suggested **6S** for the battery. We pushed back, asked for the actual current draw, and
  4S proved lighter and longer-flying.

We also caught AI errors. On one occasion it referenced a prior drone build that never existed;
we corrected the record. On another, the parasitic drag figure used in the simulator was roughly
double what the airframe geometry justified, and was corrected after review.

## What is entirely our own work

- The problem, the mission, and the decision to protect Indigenous cultural heritage
- All physical assembly, wiring, soldering and mechanical construction
- All hardware testing, calibration and tuning
- The competition presentation and all answers to judges
- Every final engineering decision

## Our position

AI was a research assistant and a reviewer. It made us faster and it caught errors that would
have cost us money and flight hardware. It did not design the robot, and it cannot answer a
judge's question about why we chose a Y3 tiltrotor over a hexacopter.

We can.

— **Aryan Wadhawan** and **Alex Tang**
