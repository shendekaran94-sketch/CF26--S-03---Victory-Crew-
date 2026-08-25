Absolutely. I went through the available full-code sections and pulled the README details from the actual implementation rather than guessing. The important mechanics—8 services, 11 dependency links, deterministic cascade propagation, weighted health, resilience scoring, scenarios, What-If prediction, recovery, and demo mode—are all present in the code.  

Here is a **complete `README.md`** you can put directly in your GitHub repository. I left the team names as placeholders because they aren't in the code.

# 🛡️ CityGuard — Urban Infrastructure Cascade Simulator

**Smart City Operations & Resilience Platform**

CityGuard is a browser-based simulation platform that demonstrates how failures in one urban infrastructure service can propagate through interconnected services and create a cascading disruption.

The system models an urban infrastructure dependency network, simulates failures and recovery, calculates live resilience metrics, and provides predictive **What-If analysis** before a failure is actually introduced.

> **Academic / Hackathon Project:** CityGuard uses simulated infrastructure data only. It is **not connected to or controlling any real-world city infrastructure.**

---

## 1. Problem Statement

Modern cities depend on interconnected infrastructure systems such as electricity, communications, transportation, water, emergency response, hospitals, and public safety.

A failure in one service may affect other services that depend on it. For example:

**Power Grid Failure → Traffic / Metro / Emergency Response → Hospital / Public Safety**

The challenge is that infrastructure failures should not be considered independently. A disruption in one system can propagate through dependency relationships and create a larger **cascade effect**.

### Problem

How can we visualize, simulate, and analyze the cascading impact of urban infrastructure failures in an understandable way?

### Objective

CityGuard aims to provide an interactive simulation that can:

* Model infrastructure dependencies.
* Simulate individual and multiple service failures.
* Propagate failures through the dependency network.
* Identify critical services at risk.
* Measure cascade depth and affected services.
* Estimate overall city resilience.
* Predict possible consequences before running a scenario.
* Simulate progressive infrastructure recovery.

---

# 2. Solution Overview

CityGuard represents urban infrastructure as a **directed dependency graph**.

Each infrastructure service is represented as a node, while dependency relationships are represented as directed edges.

For example:

```text
Power Grid
   ├──→ Traffic Control
   ├──→ Emergency Response
   ├──→ Hospital Network
   └──→ Metro System

Communication Network
   ├──→ Emergency Response
   └──→ Public Safety

Water Supply
   └──→ Hospital Network
```

When a provider fails, the simulation evaluates the services that depend on it.

A dependent service can become:

* `HEALTHY`
* `DEGRADED`
* `FAILED`
* `RECOVERING`

The simulation then updates the graph, metrics, incident status, timeline, analytics, and resilience score.

---

# 3. System Architecture / Workflow

## High-Level Architecture

```text
                 ┌─────────────────────┐
                 │      User Input     │
                 │ Fail / Scenario /   │
                 │ What-If / Recovery  │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   City Infrastructure│
                 │    Dependency Graph │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Cascade Engine    │
                 │ Wave-Based Graph     │
                 │    Propagation       │
                 └──────────┬──────────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
       ┌──────────────────┐   ┌──────────────────┐
       │ Impact / Metrics │   │ What-If Engine   │
       │ Health / Depth   │   │ Predictive       │
       │ Critical Risk    │   │ Projection       │
       └────────┬─────────┘   └──────────────────┘
                │
                ▼
       ┌──────────────────────┐
       │ Resilience & Incident│
       │ Management Dashboard │
       └──────────┬───────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │ Recovery Simulation  │
       │ Progressive Restore  │
       └──────────────────────┘
```

## Simulation Workflow

### 1. Detect

A service can fail through:

* Manual failure action.
* Predefined scenario.
* Custom scenario.
* Automated demo mode.

The failed service becomes the root of the incident.

### 2. Propagate

The cascade engine evaluates the dependency graph wave by wave.

A failed provider contributes **1.0 impairment** to a dependent service.

A degraded provider contributes **0.5 impairment**.

The implementation uses these thresholds:

