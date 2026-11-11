# RET — Timing Evidence Report

- **Team:** Kevin Chavarro (`kchavarro-C`) <!-- Añadir nombre de tu compañero/a si aplica -->
- **Boards:** ESP32 DevKitC v1.1 (CH340C, 4 MB), STM32 NUCLEO-L476RG
- **AI use:**
  - **Week 1:** Asistencia con Antigravity para configuración del entorno Zephyr, instalación de toolchains (ARM y Xtensa), medición de ciclo de iteración y estructura inicial del RET.

**Living document** — updated every week; handed in at the workshop (week 7) and
at the close (week 14).

**House rule:** *"show me the trace"* — every timing claim cites a measurement.

## 1. The system and its task set

Requirements first — one sentence each, EARS style (*when/while `<condition>`, the
system shall `<response>` within `<deadline>`*) — then the task that implements them:

| ID | Requirement |
|---|---|
| REQ-CTRL-01 | While the system is irrigating, the flow/pressure control loop shall run every 10 ms (deadline = period). |
| REQ-ESTOP-01 | When an overpressure condition is detected, the emergency stop shall shut off the pump/valve within 5 ms. |
| REQ-SAMP-01 | While the system is active, the sensor sampling task shall acquire pressure ADC readings at 1 kHz with bounded jitter. |
| REQ-TELEM-01 | While connected to the network, the telemetry task shall transmit sensor and status telemetry frames every 1 s. |
| REQ-CONS-01 | When a user enters a UART console command, the command parser shall respond within 100 ms; late responses are worthless. |

| Task | Req. | Type (H/F/S) | Period ($T_i$) | Deadline ($D_i$) | Measured $C_i$ | How it was measured |
|---|---|---|---|---|---|---|
| Sensor sampling | REQ-SAMP-01 | Hard | 1 ms | 1 ms | `____` | GPIO + logic analyzer |
| Emergency stop (e-stop) | REQ-ESTOP-01 | Hard | Sporadic (1 ms check) | 5 ms | `____` | GPIO + logic analyzer |
| Control loop | REQ-CTRL-01 | Hard | 10 ms | 10 ms | `____` | GPIO + logic analyzer |
| Command console | REQ-CONS-01 | Firm | 5 ms poll / sporadic | 100 ms | `____` | UART trace / timer |
| Telemetry to Hub | REQ-TELEM-01 | Soft | 1 s | 1 s | `____` | Scheduler trace / GPIO |

### Task Classification Rationale (Taxonomy)
- **Hard Real-Time (Sampling, Control Loop, Emergency Stop)**: A missed deadline causes physical failure or safety hazards (burst pipes due to late e-stop, poor irrigation control, invalid discrete control assumptions).
- **Firm Real-Time (Command Console)**: A late response to a user console command has zero utility (the user pressed Enter and waited, or old commands become irrelevant), but it does not cause catastrophic physical damage.
- **Soft Real-Time (Telemetry)**: Minor delays in transmitting remote status to the Hub merely degrade data freshness; the system tolerates delayed packets as long as the average throughput is maintained.

## 2. ADRs

### ADR-001 — `<title>`
**Context:** … · **Decision:** … · **Justification (with numbers):** … · **Status:** Proposed

## 3. Evidence by week

Each entry cites the `REQ`(s) it verifies.

### Week 1 — Environment and iteration loop baseline
- **Board:** ESP32 DevKitC (CH340C, 4 MB) & NUCLEO-L476RG (build verified)
- **Target:** Full iteration cycle < 60 s.

| Quantity | Prediction | Measured value | Target | Status |
|---|---|---|---|---|
| Edit → pristine build (`-p`) → flash → console | 30 s | 22.38 s | < 60 s | PASS |
| Same, incremental build (one line changed, no `-p`) | 15 s | 26.84 s | < 60 s | PASS |
| Members with blinky / hello_world running from own laptop | 100% | 1 / 1 | all | PASS |

Console evidence verified:
```text
*** Booting Zephyr OS build 2f0bc11264a8 ***
Hello World from GreenField Control Team! [v2] esp32_devkitc/esp32/procpu
```

### Week 2 — superloop baseline (state the board)
<jitter/latency table + a one-sentence reading>

### Week 3 — S3 baseline and silicon comparison
…

## 4. Schedulability analysis

U = ΣC_i/T_i with **measured** C_i; test used (RM / hyperbolic / EDF); RTA as a
script with its output; blocking B_i if there are mutexes. (Formulas: READINGS.md,
"The math that does get used".)

From week 12: the CAN frame plan (IDs, periods, sizes), bus utilization and each
frame's response time; then the end-to-end budget of each requirement that crosses
the bus, hop by hop:

| Requirement | Node `R` (APPCPU) | IPM hop | CAN `R` | Hub wake-up (max) | Actuation | Sum | Deadline | Measured end to end |
|---|---|---|---|---|---|---|---|---|
| e-stop → pump stop | ____ | ____ | ____ | ____ | ____ | ____ | 20 ms | ____ |

## 5. Functional safety (final project)

Declared safe state (failure ⇒ valve closed), watchdog, and the evidence of the
fail-safe firing.

## 6. Agent-built components (final project)

The Hub's GUI and routing, built by an agent from `spec/hub-soft.md`. Tool, model
and where the session log lives: `<…>`.

| REQ | What the spec asks | How it was measured | Measured | Met? |
|---|---|---|---|---|
| REQ-HUB-xx | … | … | ____ | ____ |

Deviations: where the agent's code departed from the spec, and which measurement
caught it.
