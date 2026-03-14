# Midnight LMSR Protocol Specification

This document defines the technical protocol for the Logarithmic Market Scoring Rule (LMSR) implementation on the Midnight blockchain. It outlines the state variables, transaction logic, and mathematical invariants that govern the market.

## 1. Overview

The protocol implements a two-outcome automated market maker (AMM) based on the Logarithmic Market Scoring Rule. The contract acts as the counterparty to all trades, ensuring liquidity and continuous pricing.

## 2. Ledger State

The protocol maintains several global state variables on the Midnight ledger:

| Variable | Type | Description |
| :--- | :--- | :--- |
| `b` | `Uint<64>` | The liquidity parameter. Higher `b` means lower price sensitivity to trades. |
| `numOutcomes` | `Uint<64>` | Fixed at `2` for the current version. |
| `shares` | `Map<Uint<64>, Uint<64>>` | Mapping of outcome ID to the total quantity of shares currently held by all market participants. |
| `totalCostPaid` | `Uint<64>` | Accumulator of all DUST/tokens paid into the contract (total volume). |
| `owner` | `Bytes<32>` | The public identity key of the market creator. |

## 3. Mathematical Invariants

### 3.1 Cost Function
The core invariant is the cost $C(q)$, which represents the total amount of money that must be in the system given the current share distribution $q = \{q_0, q_1, \dots, q_n\}$.

$$C(q) = B \cdot \ln \left( \sum_{i=0}^{n-1} e^{q_i / B} \right)$$

### 3.2 Instantaneous Price
The price $P_i$ for a single share of outcome $i$ is the partial derivative of the cost function with respect to $q_i$:

$$P_i(q) = \frac{\partial C}{\partial q_i} = \frac{e^{q_i / B}}{\sum_{j=0}^{n-1} e^{q_j / B}}$$

**Properties:**
- $0 \le P_i \le 1$
- $\sum P_i = 1$

### 3.3 Fairness and Monotonicity
- **Path Independence**: The cost to move from state $q_{start}$ to $q_{end}$ is always $C(q_{end}) - C(q_{start})$, regardless of the sequence of intermediate trades.
- **Monotonicity**: Since $P_i > 0$ for all $q_i$, the cost function is strictly increasing. Any purchase of shares ($\Delta q > 0$) results in a positive payment from the trader to the contract.

## 4. Transaction Protocol

### 4.1 Market Initialization (`constructor`)
- **Action**: Set initial liquidity `b` and `owner`.
- **Pre-condition**: `initialB > 0`.
- **Post-condition**: `shares[0] = 0`, `shares[1] = 0`, `totalCostPaid = 0`.

### 4.2 Buying Shares (`buy`)
The `buy` operation is the primary state-changing transaction.

**Inputs:**
- `outcome_id`: The ID of the outcome (0 or 1).
- `delta_q`: The quantity of shares to purchase.

**Logic Flow:**
1. **Assert** `outcome_id < 2`.
2. **Assert** `delta_q > 0`.
3. **Fetch** current quantities $\{q_0, q_1\}$ from the ledger.
4. **Calculate** $Cost_{old} = C(q_0, q_1)$ via witness.
5. **Update** target quantity: $q_{target} = q_{outcome\_id} + \Delta q$.
6. **Calculate** $Cost_{new} = C(q_{new})$ via witness.
7. **Verify** $Cost_{new} \ge Cost_{old}$.
8. **Deduct** $Payment = Cost_{new} - Cost_{old}$ from user (abstracted in current version as ledger update to `totalCostPaid`).
9. **Update** `shares` map with new quantities.

## 5. ZK-Witness Implementation

To maintain efficiency in the Midnight ZK-circuit, complex transcendental functions ($\ln$, $e^x$) are computed off-chain via **Witnesses**.

1. The client computes the exact cost and price using high-precision math.
2. The results are passed as witnesses to the circuit.
3. The circuit "discloses" these values to the ledger after verifying basic assertions (like price bounds and non-negativity).

**Current Witnesses:**
- `calculateLmsrCost`: Returns $C(q)$ scaled by $10^6$.
- `calculateLmsrPrice`: Returns $P_i(q)$ scaled by $10^6$.
- `getCurrentShares`: Provides the current ledger state to the circuit.

## 6. Numerical Precision

All values in the protocol are handled as **Fixed-Point Integers** with a scaling factor of **$10^6$**.

- **Example**: A price of `0.55` is represented as `550,000`.
- **Risk Mitigation**: The `totalCostPaid` and `shares` are `Uint<64>`. At a $10^6$ scale, this allows for volumes up to $\approx 18 \times 10^{12}$ units before overflow, which is sufficient for high-volume prediction markets while maintaining micro-precision.

## 7. Privacy Details

Midnight's ZK-architecture provides several privacy properties for this protocol:
- **Trade Privacy**: While the global share counts $q$ are currently disclosed to calculate the market-making cost, individual user balances could be moved to private state in future iterations.
- **Volume Obfuscation**: Total DUST paid is observable on the public ledger, but individual trade timing can be obfuscated via private transaction submission.