```text
Impairment >= 1.5  → FAILED
Impairment > 0     → DEGRADED
No impairment      → HEALTHY
```

This mechanism is implemented by `computeCascade()` in the simulation engine. 

### 3. Analyze

The system calculates:

* Number of affected services.
* Failed services.
* Degraded services.
* Recovery services.
* Cascade depth.
* Critical services at risk.
* Weighted system health.
* Resilience score.
* Incident severity.

These values are derived from the current simulation state rather than being manually displayed as fixed results. 

### 4. Recover

Services can be restored individually, all at once, or using a critical-first recovery strategy.

When a provider becomes healthy again, dependent services may automatically recover based on the dependency graph. 

---

# 4. Infrastructure Model

CityGuard currently models **8 urban services**.

| Service                  | Critical | Main Dependencies / Role                        |
| ------------------------ | -------- | ----------------------------------------------- |
| ⚡ Power Grid             | Yes      | Major infrastructure provider                   |
| 📡 Communication Network | Yes      | Emergency and public-safety communication       |
| 💧 Water Supply          | No       | Hospital dependency                             |
| 🚦 Traffic Control       | No       | Emergency and metro dependency                  |
| 🚇 Metro System          | No       | Depends on power and traffic                    |
| 🚑 Emergency Response    | Yes      | Depends on power, communication and traffic     |
| 🏥 Hospital Network      | Yes      | Depends on power, water and emergency response  |
| 🚨 Public Safety         | Yes      | Depends on communication and emergency response |

The model contains **11 directed dependency links**. 

---

# 5. Core Technical Mechanism

## Dependency Graph

Dependencies are stored as directed provider → dependent relationships.

Example:

```javascript
["power", "hospital"]
```

means:

```text
Power Grid → Hospital Network
```

The application builds the graph dynamically and renders it using SVG paths and HTML nodes. 

---

## Cascade Propagation

The cascade engine starts with one or more failed root services.

Each service starts as:

```text
HEALTHY
```

Root failures are immediately set to:

```text
FAILED
```

The engine then evaluates every dependent service.

### Failure weighting

```text
FAILED provider     = 1.0 impairment
DEGRADED provider   = 0.5 impairment
```

### State transition

```text
score >= 1.5 → FAILED
score > 0    → DEGRADED
score = 0    → HEALTHY
```

The engine repeatedly evaluates the network until no additional state changes occur. 

This produces cascade waves such as:

```text
Wave 0
Initial failure

        ↓

Wave 1
Directly affected services

        ↓

Wave 2
Secondary effects

        ↓

Wave 3
Further propagation
```

Cascade depth is calculated from these propagation levels.

---

# 6. Health & Resilience Calculation

CityGuard uses weighted service health.

Critical services have a weight of **2**, while standard services have a weight of **1**.

The total health loss is calculated from service states:

```text
FAILED       → 100% weighted loss
DEGRADED     → 50% weighted loss
RECOVERING   → 30% weighted loss
HEALTHY      → 0% loss
```

The system health percentage is then calculated from the weighted loss relative to total service weight. 

## Resilience Score

The resilience score is calculated as:

```text
Score =
    Health
    − (Cascade Depth × 2)
    − (Critical Services Affected × 2)
    − ((Root Failures − 1) × 2)
```

The final value is constrained between:

```text
0 and 100
```

Implementation:

```javascript
function scoreFrom(health, depth, criticalHit, rootCount){
  let score =
    health
    - depth * 2
    - criticalHit * 2
    - Math.max(0, rootCount - 1) * 2;

  return Math.max(0, Math.min(100, Math.round(score)));
}
```



### Resilience Grades

|  Score | Grade     |
| -----: | --------- |
| 90–100 | EXCELLENT |
|  75–89 | STABLE    |
|  55–74 | STRAINED  |
|  35–54 | AT RISK   |
|   0–34 | CRITICAL  |



---

# 7. Key Features

## 🗺️ City Digital Twin

An interactive dependency graph displays all monitored infrastructure services.

Users can:

* View dependencies.
* See service health states.
* Hover over nodes for quick actions.
* Simulate failures.
* Recover services.
* Open the Service Inspector.

