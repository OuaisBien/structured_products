# Pricing of Phoenix Autocallable Note With Heston Stochastic Volatility

This repository implements a robust Monte Carlo pricing engine for a **Phoenix Autocallable Note** linked to the S&P 500. The project focuses on handling high-volatility regimes and "Feller-violated" parameters using the **Andersen Quadratic Exponential (QE)** discretization scheme and **Antithetic Variates** for variance reduction.

## Project Overview

The Phoenix Autocallable is a path-dependent structured product offering enhanced yield (coupons) in exchange for downside "Knock-In" put risk. Pricing this instrument accurately requires a model that captures the **volatility skew** and **tail risk** of the equity market.

**Key Technical Features:**
* **Model:** Heston Stochastic Volatility Model (1993).
* **Simulation:** Andersen Quadratic Exponential (QE) Scheme (robust to Feller violations).
* **Variance Reduction:** Antithetic Variates (AV).
* **Calibration:** Calibrated to S&P 500 implied volatility surfaces (identifying steep negative skew).
* **Risk Analysis:** Full calculation of Greeks (Delta, Vega, Vanna, Vomma) including convexity profiles.

---

## Technical Implementation

### 1. Heston Model & Feller Violation
The standard Euler-Maruyama discretization fails for the Heston model when the variance process $v_t$ touches zero. Our calibration to the S&P 500 revealed that $2\kappa\theta - \xi^2$ is close to 0, meaning the variance process is highly volatile and frequently approaches zero.

To solve this, we implemented the **Andersen QE Scheme**, which models the non-central chi-square distribution of the variance using two regimes:
* **High Volatility Regime ($\Psi \le 1.5$):** Uses a quadratic transformation of a Gaussian variable.
* **Low Volatility Regime ($\Psi > 1.5$):** Uses an exponential distribution with a discrete mass at zero to prevent "sticky zero" bias.

### 2. Variance Reduction: Antithetic Variates
To improve convergence speed and stability, particularly for second-order Greeks (Vomma), we employed **Antithetic Variates**:
* For every simulated path driven by Brownian motions $(Z_S, Z_v)$, we simultaneously simulate a "mirror" path using $(-Z_S, -Z_v)$.
* This technique reduced the standard error of the Monte Carlo estimator by approximately **50%** for smaller $n$ compared to a naive simulation of the same computational cost.

### 3. Log-Euler Stock Discretization
The stock process utilizes the **Broadie-Kaya** exact simulation principles (via Log-Euler) to preserve the critical negative correlation ($\rho \approx -0.54$) between spot and volatility:

$$\ln S_{t+\Delta t} = \ln S_t + K_0 + K_1 \int_t^{t+\Delta t} v_s ds + K_2 (v_{t+\Delta t} - v_t) + \sqrt{1-\rho^2} \int_t^{t+\Delta t} \sqrt{v_s} dW^\perp$$

The term $K_2(v_{t+\Delta t} - v_t)$ explicitly forces the stock price to react to variance spikes, correctly pricing the deep OTM downside protection.

---

## 📊 Key Results & Risk Analysis

### Valuation
* **Fair Price:** ~98.99% (at 8.00% p.a. coupon).
* **Implied Margin:** ~1.00% (Bank Profit).
* **Conclusion:** The high coupon is necessary to fund the expensive downside protection in a skew-heavy market.

### Greek Sensitivities
We generated "Spider Plots" to visualize the dynamic risk profile:

| Greek | Value ($S_0=1000$) | Interpretation |
| :--- | :--- | :--- |
| **Delta** | `+0.09` | Low sensitivity at inception due to binary payout nature. |
| **Vega** | `-2.01` | **Short Volatility:** Increasing vol hurts the value of the short put. |
| **Vanna** | `+1.08` | **Right-Way Risk:** As Spot rises, Vega risk disappears (Autocall). |
| **Vomma** | `-0.04` | **Negative Convexity:** Risk
