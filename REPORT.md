## 1. Problem Statement and Quantitative Formulation

##### 1.1 Objective of the Exercise
The aim of this exercise isn't to construct a fully realistic private credit or FX trading system. It serves to demonstrate how to translate a loosely defined business requirement into quantitative concepts. This includes:

- formulating a concretely defined problem,
- suitability of model choices,
- ability to formulate a solution to the problem.

##### 1.2 Economic Framing
We'll be assuming the position of a private credit fund manager, with a USD-denominated fund, whose assets are EUR-denominated. And thus exposed to FX risk. All other risks such as Credit Risk, Interest Rate Risk aren't considered.

 - The investment generate EUR deterministic cashflows as follows:

| Cashflow Date | Cashflow (€ M) |
|---------------|----------------|
| 2025-10-01    | -10            |
| 2026-10-01    | 1              |
| 2027-10-01    | 1              |
| 2029-10-01    | 1              |
| 2030-10-01    | 11             |

 - The fund reports in USD
 - EURUSD FX is the only source of uncertainty affecting the USD value of the investment

### 1.3 Mathematical Framing and Problem Formulation
We define:

 - $\\{CF_i^{EUR}\\}_{i=1}^N$ as the set of deterministic EUR cashflows,
 - and $\\{T_i\\}_{i=1}^N$ their corresponding payment dates,
 - $S_t$ denote the EURUSD spot rate (USD per unit of EUR),
 - $t_0$ the analysis (valuation) date.

The unhedged USD cashflows are:

$$CF_i^{USD} = CF_i^{EUR} \cdot S_{T_i}$$

The FX spot rate is the only stochastic driver in the problem. And all other quantitites like cashflows, interest rates, option volatilites are treated deterministically at $t_0$.

The problem is to:
 - 1. Evaluate the performance of the USD investment and quantify the distribution introduced by FX uncertainty.
 - 2. Design hedging strategies using financial derivative products that mitigate the risk by modifying the distribution in controlled way.
 - 3. Identify the risk profiles associated with each hedging strategies and compare them in terms of:
    - downside risk reduction,
    - cost,
    - impact on upside potential.

## 2. Performance Metrics


### 2.1 Metric Selection
The investment profile portraid the traits of a bullet-loan where an upforont capital outlflow is followed by periodic interest payments and concluded with a final principal + interest.
These are commonly observed in private credit investments like direct lending or term loans. After some background reading, the following performance metrics were identified as appropriate trackers:

| Metric          | Question Answered                          |
|-----------------|--------------------------------------------|
| **NPV**         | Is the investment value-creating in USD terms? |
| **IRR**         | What is the annualised return profile?     |
| **MOIC**        | How much capital is returned per unit invested? |
| **Terminal Value** | What is the final USD outcome?             |

### 2.2 Metric Definitions

##### <u>__Net Present Value (NPV)__</u>

Sums the discounted USD cashflows to see if the exceed the required teturn rate, $r_m$, which is around $7\%$ in the Private Credit industry. PROVIDE REFERENCE.

$$NPV = \sum \frac{CF_i^{USD}}{(1 + r_m)^T_i}$$

- Positive NPV: Investment exceeds required return.
- Negative NPV: Investment underperforms the funds hurdle rate. 
- Sensitive to both IRR and timing of cashflows.

##### <u>__Internal Rate of Return (IRR)__</u>

The IRR finds what the $r_m$ is such that the NPV is equal to zero. IRR solves:

$$\sum \frac{CF_i^{USD}}{(1 + IRR)^T_i} = 0$$

- A high IRR (e.g. $>10\%$): Discount rate >> hurdle rate (NPV>0 at hurdle); strong outperformance.
- Zero IRR: Breakeven (NPV=0); not worth the investment.
- Negative IRR: Discount rate < 0 (NPV < 0 a $0\%$); capital destruction.
- Can become negative under adverse FX scenarios.
- Nonlinear. Small cashflow changes cause disporportional IRR swings. (ref. 2)
- Senstivity to early FX shocks.

##### <u>__Multiple of Invested Capital (MOIC)__</u>

Total USD cash inflows divided by absolute inital USD cash outflow. Not discounted.

$$MOIC = \frac{\sum CF_i^{USD}}{CF_0^{USD}}$$

- Intuitive capital efficieny metric.
- MOIC < 1 implies capital loss.
- FX depreciation directly reduces numerator.
- Unaffected by cashflow scheduel.

##### <u>__Terminal Value__</u>

 - Discloses end-state outcomes.
 - Usefule for tail analysis.

### 2.3 FX Impact

