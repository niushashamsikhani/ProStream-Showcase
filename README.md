<p align="center"><img src="docs/images/banner.svg" alt="ProStream: intelligent automotive assembly process and time-estimation platform" width="100%"></p>

<p align="center"><b>4–10 %</b> deviation from stopwatch &nbsp;·&nbsp; <b>14</b> AI-classified vehicle zones &nbsp;·&nbsp; <b>MTM + BasicMOST</b> integrated &nbsp;·&nbsp; <b>~35</b> REST endpoints</p>

<p align="center"><img src="docs/images/3d-explorer.png" alt="ProStream 3D vehicle process explorer" width="900"></p>
<p align="center"><sub>3D Vehicle Process Explorer: processes mapped to vehicle zones, with AI-mapping confidence.</sub></p>

> **Note:** This repository is a project showcase. The source code and production data belong to an industrial collaboration and are kept private for confidentiality. A live demo can be provided on request.

## Motivation

Traditional stopwatch time studies are slow, resource-intensive and prone to evaluator bias and measurement error. At the same time, a modern vehicle line contains roughly 800–1,000 assembly operations that must be balanced across workstations under precedence and operational constraints, a task that can take 30–60 working days by hand.

ProStream addresses both problems: it estimates standard times **without physical presence on the assembly line**, and it structures process knowledge so it can be reused and analysed.

## System overview

| Layer | What it does |
|---|---|
| **1. Extraction** | An OCR/NLP pipeline reads assembly process sheets and converts work elements, tools and operation sequences into structured database records. |
| **2. Estimation** | A heuristic framework combines tool-use time, manual-work time and allowances with established work-measurement techniques (**BasicMOST** and **MTM**) to estimate **Normal Time**, **Cycle Time** and **Design Standard Time**. |
| **3. Exploration & reporting** | A 3D vehicle explorer and a dynamic reporting dashboard give fast, flexible access to processes and key performance indicators. |

<p align="center"><img src="docs/images/dashboard.png" alt="ProStream reporting dashboard" width="900"></p>
<p align="center"><sub>Reporting dashboard: key performance indicators and a detailed per-process report.</sub></p>

## Architecture

```mermaid
flowchart LR
    A[Operation sheets] --> B[OCR / NLP extraction]
    B --> C[(PostgreSQL)]
    C --> D[Flask REST API]
    D --> E[React web app]
    E --> F[Dashboard & reports]
    E --> G[3D vehicle explorer]
    H[Calibrated timing parameters] --> D
```

ProStream is a full-stack system that I designed and built end to end: data extraction, a relational data model, a REST API, the estimation logic and the web interface.

## Data model and API

The PostgreSQL schema covers assembly processes, vehicle models, tools and tool stations, calibrated parameter tables (tool-type, manual-task, shipping and unpacking parameters) and stored calculation results. A REST API with about 35 endpoints exposes these resources and computes **Design Standard Time**. Schema changes are managed with versioned migrations.

## Standard-time calculation

For each part operation, engineers configure the tool type and tool time, manual-task parameters, logistics (shipping) time, unpack grade and allowance percentage. ProStream then computes **Normal Time (N.T)**, **Cycle Time (C.T)** and **Design Standard Time (DST)** per part operation and for the whole process, and exports an official time sheet to Excel or PDF.

<p align="center"><img src="docs/images/time-calculation.png" alt="Standard-time calculation view" width="900"></p>
<p align="center"><sub>Time calculation: task parameters, allowance and the resulting N.T, C.T and DST.</sub></p>

## AI-assisted 3D vehicle explorer

Assembly processes are classified into **14 predefined vehicle zones**: engine bay, windshield & wipers, four doors, dashboard, roof, interior pillars, seats & seat belts, trunk, chassis & underbody, front fascia, and wheels & tires.

<p align="center"><img src="docs/images/3d-overview.png" alt="3D explorer overview with zone tabs" width="900"></p>
<p align="center"><sub>Zone tabs with process counts; click a zone or a part of the model to see its assembly processes. The vehicle mesh is a third-party model used for demonstration.</sub></p>

**Classification and expert validation.** A large language model evaluates each process against the 14 controlled zones using structured process context and returns a zone, a vehicle side, a sub-zone and a **confidence score**. Predictions at or above 70 % confidence without a review flag are marked as high confidence; all others enter a review queue. A dedicated mapping console lets experts classify processes in batch, filter them by status (high confidence, needs review, failed, manual, pending), and confirm or override any prediction. Manual decisions are stored with full confidence and tracked separately from AI predictions.

<p align="center"><img src="docs/images/ai-mapping.png" alt="AI vehicle mapping console" width="900"></p>
<p align="center"><sub>AI Vehicle Mapping console: primary zone, confidence, review status and source for every process.</sub></p>

**Interactive exploration.** Engineers can:
- hover and click zones directly on the 3D model, with live outline highlighting
- fly the camera to a zone-specific viewpoint, with orbit, tilt and zoom controls and an interior view
- search processes by code, operation name, model, part name or part number, and focus the matching zone
- review the processes of a zone in a side panel, edit a process, or export it as a document

The viewer is model-agnostic: each vehicle has its own configuration (glTF asset, mesh-to-zone dictionary, camera and transform), and unknown models fall back to a default configuration. Models are streamed as Meshopt-compressed glTF, and loading failures degrade gracefully.

## Validation

ProStream's estimates were compared with stopwatch measurements across the evaluated processes and showed a deviation of **4–10 %**. In the sample below, ProStream's estimates are consistently slightly lower than the stopwatch values. Calibration of the timing parameters is iterative: deviations are analysed per process, and the parameters of the timing model are adjusted accordingly.

<p align="center"><img src="docs/images/calibration.png" alt="ProStream versus stopwatch comparison" width="650"></p>
<p align="center"><sub>ProStream versus stopwatch time for three sample processes, with relative deviation.</sub></p>

### Efficiency

Extracting process data and searching for it with AI takes a fraction of the time needed manually.

<table align="center">
  <tr>
    <td width="50%"><img src="docs/images/extraction-time.png" alt="Extraction time: AI-powered versus manual" width="100%"></td>
    <td width="50%"><img src="docs/images/search-time.png" alt="Search time: AI-powered versus manual" width="100%"></td>
  </tr>
</table>

<p align="center"><sub>Processing time, AI-powered versus manual: data extraction (left) and search (right).</sub></p>

## Ongoing research

My dissertation investigates **machine-learning-based assembly-line balancing**: using process times, precedence relations and production-hall constraints to build an optimization model for task allocation, workstation workload distribution and operator requirements. Model evaluation is still pending.

## Technology

- **Frontend:** React · TypeScript · Three.js · React Three Fiber · drei · glTF (Meshopt)
- **Backend:** Python · Flask · SQLAlchemy · Alembic migrations · REST API
- **Database:** PostgreSQL
- **AI:** OCR/NLP document extraction · LLM-based zone classification (Google Gemini) · heuristic estimation models

## My role

Technical Lead. I conceived the project and independently developed the end-to-end platform, from problem formulation and the estimation model to the data pipeline, the API and the web application.

## Status

In development since August 2024 in collaboration with an automotive manufacturer.

## Contact

Niusha Shamsikhani · niusha.s2015@gmail.com