The graph visually changes as failures propagate. 

---

## 🚨 Incident Management

Each disruption can create an incident such as:

```text
INC-001
INC-002
INC-003
```

Incidents track:

* Start time.
* Initial failure.
* Severity.
* Affected services.
* Cascade depth.
* Recovery progress.
* Resolution duration.

Severity can become:

```text
LOW
MODERATE
HIGH
CRITICAL
```

depending on affected critical services and resilience score. 

---

## 🔮 What-If Analysis

The What-If engine allows users to select one or more services and predict the consequences **without changing the live simulation**.

It calculates:

* Predicted affected services.
* Predicted cascade depth.
* Critical services at risk.
* Recovery complexity.
* Estimated restoration time.
* Predicted resilience score.

The prediction uses a pure projection of the cascade model, meaning it does not mutate the live simulation state. 

---

## 🎯 Scenario Center

The application includes predefined scenarios:

1. **Power Grid Failure**
2. **Traffic Network Failure**
3. **Communication Outage**
4. **Water Supply Disruption**
5. **Emergency Response Failure**
6. **Major City Crisis**

The Major City Crisis combines:

```text
Power + Communication + Water
```

to demonstrate a multi-domain disruption.  

Users can also create custom scenarios by selecting services.

---

## ⟳ Recovery Center

Recovery can be performed through:

* Individual service restoration.
* Recover All.
* Recover Critical.

A service takes approximately **2.2 simulated seconds** to complete an individual restoration action in the current implementation.

When all services become healthy, the incident is resolved and the application records the recovery duration. 

---

## 📊 Analytics

The dashboard provides:

* Service Health Index.
* Resilience Score History.
* Status Distribution.
* Maximum cascade depth.
* Peak affected services.
* Session incident count.
* Highest severity.
* Last recovery duration.

The score history keeps the latest 40 sampled values. 

---

## 🖥️ Command Center Mode

Command Center mode provides a larger operational view of the dependency graph and emphasizes the primary monitoring interface.

It adjusts the layout and graph presentation for a command-center style display. 

---

# 8. Automated Demo Mode

CityGuard includes an automated demonstration sequence.

The demo approximately follows this progression:

```text
Baseline
   ↓
Monitor infrastructure
   ↓
Power Grid warning
   ↓
Power Grid failure
   ↓
Cascade analysis
   ↓
Communication Network failure
   ↓
Multi-domain impact
   ↓
Recovery
   ↓
Full operational status
```

The demo includes timed events for warnings, failures, analysis, recovery, and completion. It can also be paused and resumed.  

---

# 9. Technology Stack

CityGuard is intentionally implemented as a lightweight browser application.

### Frontend

* HTML5
* CSS3
* Vanilla JavaScript
* SVG

### UI / Visualization

* CSS Grid
* Responsive CSS
* SVG dependency graph
* Animated status indicators
* Interactive dashboard components

### Simulation

* JavaScript-based deterministic simulation engine
* Directed dependency graph
* Wave-based cascade propagation
* Client-side state management
* Timed simulation events

### Dependencies

**No external JavaScript libraries or frameworks are required by the current single-file implementation.**

The source itself describes the build as a:

> "Single-file build. No dependencies. Deterministic simulation."



---

# 10. Project Structure

The current implementation can run as a single HTML file:

```text
CityGuard/
│
├── index.html
└── README.md
```

The HTML file contains:

```text
HTML
├── Dashboard structure
│
├── CSS
│   ├── Layout
│   ├── Components
│   ├── Graph styling
│   └── Responsive design
│
└── JavaScript
    ├── City model
    ├── Simulation state
    ├── Graph rendering
    ├── Cascade engine
    ├── Incident management
    ├── Recovery
    ├── Metrics
    ├── What-If engine
    ├── Scenario center
    ├── Demo mode
    └── UI initialization
```

---

# 11. Setup & Installation

## Requirements

A modern web browser is sufficient.

Recommended:

* Google Chrome
* Microsoft Edge
* Mozilla Firefox

