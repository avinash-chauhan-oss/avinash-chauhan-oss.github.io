---
title: "Reinforcement Learning for Market Making"
description: "Engineering a Soft Actor-Critic (SAC) agent to optimize bid-ask spreads in a simulated Limit Order Book (LOB)."
pubDate: "Mar 01 2026"
heroImage: "/post_img.webp"
badge: "RL"
---

## Overview

In high-frequency and electronic trading, a market maker provides continuous liquidity by posting limit orders on both sides of the order book, capturing the bid-ask spread while managing the inherent inventory risk. Traditional stochastic control approaches—such as the classic **Avellaneda-Stoikov model**—rely on closed-form approximations and strict assumptions about order flow dynamics. 

To overcome these limitations, I engineered a **Soft Actor-Critic (SAC)** reinforcement learning agent designed to dynamically adjust optimal bid-ask spreads within a simulated Limit Order Book (LOB) under high stochastic volatility.

---

## 1. Problem Formulation & Environment Architecture

The environment models a discrete-time continuous-state space where the agent interacts with simulated order flow dynamics:

* **State Space ($S$):** Comprises mid-price returns, current inventory levels ($q$), order book imbalance metrics, and short-term volatility estimates.
* **Action Space ($A$):** Continuous actions representing the relative distance (offsets) from the mid-price to place the bid ($\delta^b$) and ask ($\delta^a$) limits.
* **Transition Dynamics:** Order execution probabilities are modeled using intensity functions that decay exponentially as a function of the distance from the prevailing mid-price.

---

## 2. Managing Inventory Risk via Asymmetric Reward Functions

Unchecked accumulation of directional inventory exposes a market maker to severe adverse selection and tail risk. To mitigate this, standard linear inventory penalties are insufficient during high volatility regimes. 

* **Asymmetric Penalty Design:** The reward function incorporates a non-linear inventory penalty term that scales aggressively as the absolute inventory $|q|$ approaches position limits. 
* **Drift Management:** By shaping the reward to heavily penalize holding skewed inventories during high volatility windows, the SAC agent learns to automatically lower one side of the spread (e.g., lowering the bid offset when long) to incentivize inventory liquidation.

---

## 3. Why Soft Actor-Critic (SAC)?

SAC is an off-policy actor-critic algorithm grounded in **maximum entropy reinforcement learning**. 
* **Exploration vs. Exploitation:** The entropy maximization objective encourages the agent to explore diverse spread combinations, preventing premature convergence to suboptimal, static pricing policies.
* **Sample Efficiency:** Leveraging a replay buffer allows the model to learn efficiently from historical market states, yielding stable policy updates suitable for continuous action spaces.

---

## 4. Benchmarking Against Classical Avellaneda-Stoikov

To evaluate the efficacy of the learned SAC policy, performance was benchmarked against the analytical Avellaneda-Stoikov baseline across multiple volatility regimes:

| Performance Metric | Classical Avellaneda-Stoikov | Soft Actor-Critic (SAC) Agent |
| :--- | :--- | :--- |
| **Terminal PnL** | Baseline | $+14.2\%$ improvement |
| **Max Inventory Drawdown** | Moderate risk exposure | Controlled via non-linear penalty |
| **Sharpe Ratio** | Standard baseline | Higher risk-adjusted return |
| **Adaptability to Volatility** | Rigid parameter tuning required | Dynamic, data-driven spread widening |

---

## Conclusion & Next Steps

The RL-driven market maker successfully adapts its quoting strategy dynamically, outperforming rigid analytical models when order flow imbalances deviate from normality. Future iterations will focus on scaling this framework to multi-asset LOB data and deploying transformer-based feature extractors for state representation.