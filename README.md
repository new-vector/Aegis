<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/d50003da-afb7-4712-bf82-32a556a729fe" />



## System Architecture

</div>

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#1a1a1a",
    "primaryTextColor": "#e0e0e0",
    "primaryBorderColor": "#444444",
    "lineColor": "#888888",
    "secondaryColor": "#2a2a2a",
    "tertiaryColor": "#111111",
    "background": "#000000",
    "mainBkg": "#1a1a1a",
    "nodeBorder": "#555555",
    "clusterBkg": "#222222",
    "titleColor": "#ffffff",
    "edgeLabelBackground": "#1a1a1a",
    "attributeBackgroundColorEven": "#1a1a1a",
    "attributeBackgroundColorOdd": "#111111"
  }
} }%%
flowchart TD
    classDef data fill:#1a1a1a,stroke:#aaaaaa,stroke-width:2px,color:#e0e0e0,font-weight:bold
    classDef front fill:#1a1a1a,stroke:#cccccc,stroke-width:2px,color:#ffffff,font-weight:bold
    classDef middle fill:#111111,stroke:#999999,stroke-width:2px,color:#dddddd,font-weight:bold
    classDef back fill:#222222,stroke:#bbbbbb,stroke-width:2px,color:#eeeeee,font-weight:bold
    classDef exec fill:#0d0d0d,stroke:#ffffff,stroke-width:2px,color:#ffffff,font-weight:bold
    classDef hold fill:#2a2a2a,stroke:#666666,stroke-width:2px,color:#aaaaaa,font-weight:bold

    subgraph Data["  HIGH-FREQUENCY MARKET DATA FEEDS  "]
        A1[(NASDAQ\nTick Data)]:::data
    end

    subgraph Front["  FRONT-END · SIGNAL ENGINEERING  "]
        B1["Micro-Market Data Aggregation\nOrder Book  ·  OHLCV Ticks"]:::front
        B2["Technical Indicator\nCalculation Engine"]:::front
        B3["Fast VMD\nDecompose Noise from Signal"]:::front
    end

    subgraph Middle["  MIDDLE-END · BAYESIAN PROBABILITY ENGINE  "]
        C1["BiGRU Network\nBidirectional Sequence Feature Extraction"]:::middle
        C2["Likelihood Estimator\nP( Signal | Profit )"]:::middle
        C3(["Bayesian Updater\nPosterior  ∝  Likelihood  ×  Prior"]):::middle
        C4[("Rolling Bias State\nPosterior → Next Prior")]:::middle
        C5{"P( Profit ) > 60% ?"}:::middle
    end

    subgraph Back["  BACK-END · T+0 STRATEGY LOGIC  "]
        D1["Micro-Transaction Cost Calc\nSpread  +  Exchange Fees"]:::back
        D2{"Net Expected\nYield > 0 ?"}:::back
        D3["Order Router\nDirect Market Access"]:::back
    end

    subgraph Exec["  EXECUTION LAYER  "]
        E1(["NASDAQ Trade\nHigh-Freq Limit / Market"]):::exec
    end

    Hold(["HOLD / LIQUIDATE\nNo Edge Detected"]):::hold

    A1 --> B1
    B1 --> B2 --> B3 --> C1
    C1 --> C2 --> C3
    C4 -. "Prior\nProbability" .-> C3
    C3 -- "Updates" --> C4
    C3 -- "Current\nProbability" --> C5
    C5 -- "YES › ACTION" --> D1
    C5 -- "NO › INACTION" --> Hold
    D1 --> D2
    D2 -- "Profitable" --> D3
    D2 -- "Costs Too High" --> Hold
    D3 -- "Execute" --> E1
```

---

<div align="center">

## Component Breakdown

</div>

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#1a1a1a", "primaryTextColor": "#e0e0e0", "primaryBorderColor": "#555555", "lineColor": "#777777", "background": "#000000", "mainBkg": "#1a1a1a", "titleColor": "#ffffff"}} }%%
mindmap
  root(("AEGIS\nCore"))
    Data Layer
      NASDAQ Tick Feed
      Order Book Depth
    Signal Engineering
      OHLCV Aggregation
      Technical Indicators
        RSI · MACD · BB
      Fast VMD
        Mode Decomposition
        Noise Suppression
    Prediction System
      BiGRU Network
        Forward Pass
        Backward Pass
      Bayesian Updater
        Likelihood Estimation
        Posterior Rolling State
      Probability Thresholds
    Risk & Execution
      Transaction Cost Model
        Spread Analysis
        Fee Calculation
      Market Routing
```