No backend server or package installation is required for the current single-file version.

## Run Locally

### Option 1 — Directly

Download or clone the repository and open the HTML file in a browser.

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Then open:

```text
index.html
```

in your browser.

### Option 2 — VS Code Live Server

1. Open the project folder in VS Code.
2. Open `index.html`.
3. Install/use the **Live Server** extension.
4. Click **Go Live**.
5. Open the displayed local address in your browser.

---

# 12. Usage Instructions

## Run the automated demonstration

Click:

```text
▶ Run Demo
```

The application automatically demonstrates an infrastructure cascade and recovery sequence.

---

## Simulate a failure manually

1. Find a service in the dependency graph.
2. Hover over the service.
3. Click **FAIL**.
4. Watch the dependency network update.
5. Observe:

   * Health
   * Affected services
   * Cascade depth
   * Critical services
   * Resilience score
   * Event stream
   * Incident timeline

---

## Run a predefined scenario

1. Open **Scenarios**.
2. Select a scenario.
3. Click **Run Scenario**.
4. Observe the resulting cascade.

---

## Test a custom scenario

1. Open **Scenarios**.
2. Select one or more services under **Create Custom Scenario**.
3. Click **Run Custom Scenario**.

---

## Use What-If Analysis

1. Open **What If?**
2. Select one or more infrastructure services.
3. Click **Predict Impact**.
4. Review the predicted:

   * Affected services.
   * Cascade depth.
   * Critical risk.
   * Recovery complexity.
   * Resilience score.

The live simulation is not changed until **Run This Scenario For Real** is selected. 

---

## Recover the city

Use:

```text
⟳ Recover All
```

or:

```text
✚ Recover Critical
```

You can also restore individual services through the Recovery Center or Service Inspector.

---

# 13. Validation / Experiments / Results

CityGuard was designed so that simulation outputs are calculated from the dependency graph and current state.

The following validation scenarios are available directly through the application.

## Experiment 1 — Power Grid Failure

**Input:**

```text
Power Grid → FAILED
```

Expected behavior:

```text
Power Grid
    ↓
Traffic Control
    ↓
Emergency Response
    ↓
Hospital / Public Safety
```

Power also directly affects Metro System and Hospital Network according to the dependency model.

The dashboard should show increased affected services and reduced resilience.

---

## Experiment 2 — Communication Outage

**Input:**

```text
Communication Network → FAILED
```

The dependency graph propagates the disruption toward:

```text
Communication Network
       ├──→ Emergency Response
       └──→ Public Safety
```

This allows the impact on emergency and public-safety coordination to be observed.

---

## Experiment 3 — Major City Crisis

**Input:**

```text
Power Grid
Communication Network
Water Supply
```

are failed simultaneously.

This tests a multi-domain disruption and demonstrates how multiple root failures can produce a larger cascade.

The predefined scenario is explicitly implemented in the Scenario Center. 

---

## Experiment 4 — What-If Prediction

Select one or more services in the What-If panel.

The application calculates the projected cascade using the same dependency model while leaving the live state unchanged.

This provides a way to compare a predicted outcome before executing the scenario. 

---

## Experiment 5 — Recovery

After creating a failure:

1. Start recovery.
2. Restore individual or critical services.
3. Observe dependent services automatically becoming healthy when their dependencies are restored.
4. Continue until all services return to `HEALTHY`.
5. Verify that the incident is marked `RESOLVED`.

The recovery system records the simulated recovery duration. 

---

# 14. Validation Criteria

A scenario is considered to behave correctly when:

* The selected root service enters `FAILED`.
* Dependent services are evaluated according to the dependency graph.
* Degraded/failed states propagate according to the impairment thresholds.
* Cascade depth increases when secondary effects occur.
* Critical services are identified.
* Health and resilience values change.
* Incident information is updated.
* Recovery restores services.
* The simulation eventually returns to a fully healthy state after successful recovery.

The application also exposes the calculated state through its dashboard, event stream, timeline, analytics, and incident panels.

---

# 15. Limitations

CityGuard is a **simulation and educational prototype**, not a production infrastructure management system.

