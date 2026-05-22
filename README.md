
<div align="center">

## What is Aegis?
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/d50003da-afb7-4712-bf82-32a556a729fe" />

---
Aegis is rather simple; implementation of base technicals; MACD, BB, RSI to create signals ("Signals" are simply the conventional 'breakouts' or 'bullish' intervals of all TIs, aggregated). Clean signals are passed through VMD decomposition to strip microstructure noise, then fed into Gemma 4 31B (no specialised fine-tuning) to estimate profitability, OHLVC is aggregated and simplified to variables through a scoring system [high - probability of profit (POP): 0.5<, Low - POP: 0-0.5, max: 1), the estimate is continously refined via a bayesian updater, execution is routed through Alpaca API and only proceeds if P(Profit) > 60% and net yield clears transaction costs.

<div align="center">

---
## System Architecture

</div>

```mermaid
flowchart TD
    A[(NASDAQ\nTick Feed)]
    
    A --> B[OHLCV Aggregation\nOrder Book Depth]
    B --> C[Technical Indicators\nRSI · MACD · BB]
    C --> D[VMD De-noising\nSignal / Noise Separation]
    
    D --> E{Volatility > 3σ ?}
    E -- Yes --> HOLD
    E -- No --> F[BiGRU\nProfitability Prediction]
    
    F --> G[Bayesian Updater\nPosterior ∝ Likelihood × Prior]
    G --> H{P Profit > 60% ?}
    
    H -- No --> HOLD
    H -- Yes --> I{Net Yield\nAfter Costs > 0 ?}
    
    I -- No --> HOLD([Hold / No Edge])
    I -- Yes --> J([Execute\nLimit / Market Order])
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

---

<div align="center">

## Research Notes

> **`[STATUS: NON-VIABLE]`**

---

Raw tick data is decomposed via VMD minimisation to isolate tradeable signal components from market microstructure noise:

$$f(\alpha, \beta, \gamma) = \min_{k,\,\beta,\,\gamma} \{ \| \delta(t) - \sum_{k}^{n} u_k(t) \| + \alpha \| \partial_t [ u_k(t)\,e^{-i\omega_k t} ] \| \}$$

Extracts discrete sub-signals $u_k$ from raw input $\delta(t)$. The penalty term $\alpha$ regularises the derivative of each frequency-shifted mode, suppressing high-frequency noise without distorting the underlying signal structure.

---

Post-BiGRU, the system applies standard Bayesian inference over a rolling state:

$$\text{Posterior}(\theta \mid y) \;\propto\; P(y \mid \theta)\,P(\theta)$$

Each inference cycle maps the prior $P(\theta)$ against the likelihood of observed market data $P(y \mid \theta)$ to yield a continuous posterior distribution over signal validity. The posterior is recycled as the next prior, maintaining an adaptive belief state across time.

---

Two hard gates govern execution eligibility:

| Gate | Condition | Description |
|---|---|---|
| Volatility Filter | $\sigma_t \leq 3\sigma$ | Suspends execution during fat-tail regimes; no edge assumed beyond 3 standard deviations |
| Execution Gate | $\psi^* \xi > 0.6$ | Bayesian posterior $P(\text{Profit})$ must exceed 60% before order routing proceeds |

---

**Primary bottleneck:** Pronounced performance decay during structural regime shifts, Bayesian prior state exhibits adaptability drag under non-stationary volatility, causing posterior miscalibration. Regime change analysis ongoing.

<div align="left">
  
---

## Sources

1. Cont, R. & Larrard, A. — *Dynamics of Order Positions and Related Queues in a Limit Order Book*
   https://www.researchgate.net/publication/277023747_Dynamics_of_Order_Positions_and_Related_Queues_in_a_Limit_Order_Book

2. Linux Kernel Documentation — *AF_XDP*
   https://docs.kernel.org/networking/af_xdp.html

3. DPDK — *Poll Mode Driver* (v24.03)
   https://doc.dpdk.org/guides-24.03/prog_guide/poll_mode_drv.html

4. DPDK — *OPDL Event Device* (v24.07)
   https://doc.dpdk.org/guides-24.07/eventdevs/opdl.html

5. DPDK — *ENA NIC Driver* (v25.03)
   https://doc.dpdk.org/guides-25.03/nics/ena.html

6. DPDK — *ICE NIC Driver* (v25.11)
   https://doc.dpdk.org/guides-25.11/nics/ice.html

7. DPDK Power Docs — *ENA NIC*
   https://dpdk-power-docs.readthedocs.io/en/latest/nics/ena.html

8. NVIDIA — *TensorRT Developer Guide*
   https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/

9. AMD/Solarflare — *Enterprise Onload User Guide* (SF-104474-CD-34)
   https://www.amd.com/content/dam/amd/en/support/downloads/solarflare/onload/enterprise-onload/SF-104474-CD-34_Onload_User_Guide.pdf