---

<div align="center">

## Model Performance & Backtest Status

</div>

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#1a1a1a", "primaryTextColor": "#e0e0e0", "primaryBorderColor": "#555555", "lineColor": "#777777", "background": "#000000"}} }%%
xychart-beta
    title "Latency Profile vs. Target Threshold (ms)"
    x-axis ["BiGRU Pass", "VMD Decomp", "Bayesian Upd", "Cost Calc"]
    y-axis "Latency (ms)" 0 --> 1.0
    bar [0.80, 0.45, 0.20, 0.15]
    line [0.05, 0.05, 0.05, 0.05]
```

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#1a1a1a", "primaryTextColor": "#e0e0e0", "background": "#000000", "pie1": "#ffffff", "pie2": "#aaaaaa", "pie3": "#777777", "pie4": "#444444"}} }%%
pie title Latency Budget Breakdown
    "BiGRU Inference" : 41
    "VMD Decomposition" : 29
    "Bayesian Update" : 18
    "Cost Calculation" : 12
```

---

<div align="center">

## Development Roadmap

</div>

```mermaid
%%{init: {"theme": "base", "themeVariables": {"primaryColor": "#1a1a1a", "primaryTextColor": "#e0e0e0", "primaryBorderColor": "#555555", "lineColor": "#777777", "background": "#000000", "mainBkg": "#1a1a1a", "titleColor": "#ffffff", "taskBkgColor": "#1a1a1a", "taskBorderColor": "#555555", "taskTextColor": "#dddddd", "activeTaskBkgColor": "#2a2a2a", "activeTaskBorderColor": "#aaaaaa", "doneTaskBkgColor": "#333333", "doneTaskBorderColor": "#888888", "critBkgColor": "#111111", "critBorderColor": "#ffffff", "sectionBkgColor": "#000000", "altSectionBkgColor": "#0d0d0d", "gridColor": "#333333"}} }%%
gantt
    title AEGIS Timeline
    dateFormat YYYY-MM
    axisFormat %b '%y

    section Phase I · Foundation
    VMD Signal Decomposition        :done,    p1a, 2024-10, 2024-12
    BiGRU Architecture Design       :done,    p1b, 2024-11, 2025-01
    Bayesian Probability Engine     :done,    p1c, 2025-01, 2025-03

    section Phase II · Backtesting
    Initial Backtest Suite          :done,    p2a, 2025-03, 2025-05
    Latency Profiling               :done,    p2b, 2025-04, 2025-06
    Regime Change Analysis          :active,  p2c, 2025-06, 2026-02

    section Phase III · Optimisation
    BiGRU → Linear Logic Refactor   :active,  p3a, 2026-01, 2026-06
    Latency Reduction Target        :         p3b, 2026-03, 2026-08

    section Phase IV · Live Alpha
    Paper Trading Validation        :crit,    p4a, 2026-08, 2026-11
    Alpha Confirmation              :crit,    p4b, 2026-10, 2027-01
    Live Deployment                 :crit,    p4c, 2027-01, 2027-03
```

---

<div align="center">

## Research Notes

</div>

> **`[STATUS: PRE-ALPHA / NON-VIABLE]`**
>
> Current iteration fails strict HFT latency constraints. Backtesting indicates severe performance decay during structural regime shifts due to model adaptability drag. 
> 
> The primary bottleneck is deep learning inference overhead (`0.10–0.80ms`). To achieve true HFT execution bounds, the BiGRU model's feature extraction logic must be distilled and replicated via linearised logic approximations. Development is pivoting to optimise for microsecond-level execution.

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/8c167af7-b2eb-487f-a431-44fe229d514d" />


---

<div align="center">

**AEGIS**
</div>