### Current limitations

1. **Simulated data only**

   The system is not connected to live infrastructure, sensors, SCADA systems, traffic systems, hospital systems, or city databases.

2. **Small infrastructure model**

   The current model contains 8 services and 11 dependency links.

3. **Deterministic rules**

   Cascade behavior is based on predefined impairment thresholds rather than real-world probabilistic failure models.

4. **Simplified dependencies**

   Real infrastructure systems have much more complex relationships, redundancy, geographic constraints, capacity limits, and human factors.

5. **No live telemetry**

   The application does not ingest real-time sensor or infrastructure data.

6. **No persistent backend**

   Simulation state is maintained in browser memory.

7. **Recovery is simulated**

   Recovery durations and behavior are demonstration values rather than predictions of real operational repair times.

8. **No real-world decision authority**

   The system is intended for visualization, education, experimentation, and demonstration—not for directly controlling infrastructure.

The application itself explicitly identifies the simulation as academic and states that it is not connected to real city infrastructure.

---

# 16. Future Scope

Potential future improvements include:

### 🌐 Real-Time Data Integration

Connect the model to:

* IoT sensors.
* Weather feeds.
* Traffic APIs.
* Power-grid telemetry.
* Water monitoring systems.

### 🧠 Advanced AI Prediction

Use machine learning to estimate:

* Failure probability.
* Cascade probability.
* Recovery time.
* High-risk infrastructure nodes.

### 🗺️ Geographic Digital Twin

Add an actual city map and geographic infrastructure locations.

### 🔁 Redundancy Modeling

Support:

* Backup power.
* Alternate communication routes.
* Emergency capacity.
* Multiple water sources.
* Failover systems.

### 📈 Historical Analysis

Store simulation runs and compare:

```text
Scenario A vs Scenario B
```

over time.

### 👥 Multi-User Command Center

Allow multiple operators to collaborate during a simulated incident.

### 🏙️ Larger Infrastructure Network

Expand beyond 8 services to include:

* Waste management.
* Fuel distribution.
* Airports.
* Railways.
* Food supply.
* Data centers.
* Financial infrastructure.

---

# 17. AI Assistance Disclosure

AI tools were used during development of this hackathon project.

AI assistance was used to support activities such as:

* Generating and refining parts of the frontend implementation.
* Assisting with HTML/CSS/JavaScript development.
* Improving UI structure and presentation.
* Assisting with debugging and implementation ideas.
* Helping structure the simulation and dashboard concepts.

The project team reviewed, integrated, tested, and demonstrated the resulting implementation.

**AI assistance does not imply that the application uses an external AI model at runtime.** The current CityGuard simulation engine itself is implemented using client-side JavaScript rules and deterministic graph propagation.

---

# 18. Team Members

| Name                | Role                       |
| ------------------- | -------------------------- |
| **[Team Member 1]** | Project Lead / Integration |
| **[Team Member 2]** | Frontend / UI              |
| **[Team Member 3]** | Simulation / Logic         |
| **[Team Member 4]** | Testing / Presentation     |

> Replace the placeholders with your actual team members and roles before submitting.

---

# 19. Hackathon Contribution

CityGuard demonstrates how a relatively simple dependency graph can be used to communicate a complex urban-resilience concept interactively.

Instead of displaying only static information, the project allows a reviewer to:

```text
Choose a failure
      ↓
Observe propagation
      ↓
Measure impact
      ↓
Predict another scenario
      ↓
Run it
      ↓
Recover the infrastructure
      ↓
Measure resilience again
```

This makes the infrastructure cascade concept directly observable rather than purely theoretical.

---

# 20. Project Status

**Status:** Hackathon Prototype / Academic Demonstration

**Simulation:** Deterministic

**Infrastructure:** Simulated

**Runtime:** Browser

**External Dependencies:** None required by the current single-file build

**Real Infrastructure Connection:** None

---

## Disclaimer

CityGuard is a software simulation using simulated city infrastructure data.

It is not connected to, monitoring, controlling, or making decisions for any real city infrastructure.

Built for academic and hackathon demonstration purposes.
