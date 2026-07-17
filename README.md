# Systemic-Risk-Contagion-Modeling-using-DebtRank-Algorithm-on-Interbank-Exposure-Networks
# Systemic Risk Contagion Modeling using DebtRank on US Bank Interbank Networks

## Overview
This project models financial contagion risk across the 30 largest US banks using the **DebtRank algorithm** (Battiston et al., 2012), the same methodology used by the European Central Bank to assess systemic risk. It simulates how a shock to one bank propagates losses through the banking system via interbank lending exposure.

## Motivation
Traditional credit risk models (e.g., my earlier American Express Default Prediction project) assess *individual* borrower risk. This project addresses a different, higher-level question: **how does the failure of one financial institution cascade through the entire banking system?** This is the core question behind bank regulation (Basel III), stress testing, and "Too Big to Fail" / "Too Interconnected to Fail" designations.

## Data Source
- **FDIC BankFind Suite API** (`banks.data.fdic.gov`) — public, no authentication required
- Real Q4 2023 balance sheet data (Total Assets, Equity, Liabilities, Deposits) for the 30 largest FDIC-insured US banks by assets, including JPMorgan Chase, Bank of America, Wells Fargo, Citibank, Goldman Sachs, and others

## Methodology

### 1. Interbank Exposure Reconstruction (Maximum Entropy Method)
Bilateral interbank lending data is confidential and not publicly available — this is a well-known constraint in systemic risk research. Following the standard approach used in central bank literature (Upper & Worm, 2004), I:
- Estimated each bank's interbank assets/liabilities as 15% of total assets/liabilities (a documented industry approximation)
- Reconstructed a plausible 30×30 bilateral exposure matrix using the **RAS/Maximum Entropy algorithm**, which finds the most "unbiased" distribution of exposures consistent with each bank's known totals

### 2. DebtRank Simulation
Implemented the DebtRank algorithm to simulate cascading distress:
- A shock is applied to one bank (e.g., 50% equity loss)
- Distress propagates to banks with exposure to the shocked bank, weighted by exposure-to-equity ratio
- Propagation continues iteratively until no new distress occurs
- Output: a systemic impact score (0–1) representing total economic value lost across the system

### 3. Sensitivity Analysis
Repeated the simulation shocking each of the 30 banks individually to rank systemic importance.

## Key Findings
- A 50% shock to JPMorgan Chase produces a **DebtRank systemic impact score of 0.309** — roughly 31% of total system equity is affected
- **Charles Schwab Bank showed the highest final distress level (0.68)** among all 30 banks — higher than even JPMorgan itself post-shock. This illustrates a core insight of DebtRank: systemic fragility depends on exposure *relative to equity*, not absolute exposure size. A bank with a smaller capital buffer can be more vulnerable to contagion than its balance sheet size suggests.
- Systemic impact scaled with bank size in this dataset, though this is partly an artifact of using a uniform interbank exposure ratio across all banks (see Limitations)

## Limitations & Honest Caveats
- Interbank exposures are **estimated, not observed** — real bilateral lending data is confidential (this is true of virtually all public systemic risk research, not unique to this project)
- A uniform 15% interbank ratio was applied across all banks; in reality this ratio varies by institution type and would produce more nuanced "too interconnected to fail" outliers
- Single-shock scenario; does not model liquidity spirals, fire-sale effects, or market-wide feedback loops

## Tech Stack
`Python` · `pandas` · `numpy` · `networkx` · `matplotlib` · `requests` (FDIC API)

## How to Run
```bash
pip install -r requirements.txt
jupyter notebook debtrank_systemic_risk.ipynb
```

## References
- Battiston, S., et al. (2012). "DebtRank: Too Central to Fail? Financial Networks, the FED and Systemic Risk." *Scientific Reports*.
- Upper, C., & Worm, A. (2004). "Estimating Bilateral Exposures in the German Interbank Market." *European Economic Review*.
