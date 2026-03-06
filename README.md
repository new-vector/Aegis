# Dual-market-capital-rotation-strategy-VMD-BiGRU-model

```mermaid
flowchart TD
    classDef data fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef front fill:#e0f2fe,stroke:#0284c7,stroke-width:2px,color:#0c4a6e
    classDef middle fill:#f3e8ff,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    classDef back fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    classDef exec fill:#ffedd5,stroke:#ea580c,stroke-width:2px,color:#7c2d12
    classDef hold fill:#f1f5f9,stroke:#94a3b8,stroke-width:2px,color:#475569

    subgraph Data["High-Frequency Market Data Feeds"]
        A1[(SSE Tick Data)]:::data
        A2[(NASDAQ Tick Data)]:::data
        A3[(HKEx FX Tick Data)]:::data
    end

    subgraph Front["Front-End"]
        B1["Micro-Market Data Aggregation\nOrder Book · OHLCV Ticks"]:::front
        B2["Calculate Technical Indicators"]:::front
        B3["Fast VMD\nDecompose noise from signal"]:::front
    end

    subgraph Middle["Bayesian Probability Factor, middle end"]
        C1["BiGRU Network\nExtracts sequence features"]:::middle
        C2["Likelihood Estimator\nP(Signal | Profit)"]:::middle
        C3(["Bayesian Updater\nPosterior ∝ Likelihood × Prior"]):::middle
        C4[("Rolling Bias State\nPosterior → Next Prior")]:::middle
        C5{"P(Profit) > 60%?"}:::middle
    end

    subgraph Back["T+0 Strategy Logic"]
        D1["Micro-Transaction Cost Calc\nSpread + Fees + FX"]:::back
        D2{"Net Expected Yield > 0?"}:::back
        D3["Clock Manager\nRoutes to active market"]:::back
    end

    subgraph Exec["Execution & Capital Rotation"]
        E1(["SSE Trade\nHigh-Freq Buy / Sell"]):::exec
        E2(["HKEx FX Settlement\nCNY ↔ CNH ↔ USD"]):::exec
        E3(["NASDAQ Trade\nHigh-Freq Buy / Sell"]):::exec
    end

    Hold(["⏸ Hold / Liquidate"]):::hold

    A1 & A2 & A3 --> B1
    B1 --> B2 --> B3 --> C1

    C1 --> C2 --> C3
    C4 -.->|Prior Probability| C3
    C3 -->|Updates| C4
    C3 -->|Current Probability| C5

    C5 -- Yes › ACTION --> D1
    C5 -- No › INACTION --> Hold

    D1 --> D2
    D2 -- Profitable --> D3
    D2 -- Costs too high --> Hold

    D3 -- China Hours --> E1
    D3 -- U.S. Hours --> E3

    E1 -->|Liquidate & Convert| E2
    E2 -->|Transfer Funds| E3
    E3 -->|Liquidate & Convert| E2
    E2 -->|Transfer Funds| E1
```

## Breakdown

*   **Data Feeds**: Ingests high-frequency tick data from SSE, NASDAQ, and HKEx.
*   **Front-End**: Handles raw data aggregation, technical indicator calculation, and noise-signal decomposition using Fast VMD.
*   **Middle-End**: Employs a BiGRU network and Bayesian probability updating to dynamically predict trade profitability. 
*   **Back-End**: Calculates micro-transaction costs and manages market routing based on the active trading clock.
*   **Execution**: Manages the actual high-frequency buying/selling and handles capital rotation (currency conversion) between the US and Chinese markets via HKEx FX settlement.

---

Currently, it is not viable, as seen from backtests (output), because the model struggles with high latency (0.10-0.80 ms) due to the deep learning addition and regime changes. Alpha will be achieved sooner or later; the Bi-GRU model will be filtered and replicated through linear logic instead.


