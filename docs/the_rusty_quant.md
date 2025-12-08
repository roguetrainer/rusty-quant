# **The Rusty Quant: A Refresher on the Post-2008 Landscape**

**"Everything you knew in 2007 is now a special case of something more complex."**

---
![RQ](../img/rusty-quant.png)
---

If your quantitative memory fades around the time of the Global Financial Crisis (GFC), you are likely remembering a world where "risk-free" meant LIBOR, counterparty risk was a rounding error, and C++ was the only language that mattered.

This document bridges the gap between the "Classic" era and the "Modern" era (2025), explaining why the ground shifted and how the tooling has evolved.

## **1. The Paradigm Shift: When Vanilla Became Exotic**

### **Pre-2008: The "Single Curve" World**

Before Lehman Brothers collapsed, the quantitative world was relatively clean:

* **One Curve to Rule Them All:** We discounted cash flows at LIBOR. We projected forward rates at LIBOR.  
* **Risk-Free Assumption:** Banks (AA/AAA rated) were considered effectively risk-free counterparties for collateralized trades.  
* **Separation of Concerns:** The "Pricing" desk calculated the fair value of a trade. The "Credit" desk worried about defaults separately.

### **Post-2008: The Reality Check**

The crisis revealed that major banks *could* default. This shattered the core assumption of derivatives pricing. Suddenly, a simple **Interest Rate Swap (IRS)** wasn't just a bet on rates; it was a bet on your counterparty surviving to pay you.

* **Dual Curve Pricing:** As detailed in theory_landscape.md, we realized LIBOR contained credit risk. We moved to **OIS Discounting** (Overnight Index Swaps) for collateralized trades because OIS rates (Fed Funds, EONIA) better reflected the risk-free rate of collateral.  
* **Basis Spreads Explosion:** The spread between LIBOR 3M and LIBOR 6M (the "Tenor Basis"), which used to be negligible (fractions of a basis point), blew out to 50+ bps. A 3M vs 6M basis swap became a non-trivial instrument to model.

### **"Vanilla is the New Exotic"**

Because of these basis spreads and the need for rigorous collateral modelling, pricing a "Vanilla" Swap now requires:

1. OIS Discount Curve (for collateral).  
2. LIBOR Projection Curve (for the specific tenor).  
3. Cross-Currency Basis Curves (if FX is involved).  
4. Credit/Funding Adjustments (XVA).

The mathematical machinery required to price a "simple" swap in 2025 is more complex than what was used for some exotic options in 2005.

## **2. The Rise of the XVA Desk**

If the 2008 crisis had a mascot, it would be **CVA (Credit Valuation Adjustment)**.

### **The Alphabet Soup of Valuation Adjustments**

The industry realized that the "Risk-Neutral Price" of a derivative wasn't the *actual* price to the bank. We had to add add-ons (costs) for various risks:

* **CVA (Credit VA):** The cost of hedging your counterparty's potential default. "How much do I lose if *they* go bust?"  
* **DVA (Debit VA):** The benefit of your *own* default risk (controversial but accounting-mandated). "How much do I gain if *I* go bust?"  
* **FVA (Funding VA):** The cost of funding uncollateralized trades. "If I don't get collateral from them, I have to borrow cash from my treasury at a spread."  
* **MVA (Margin VA):** The cost of posting Initial Margin (IM) to a central clearing house (CCP).  
* **KVA (Capital VA):** The cost of holding regulatory capital against the trade.

### **Wrong Way Risk (WWR)**

This is the quant's nightmare scenario: **Exposure and Default Probability are correlated.**

* *Example:* You buy a put option on Oil from an Oil Company. If Oil crashes, your option is worth billions (high exposure). But because Oil crashed, the Oil Company is likely to default.  
* *Impact:* You thought you were hedged, but your hedge evaporates exactly when you need it.  
* *Modeling:* This requires joint simulation of Market Factors (Rates, FX, Commodities) and Credit Factors (Hazard Rates).

For deep learning approaches to these problems, see:

* [**deep-xva**](https://github.com/roguetrainer/deep-xva): Using Neural Networks to approximate the CVA pricing function.  
* [**quantlib-xva-engine**](https://github.com/roguetrainer/quantlib-xva-engine): Traditional Monte Carlo XVA engines.

## **3. Tooling Revolution: Beyond C++**

In 2008, if you wanted speed, you wrote C++. If you wanted prototypes, you wrote Excel/VBA or Matlab. Python was a niche scripting glue.

### **The Rise of Python**

Python is now the *lingua franca* of quantitative finance.

* **Why:** The ecosystem (NumPy, Pandas, Scipy) became robust enough to handle data wrangling better than C++.  
* **Integration:** C++ is still used for the "inner loop" (like QuantLib's core), but it's almost always wrapped in Python.

### **The Contenders: Rust, Go, and Julia**

* **Rust:** The new C++. It offers memory safety without garbage collection. High-frequency trading (HFT) shops are adopting it to prevent segfaults and reduce latency variance. It is becoming popular for crypto-native defi protocols.  
* **Julia:** Designed for scientific computing. It solves the "Two Language Problem" (prototyping in Python, rewriting in C++) by being dynamic *and* compiling to fast machine code. It has a cult following in asset management and risk modeling.  
* **Go:** Used more in the *infrastructure* of finance (matching engines, order gateways) rather than the math/pricing libraries, due to its simplicity and concurrency model.

See [**tensor-scalpel**](https://www.google.com/search?q=https://github.com/roguetrainer/tensor-scalpel) for examples of high-performance tensor operations that might leverage these modern backends.

## **4. Regulation: The Quant as Compliance Officer**

Pre-2008, regulation was often a "box-ticking" exercise for the back office. Post-2008, regulations drive the math itself.

### **Key Regulations**

* **Basel III (Global):** Introduced strictly higher capital requirements. Quants now have to calculate **RWA (Risk Weighted Assets)** for every trade before execution to see if it consumes too much capital (KVA).  
* **Dodd-Frank (US) / EMIR (EU):** Mandated **Central Clearing** for standard swaps. This forced the standardization of OIS discounting and Initial Margin (IM).  
* **FRTB (Fundamental Review of the Trading Book):** The "next big thing." It fundamentally changes how Market Risk capital is calculated, requiring **Expected Shortfall** (ES) instead of Value at Risk (VaR), and demanding rigorous "Risk Factor Eligibility Tests."

### **Impact on Quants**

A quant today spends as much time optimizing **Regulatory Capital** as they do optimizing **P\&L**. A trade that looks profitable on a cash-flow basis might be rejected because its KVA (Capital Cost) makes the Return on Equity (RoE) negative.

## **5. Unification of Pricing and Risk (AAD)**

Historically, "Pricing" (Front Office) and "Risk" (Middle Office) used different systems.

* *Front Office:* Fast, local PDE solvers or trees.  
* *Middle Office:* Massive, overnight Monte Carlo simulations.

The Dream: A unified "Real-Time Risk" system.  
The Enabler: AAD (Adjoint Algorithmic Differentiation).  
As demonstrated in this repo's neural.py and referenced in [**tensor\_networks\_finance**](https://github.com/roguetrainer/tensor_networks_finance), AAD allows us to calculate the risk (Greeks) of a portfolio *instantly* as a byproduct of pricing it. This allows Middle Office risk numbers to be available to traders in real-time, closing the feedback loop.

## **Summary of Changes (2008-2025)**

| Concept | 2008 View | 2025 View |
| :---- | :---- | :---- |
| **Rates** | LIBOR is king. | LIBOR is dead. SOFR/SONIA reign. |
| **Credit** | Separate desk. | Baked into price (CVA). |
| **Funding** | Ignored / Treasury's problem. | Baked into price (FVA). |
| **Compute** | CPU Grids (C++). | GPUs, AAD, and Cloud (Python/Rust). |
| **Model** | Black-Scholes / Heston. | Rough Volatility, Neural SDEs. |
| **Constraint** | CPU Cycles. | Regulatory Capital. |