**EUR appreciates against USD, $S_{T_i}$ Increases**

 - USD value of future cash inflows increases.
 - NPV increases
 - IRR increases (larger effect when $S_{T_i}$ appreciation happens sooner)
 - MOIC increase proportionally to terminal USD cashflows


**EUR depreciates against USD, $S_{T_i}$ Decreases**

 - USD value of future cash inflows deacreases.
 - NPV may become negative.
 - IRR falls below zero if discounted cash inflows fail to recuperate initial cash outflow.

## 3. Modelling Framework


### 3.1 FX Spot Rate Model

##### <u>__Model Specification__</u>

The EURUSD Spot rate $S_{t}$, quoted as USD per EUR, is modelled as a Geometric Brownian Motion (GBM)_

$$dS_t = \sigma S_t dW_t$$

where:
 - $\sigma$ is the constant volatility,
 - $W_t$ a standard Brownian motion.

The exact solution to the SDE is:

$$S_t = S_0  \mathrm{exp} \left( - \frac{1}{2} \sigma^{2} t + \sigma W_{t} \right)$$

##### <u>__Choice of Drift__</u>

The drift term is set to:

$$ \mu = 0$$

The choice reflects a flat expected FX path scenario in our simulations.

**Rationale**

The scope and purpose of the exercise were considered. Introducing a non-zero drift would imbed macro views.
This would influence investment decisions from the onset which is conceptually seperate from the task at hand. The aim is to solve a hedging problem.
Setting $\mu$ to zero ensures that all risk stems purely from FX volatility. Like this, strategy differences arise from the hedging structures and investment performances are not inflated by macro trends.

**Why not Risk Neutral?**

Risk neutral dynamics are necessary for derivative pricing, but not for evaluating the real-world investments performances we are measuring. Under our framework:

 - FX Forwards and FX Options are priced analytically under no-arbitrage conditions using interest rate differentials.
 - Investment performance measures are evaluated under a real-world measure.
 - FX spot paths are simulated independenetly of pricing measure assumptions.

The seperation avoids assuming the investment as a risk-free asset. 

##### <u>__Volatility Estimation__</u>

Volatility is estimated from historical daily log returns of EURUSD:

 - Business-day sampling is used.
 - Weekend effects are implicitly embeded through friday to monday returns.
 - Each business day return representes a full calalander day exposure to FX risk.
 - Mid-rate log returns are used.

This ensured that the calibrated volatility is consistend with the simulation time grid. Time is measured in calander years using (ACT/365) day-count convention.

### 3.2 FX Forward Pricing Model

##### <u>__Pricing Relationship__</u>

FX forward rates are computed using covered interest parity (CIP):

$$ F \left(T\right) = S_{0}   \mathrm{exp} \left( \left( r_{USD} - r_{EUR} \right) T \right) $$

where:

 - $S_0$ is the EURUSD spot rate observed at the valuation date,
 - $r_{\text{USD}}$ and $r_{\text{EUR}}$ are continuously compounded USD and EUR interest rates,
 - $T$ is the year fraction between the valuation date and the cashflow date (ACT/365).

This relationship is applied independently for each loan cashflow maturity.

##### <u>__Interest Rate Inputs and Proxies__</u>

USD and EUR interest rates are assumed deterministic and flat across maturities.
Proxy rates observed on the valuatation date, August 1st 2025, are used:

 - USD rate: $\mathrm{SOFR} = 4.34 \%$ (ref 5)
 - EUR rate: $\mathrm{€SRT} = 1.927 \%$ (ref 4)

These ares are sourced from Federal Reserve and ECB publications, respectively.

In a fully market-consistent implementation, forward rates would be computed using maturity-specific zero-coupon discount factors extracted from USD and EUR OIS curves. In the present analysis, flat rates are used:

 - isolate FX spot risk,
 - avoid curve-construction.

##### <u>__Interpretation__</u>

The forward rate embeds:

 - FX carry,
 - funding differentials betwen USD and EUR,
 - the oportunity cost of holding one currency over the other.

 Using CIP ensures:

  - no-arbitrage consistency,
  - determinsitic hedge cashflows once established.

Cross-currency basis effects are ignored and covered interest parity is assumed to hold.

### 3.3 FX Option Pricing Model

##### <u>__Model Choice__</u>

FX Options are priced using Garman-Kohlhagen model, which extends the Black Scholes model to foreign exchange markets.

The price of a European FX put option is given by:

$$ P_0 = e^{-r_{\text{USD}} T} \left( K \, \Phi(-d_2) - F(T)\, \Phi(-d_1) \right) $$

where:

$$
d_1 = \frac{\ln\!\left(\frac{F(T)}{K}\right) + \tfrac{1}{2}\sigma^2 T}{\sigma \sqrt{T}},
\qquad
d_2 = d_1 - \sigma \sqrt{T}.
$$

and:

 - $F\left(T\right)$ is the forward FX rate,
 - $K$ is the strike,
 - $\sigma$ is the implied volatility,
 - $\Phi\left(\cdot\right)$ is the standard normal CDF.

##### <u>__Volatility Input__</u>

ATM implied volatilities are used as model inputs:

 - 1-year and 5-year ATM implied vols are observed at the valuation date,
 - volatilities for intermediate maturities are obtained via linear interpolation,
 - smile and skew effects are ignored.

This choice reflects:

 - market observability,
 - alignment with the objective of downside protection rather than volatility trading.

##### <u>__Premium Treatment__</u>

Option premiums are computed at the valuation date and treated as upfront USD cash outflows.

In implementation:

 - a new cashflow date equal to the analysis date is introduced,
 - the premium is recorded as a negative USD cashflow at that date,
 - premiums are not discounted again, avoiding double counting.

This ensures correct cashflow timing and economic consistency.

## 4. Risk Metrics

Quantifying the FX risk is done via distributional risk measures computed across the Monte Carlo paths.
For each performance metric:

 - Mean, standard devaiton
 - Percentiles ($5\%$, $50\%$, $95\%$)
 - Tail probabilites:
    - $\mathrm{P}\left(\mathrm{NPV}<0\right)$
    - $\mathrm{P}\left(\mathrm{IRR}<0\right)$
    - $\mathrm{P}\left(\mathrm{MOIC}<1\right)$

Additionally:
 - Value-at-Risk (VaR)
 - Expected Shortfall (ES)

These metrics, implemented in the `metrics.risk.py` module, provide the link between the FX uncertainty and the investment outcomes. And allow for the direct comparison across strategies.

## 5. Hedging Strategies

Given the constraint and the nature of the exercise, focuse was only given to the canonical hedging strategies:
 - FX Forwards 
 - FX Options

In the first strategy we maximise certainty while in the second we mitigate downside risk while retaing upside potential.

The code was written in a modular manner that allows for other strategies to be explored in the future. 

##### 5.1 Forward Hedge Strategy

##### <u>__Design__</u>

The forward hedge is implemented as:
 - a cashflow-by-cashflow hedging,
 - 100% notional coverage of positive EUR cashflows,
 - a static hedge established at the valuation date.

For each EUR inflow at time $T_i$ a forward contract is entered with the notional equal to the expected EUR cashflow.

##### <u>__Pathwise Cashflow Construction__</u>

Unhedged USD cashflows are:

$$
CF_i^{\text{USD}} = CF_i^{\text{EUR}} \cdot S_{T_i}
$$

The forward hedge payoff is:

$$
\mathrm{Hedge}_i = CF_i^{\text{EUR}} \cdot \left( F(T_i) - S_{T_i} \right)
$$

Resulting inthe fully hedged USD cashflows:

$$
\begin{aligned}
CF_i^{\text{USD, hedged}}
&= CF_i^{\text{EUR}} \cdot S_{T_i} + CF_i^{\text{EUR}} \cdot \left( F(T_i) - S_{T_i} \right) \\
&= CF_i^{\text{EUR}} \cdot F(T_i) \\
\end{aligned}
$$

FX uncertainty is therefore eliminated pathwise for hedged cashflows.

##### <u>__Risk Profile__</u>
 - FX volatility is almoset entirely removed.
 - Performance metric distributions become tightle concentrated.
 - Downside risk is minimised. 

##### <u>__Pros and Cons__</u>

**Pros**
- Simple and transpatent
- No upfront cost
- Predictable Outcomes

**Cons**
- No upside participation
- Locks in FX opportunity cost

## 5.2 Option Hedge Strategy

##### <u>__Design__</u>

The option hedge consists of purchasing:
- ATMF EUR Put (equivalent to USD Call)
- one option per EUR cashflow maturity.
- strikes set equaly to forward rate,
- premiums paid upfront at the valuation date.

##### <u>__Payoff Structure__</u>

The option payoff per cashflow is:

$$ \text{Payoff}_i = CF_i^{\text{EUR}} \cdot \max   \left( K_i - S_{T_i},  0 \right) $$

where $K_i = F(T_i)$.

The USD cashdlow per path then becomes:

$$ CF_i^{\text{USD, hedged}} = CF_i^{\text{EUR}} \cdot S_{T_i} + \text{Payoff}_i$$

with the option premium recorded as an intial USD outflow.

