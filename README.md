# HVAC Digital Twin System — Business Overview & User Guide

> **Audience:** Facility Managers, Operations Teams, HVAC Engineers, Business Stakeholders
> **Purpose:** Conceptual overview of the HVAC Digital Twin platform — what it is, how it works, and how to use it

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [What Is a Digital Twin?](#2-what-is-a-digital-twin)
3. [System Components](#3-system-components)
4. [Architecture](#4-system-architecture)
5. [Monitored Equipment & Control Parameters](#5-monitored-equipment--control-parameters)
6. [Key Performance Indicators](#6-key-performance-indicators-kpis)
7. [Anomaly Detection](#7-anomaly-detection)
8. [Predictive Analytics](#8-predictive-analytics)
9. [Actionable Insights](#9-actionable-insights)
10. [Example Use Cases](#10-example-use-cases)
11. [How to Use the Application](#11-how-to-use-the-application)

---

## 1. Executive Summary

The **HVAC Digital Twin** is a software system that creates a live, intelligent virtual replica of a building's HVAC infrastructure — specifically an **Air Handling Unit (AHU)** and a **Chiller**. It continuously monitors sensor data, measures efficiency, detects faults early, predicts future performance, and recommends corrective actions — all through a simple API that can feed any dashboard or building management interface.

**Key business benefits at a glance:**

| Benefit | How the Twin Delivers It |
|---|---|
| Lower energy costs | Continuous COP and power monitoring surfaces inefficiency before bills spike |
| Reduced downtime | Early anomaly warnings enable planned repairs, not emergency call-outs |
| Smarter maintenance | Data-driven insights replace fixed-interval schedules |
| Informed planning | Physics and ML-based predictions answer "what-if" questions |
| Audit-ready evidence | Timestamped telemetry history supports compliance and reporting |

---

## 2. What Is a Digital Twin?

A **digital twin** is a virtual model of a physical system that runs continuously alongside the real equipment. It ingests the same data the real system produces, processes it, and lets you observe, analyse, and predict behaviour — without touching the physical machinery.

Think of it as a **flight simulator for your HVAC plant**: engineers can test scenarios, validate decisions, and understand failure modes in a safe virtual environment before making changes to the real system.

### How this twin works — step by step

```
Real / Simulated Sensors
        │
        ▼
  Telemetry Snapshot  ──►  State Store (JSON)
        │
        ▼
 ┌──────────────┐   ┌─────────────────┐   ┌──────────────────┐
 │ KPI          │   │ Anomaly         │   │ Prediction       │
 │ Calculator   │   │ Detector        │   │ Engine           │
 └──────┬───────┘   └────────┬────────┘   └────────┬─────────┘
        │                    │                      │
        └──────────┬─────────┘                      │
                   ▼                                │
           Insights Engine                          │
                   │                                │
                   └──────────┬─────────────────────┘
                              ▼
                         REST API
                              │
                    Dashboards / BMS / Apps
```

---

## 3. System Components

The platform is made up of six specialised modules. Each has a single, well-defined responsibility.

| Component | Responsibility |
|---|---|
| **Mock Data Generator** | Produces realistic AHU and Chiller sensor readings with daily sinusoidal patterns and periodic fault injection (spikes, drifts, stuck sensors) |
| **KPI Calculator** | Derives Chiller COP, AHU Temp Delta, Total Power, and Energy Efficiency from live telemetry; assigns Good / Warning / Critical status to each |
| **Anomaly Detector** | Compares every reading against defined safety thresholds; raises Warning or Critical anomalies with component, metric, value, and timestamp |
| **Insights Engine** | Translates KPIs and anomalies into prioritised, plain-language recommendations for operators |
| **Prediction Engine** | Forecasts future KPI values using either a thermodynamics-based physics model or a pre-trained machine-learning regression model |
| **State Manager** | Persists the complete twin state to a JSON file; restores automatically on restart; no external database required |

---

## 4. System Architecture

The twin is organised into four logical layers:

### Layer 1 — Data Layer
Raw telemetry snapshots (sensor readings) are generated every simulation step. Up to **500 snapshots** are kept in a rolling history. The entire state — including AHU readings, Chiller readings, telemetry history, and step counter — is saved to a single JSON file after every step.

### Layer 2 — Analytics Layer
Three engines process the live state simultaneously:
- The **KPI Calculator** derives efficiency metrics
- The **Anomaly Detector** checks thresholds
- The **Insights Engine** synthesises both into recommendations

### Layer 3 — Prediction Layer
Two prediction approaches are available on demand:
- **Physics Predictor** — uses thermodynamics equations (Carnot COP, heat transfer, barometric formula for altitude) to forecast performance from first principles
- **ML Predictor** — uses hardcoded pre-trained linear regression coefficients to produce data-driven forecasts; useful for comparing against the physics model

### Layer 4 — API Layer
A RESTful HTTP API exposes all capabilities. Any dashboard, building management system, or analytics tool can consume the endpoints using standard HTTP calls.

---

## 5. Monitored Equipment & Control Parameters

### 5.1 Air Handling Unit (AHU)

The AHU conditions and circulates air throughout occupied spaces.

| Parameter | Description | Normal Range |
|---|---|---|
| Supply Air Temperature | Temperature of air delivered to zones | 5 – 22 °C |
| Return Air Temperature | Air temperature returning from occupied areas | 18 – 35 °C |
| Mixed Air Temperature | Blend of fresh and return air before conditioning | Derived |
| Supply Fan Speed | Fan operating level as a percentage of maximum | 0 – 100 % |
| Supply Air Pressure | Static pressure in the supply ductwork | 100 – 500 Pa |
| Flow Rate | Volume of conditioned air being circulated | Non-negative |
| Power Consumption | Electrical energy consumed by the AHU fan | Non-negative kW |

### 5.2 Chiller

The Chiller produces chilled water used to remove heat from the building.

| Parameter | Description | Normal Range |
|---|---|---|
| Chilled Water Supply Temp | Temperature of water sent to cooling coils | 3 – 15 °C |
| Chilled Water Return Temp | Water temperature returning from the building | > Supply temp |
| Condenser Water Temperature | Governs heat rejection efficiency; tied to wet-bulb temp | Derived |
| Compressor Power | Electrical energy consumed by the compressor | > 0 kW |
| Cooling Capacity | Total heat removed from the building | > 0 kW |
| COP | Coefficient of Performance (efficiency ratio) | Target > 3.0 |
| Refrigerant Pressure | Pressure in the refrigeration circuit | 200 – 800 kPa |

### 5.3 Ambient Conditions (for Predictions)

The prediction engine also accepts environmental inputs that affect HVAC performance:

| Parameter | Description | Default |
|---|---|---|
| Outdoor Temperature | Ambient dry-bulb temperature (°C) | Estimated from time of day |
| Wet-Bulb Temperature | Reflects humidity; drives condenser performance | Estimated at ~80% RH |
| Altitude | Elevation above sea level (metres); affects air density and fan power | 0 m (sea level) |
| Hour of Day | 0–23; used for sinusoidal load estimation | Current UTC hour |
| Occupancy Load Factor | 0.0 (empty) to 1.0 (full occupancy) | Required input |

---

## 6. Key Performance Indicators (KPIs)

The system calculates four KPIs continuously. Each receives a traffic-light status based on defined thresholds.

| KPI | Formula | Unit | Good | Warning | Critical |
|---|---|---|---|---|---|
| **Chiller COP** | Cooling Capacity ÷ Compressor Power | kW/kW | ≥ 3.0 | 2.0 – 3.0 | < 2.0 |
| **AHU Temp Delta** | Return Air Temp − Supply Air Temp | °C | 8 – 12 | Outside range | — |
| **Total Power** | AHU Power + Compressor Power | kW | < 150 | 150 – 200 | > 200 |
| **Energy Efficiency** | Cooling Capacity ÷ Total Power | kW/kW | ≥ 2.0 | 1.0 – 2.0 | < 1.0 |

> **What COP means in practice:** A COP of 3.0 means the chiller delivers 3 kW of cooling for every 1 kW of electricity consumed. A COP below 2.0 signals a critical fault — the system is consuming far more energy than it should, directly driving up utility costs.

---

## 7. Anomaly Detection

The Anomaly Detector compares every telemetry reading against fixed thresholds and raises anomalies in real time.

### Threshold Reference

| Component | Metric | Warning Threshold | Critical Threshold |
|---|---|---|---|
| AHU | Supply Air Temperature | > 18 °C | > 22 °C |
| AHU | Supply Air Pressure | < 100 Pa or > 500 Pa | — |
| Chiller | COP | < 3.0 | < 2.0 |
| Chiller | Refrigerant Pressure | — | < 200 kPa or > 800 kPa |
| System | Total Power | > 150 kW | > 200 kW |

Every anomaly record includes:
- **Component** — AHU, Chiller, or System
- **Metric** — the specific sensor or KPI affected
- **Severity** — Warning or Critical
- **Current Value** — the actual reading that triggered the alert
- **Threshold** — the limit that was crossed
- **Detected At** — UTC timestamp

---

## 8. Predictive Analytics

The Prediction Engine answers the question *"How will my HVAC system perform under future conditions?"* Two approaches are available, selectable per request.

### Physics-Based Prediction (`method=physics`)

Uses thermodynamics equations to derive predictions from first principles:

- **Carnot COP** is calculated from evaporator and condenser temperatures in Kelvin, then adjusted by a real-world efficiency factor
- **Condenser temperature** is derived from wet-bulb temperature plus a fixed approach offset (reflecting how cooling towers reject heat to wet-bulb conditions)
- **Cooling load** is estimated from outdoor temperature, occupancy factor, and a base internal heat gain constant
- **Fan power** is adjusted using the barometric altitude correction factor, as thinner air at altitude requires more fan energy to maintain the same mass flow rate
- Intermediate parameters (Carnot COP bound, efficiency factor, air density correction, derived temperatures) are returned alongside predictions for full transparency

### ML-Based Prediction (`method=ml`)

Uses pre-trained linear regression coefficients to produce data-driven forecasts:

- Feature vector includes: outdoor temp, hour of day, occupancy load, wet-bulb temp, altitude, and sine/cosine of the hour (to capture daily periodicity)
- Each KPI is predicted as a weighted dot product of the feature vector plus a bias term
- Outputs are clamped to physically plausible ranges (e.g. COP 0.5–8.0, Temp Delta 2–20 °C)

Both methods return the same response structure, making it straightforward to display and compare them side by side.

---

## 9. Actionable Insights

The Insights Engine translates KPIs and anomalies into plain-language recommendations, automatically prioritised by severity.

### Priority Levels

| Priority | Triggered By | Example Insight |
|---|---|---|
| **High** | Critical anomalies | "Chiller COP is critically low — schedule compressor inspection immediately" |
| **Medium** | Warning conditions or KPI deviation | "AHU temperature delta is outside target range — check airflow balance" |
| **Low** | General optimisation opportunities | "System is operating within normal parameters — consider adjusting setpoints for overnight energy savings" |

The engine always returns **at least one insight**, even when all readings are normal, ensuring operators always have a starting point for optimisation.

Each insight includes: title, description, priority, affected component, and — where relevant — a **potential savings estimate** when energy inefficiency is detected.

---

## 10. Example Use Cases

### Use Case 1 — Real-Time Energy Optimisation
**Who:** Facility Manager
**Scenario:** The energy bill for last month was 12% above budget. The manager queries `/api/twin/kpis` and sees Total Power flagged as Warning at 165 kW and COP at 2.4. The Insights Engine recommends checking condenser water setpoints. The operations team adjusts the cooling tower target, COP rises to 3.1, and power drops to 138 kW within the hour.

### Use Case 2 — Early Fault Detection
**Who:** Operations Team
**Scenario:** A refrigerant pressure reading begins drifting upward over several simulation steps. The Anomaly Detector raises a Critical alert on `refrigerant_pressure` at 812 kPa. The team schedules a refrigerant check three days before the pressure would have caused a compressor trip, avoiding an unplanned outage worth approximately £8,000 in lost cooling and emergency labour.

### Use Case 3 — What-If Planning for a New Site
**Who:** Engineering Team
**Scenario:** The company is evaluating installing the same chiller at a facility at 1,800 m altitude in a hot climate. The team calls `/api/twin/predictions?method=physics&altitude=1800&outdoor_temp=38&occupancy_load=0.9` and receives predicted COP of 2.1 and Total Power of 192 kW — both in Warning range. The result informs the decision to upsize the chiller before procurement.

### Use Case 4 — Compliance Reporting
**Who:** Sustainability Lead
**Scenario:** An annual energy audit requires evidence of HVAC efficiency over the past quarter. The lead calls `/api/twin/telemetry?last_n=500` to export the rolling telemetry history, then queries `/api/twin/kpis` for the current efficiency snapshot. The timestamped data is submitted directly to the auditor as objective performance evidence.

### Use Case 5 — Predictive Maintenance Scheduling
**Who:** Building Manager
**Scenario:** The Insights Engine has flagged recurring anomalies on `supply_air_pressure` over several days. Rather than waiting for the next scheduled service in six weeks, the manager raises a maintenance work order immediately. The filter is found to be 70% blocked. Replacing it early restores full airflow and avoids a fan motor overload.

### Use Case 6 — Operator Training
**Who:** New HVAC Technician
**Scenario:** A newly hired technician uses the `POST /api/twin/simulate` endpoint repeatedly to observe how sensor readings evolve over a simulated day, including injected fault patterns (spikes, drifts, stuck sensors). The `POST /api/twin/reset` endpoint lets them start fresh after each training session, with zero risk to real equipment.

---

## 11. How to Use the Application

The digital twin exposes a REST API. All interactions are standard HTTP calls that any dashboard, BMS, or integration tool can consume.

### API Endpoint Reference

| Method | Endpoint | What It Does |
|---|---|---|
| `GET` | `/api/twin` | Returns the complete current state of both AHU and Chiller |
| `GET` | `/api/twin/telemetry` | Returns recent telemetry history (default: last 50, max: 500) |
| `GET` | `/api/twin/kpis` | Returns all four KPIs with Good / Warning / Critical status |
| `GET` | `/api/twin/anomalies` | Lists all currently active anomalies with severity and context |
| `GET` | `/api/twin/insights` | Returns prioritised, plain-language recommendations |
| `GET` | `/api/twin/predictions` | Predicts future KPIs — requires `?method=physics` or `?method=ml` |
| `POST` | `/api/twin/simulate` | Advances the simulation by one step and generates new telemetry |
| `POST` | `/api/twin/reset` | Resets the twin to its default initial state |

### Query Parameters

**`/api/twin/telemetry`**
- `last_n` (integer, default 50) — number of snapshots to return
- `component` (`ahu` or `chiller`) — filter to a single component

**`/api/twin/predictions`**
- `method` (**required**) — `physics` or `ml`
- `outdoor_temp` (float, °C) — ambient dry-bulb temperature
- `hour_of_day` (integer, 0–23) — time of day
- `occupancy_load` (float, 0.0–1.0) — building occupancy fraction
- `wet_bulb_temp` (float, °C) — optional; estimated automatically if omitted
- `altitude` (float, metres) — optional; defaults to 0 (sea level)

### Common Workflows

**Check system health right now:**
```
GET /api/twin/kpis
GET /api/twin/anomalies
GET /api/twin/insights
```

**Run a simulation cycle and review results:**
```
POST /api/twin/simulate     ← advance one step
GET  /api/twin              ← view updated state
GET  /api/twin/kpis         ← review new KPIs
```

**Predict performance for tomorrow's heatwave:**
```
GET /api/twin/predictions?method=physics&outdoor_temp=40&occupancy_load=1.0&hour_of_day=14
GET /api/twin/predictions?method=ml&outdoor_temp=40&occupancy_load=1.0&hour_of_day=14
```

**Export recent telemetry for analysis:**
```
GET /api/twin/telemetry?last_n=500
GET /api/twin/telemetry?last_n=500&component=chiller
```

**Reset for a clean demo or training session:**
```
POST /api/twin/reset
```

### Understanding Responses

All responses are validated JSON. Key fields to look for:

- **`status`** on a KPI — `good`, `warning`, or `critical`
- **`severity`** on an anomaly — `warning` or `critical`
- **`priority`** on an insight — `high`, `medium`, or `low`
- **`parameters`** in a physics prediction — shows intermediate values like Carnot COP, condenser temperature, and air density correction for full transparency
- **`step_count`** — a counter that increments with every simulation step, useful for tracking how far the simulation has run

---

## Glossary

| Term | Definition |
|---|---|
| **AHU** | Air Handling Unit — conditions and circulates air throughout the building |
| **Chiller** | Produces chilled water to remove heat from the building |
| **COP** | Coefficient of Performance — cooling output (kW) divided by power input (kW) |
| **Carnot COP** | Theoretical maximum COP based on operating temperatures; used as an upper bound |
| **LMTD** | Log Mean Temperature Difference — used in heat exchanger performance calculations |
| **Wet-Bulb Temperature** | Temperature reflecting combined heat and humidity; governs cooling tower performance |
| **Air Density Correction** | Factor accounting for reduced air density at altitude; affects fan power and heat transfer |
| **Telemetry** | A timestamped snapshot of all sensor readings at a given simulation step |
| **KPI** | Key Performance Indicator — a derived metric summarising system efficiency |
| **Anomaly** | A sensor reading that has crossed a defined safety or performance threshold |
| **Insight** | A plain-language, prioritised recommendation generated from KPIs and anomalies |
| **Twin State** | The complete data object representing the current state of the digital twin |

---

*HVAC Digital Twin Platform — Business Overview & User Guide*
