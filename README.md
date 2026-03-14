# LMSR Prediction Market Protocol

A protocol specification for a **Logarithmic Market Scoring Rule (LMSR)** based prediction market designed for deployment on privacy-preserving blockchains such as Midnight.

This repository contains the **formal protocol description, economic model, and research documentation** for an LMSR-based prediction market system.

Unlike constant-product AMMs, LMSR markets are designed specifically for **information aggregation**, allowing participants to trade on the probability of future events.

## Overview

Prediction markets allow participants to buy and sell shares representing possible outcomes of future events.  
The price of each outcome represents the **market's collective probability estimate**.

This protocol implements a **Logarithmic Market Scoring Rule (LMSR)** automated market maker.

Key properties:

- Continuous liquidity
- Bounded loss for the market maker
- Convex pricing mechanism
- Probabilistic price interpretation

## Mathematical Model

The LMSR cost function:


$$
C(q) = b \cdot \ln \left( \sum_i e^{q_i/b} \right)
$$


Where:

- \(q_i\) = shares of outcome \(i\)
- \(b\) = liquidity parameter

The price of outcome \(i\):

$$
P_i = \frac{e^{q_i/b}}{\sum_j e^{q_j/b}}
$$

Prices always sum to **1.0**, representing a probability distribution.

## Repository Structure

```
README.md
ARCHITECTURE.md
PROTOCOL.md
SPEC.md
ECONOMICS.md
MARKET_MECHANISM.md
PRICE_DYNAMICS.md
SIMULATION.md
EXAMPLES.md
SECURITY.md
ROADMAP.md
LICENSE
```

## Applications

Potential applications of LMSR markets include:

- decentralized oracles
- DAO governance forecasting
- insurance and risk markets
- futarchy-based governance

## License

This repository is licensed under the Apache 2.0 License.