$$
\begin{aligned}
CF_i^{\text{USD, hedged}}
&= CF_i^{\text{EUR}} \cdot S_{T_i} + CF_i^{\text{EUR}} \cdot \max   \left( F(T_i) - S_{T_i},  0 \right) \\
&= CF_i^{\text{EUR}} \cdot \max  \left( S_{T_i},   F(T_i) \right).
\end{aligned}
$$

##### <u>__Risk Profile__</u>
 - Downside FX risk is capped.
 - Upside FX appreciation is presereved.
 - Distribution exhibit truncated left tails adn retained right tails.

##### <u>__Pros and Cons__</u>

**Pros**
- Downside protection
- Retained upside exposure

**Cons**
 - Premium cost
 - Lower Expected return than forwards

## 6. Parameter Choices and Risk Alignment


The two hedging strategies considered in this analysis are designed to align with distinc investor objectives and risk profiles. Rather than optimising a single criterion, the framework highlights the trad-offs between certainty, cost and convexity.

**Forward hedging** aligns with mandates prioritising capital certainty and cashflow predictability. By fixing the exchange rates in the future, you are eliminating any FX-driven variability in USD outcomes.This results in more concentrated distributions of IRR, NPV and MOIC with minimal downside risk. Such strategy is consistent with investors who value stability.

**Option-based hedging**, by contrast, alighs with downside controlled but upside seeking mandates. Buying ATMF EUR put options establishes a florr on the USD value of future cashflows without abdicating participation in favourable FX moves. This convex payoff structure truncates the left-tail outcomes at the cost of an upfront premium. The strategy is consistent with investors willing to pay for insurance against adverse FX scenarios but want to retain optionality, placing him or her somewhere between the unhedged investor and the forward hedging investor.

The appropriate strategy, therefore, depends on the investor preferences regarding:
 - Drawdown tolerance
 - Cost sensitivity (from Option premiums)
 - Demand for convexity

## 7. Assumptions, Simplifications, and Limitations


The analaysis relies on a number of explicit assumptions that were deliberate design choises.

##### <u>__Hedging Structure__</u>

 - **100% hedging of EUR inflows**

    Each positive EUR cashflow is hedged in full. Partial hedging or dynamic hedge ratios are not considered. Hedging ratio
    can be adjusted through the global varaible parameter `global_variables.GlobalVariables.hedging_ratio`.

- **Static cashflow-by-cashflow hedging**

    All hedges are established at the valuation date and held to maturity. There are no rebalancing, rolling or dynamic hedging perfomed.

- **Initial EUR cash outflow not hedged**

    The inital outflow of, what appears a loan principal, isn't hedged. The focus of analysis is on protecting future EUR inflows rather that funding FX risk at inception.

##### <u>__Costs and Market Frictions__</u>

- **Transaction costs**

    No transaction costs are modelled for the forwards. For options, the premium represents the sole explicit cost.

- **Bid-ask convention**

    Market-consistent pricing conventions were applied. Namely, FX spot path simulated mid price and was calibrated as such. Option premiums, on the other hand, were priced using ask implied volatilities, reflecting the cost of buying protection.

- **Bid-ask spread effect**

    No bid-ask spreads on forwards and options are modelled explicitly beyonf the use of ask volatilities for options. In practice, wider spreads would further increas transction costs impacting performance.

##### <u>__Interest Rates and FX Pricing__</u>

- **Deterministic interest rates**
    
    USD and EUR interest rates are assumed determinisitc and observed at the valuation date. They are solely used for FX forward and option pricing. 

- **Flat term structures**

    Flat proxy rates are used across diferent maturities (Section 3.2)

- **No cross-currency basis**

    Effects stemming from CCY basis were ignored. Theswe can have material impact in practice. Instead CIP is assumed (Section 3.2)

##### <u>__Option Specification__</u>

- **ATMF option strikes**

    Option strikes are set to the Forward rate at each maturity.

- **No volatility smile or skew modelling**

    Although market data for $25\Delta$ risk reversals and butterflies are provided, these are not used due to time constraints.

##### <u>__Stochastic Driver__</u>

- **FX spot as the sole stochastic driver**

    Credit risk, liquidity risk, interest rate risk, coutnerparty credit risk are not modelled

##### <u>__Interpretation__</u>

Put together, these assumptions imply a framework that should be interpreted as a clean and isolated FX risk exercise.
Far from a fully comprehensive investment model. The goal, reiterated here, is to:

- demonstrate how FX risk propagates through investment performance metrics
- compare hedging strategies under controlled conditions
- and provide a base that then can be extended to other strategies, model use and risk considerations.
