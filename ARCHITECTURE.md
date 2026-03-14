# System Architecture

The Midnight LMSR implementation is designed for decentralization, data integrity, and high-performance simulation.

## Architecture Overview

The project consists of three primary layers: the **On-Chain Logic**, the **Client Integration**, and the **Research/Simulation Layer**.

```text
       ┌──────────────────────────┐
       │      User (CLI)          │
       └──────────┬───────────────┘
                  │ (1) Interact
                  ▼
       ┌──────────────────────────┐
       │       Pricing Engine     │ (Internal Math Helpers)
       │                          │
       └──────────┬───────────────┘
                  │ (2) Calculate P/C
                  ▼
       ┌──────────────────────────┐      ┌──────────────────────────┐
       │   Midnight Smart         │      │     Simulation Engine    │
       │     Contract             │◄────►│                          │
       │                          │                                 │
       └──────────┬───────────────┘      └──────────────────────────┘
                  │ (3) Ledger State
                  ▼
       ┌──────────────────────────┐
       │    Midnight Blockchain   │
       └──────────────────────────┘
```

## Layer Responsibilities

### 1. Smart Contract
The "Single Source of Truth."
- **State Management**: Persists the liquidity parameter ($b$), outcome shares ($q$), and global volume.
- **Validation**: Ensures that every purchase follows the LMSR cost curve ($payment = C_{new} - C_{old}$).
- **Witness Verification**: Uses off-chain witnesses to verify complex mathematical results that would be too expensive to compute directly in a constrained circuit.

### 2. CLI Tooling
The "Interface Layer."
- **Wallet Connection**: Manages Midnight keys and DUST balances.
- **Transaction Construction**: Builds and submits transactions to the Midnight preprod network.
- **Local Math**: Provides instantaneous price feedback using the same mathematical model as the contract.

### 3. Simulation Engine
The "Research Layer."
- **High-Performance Modeling**: An in-memory implementation of the LMSR contract logic.
- **Agent Orchestration**: Handles multiple trading agents with independent beliefs and strategies.
- **Data Export**: Facilitates observation of price trends and convergence metrics.

## Data Flow: Buying Shares

1. **Input**: User specifies outcome ID and quantity.
2. **Local Calculation**: CLI uses `math.ts` to estimate the cost.
3. **Transaction**: CLI calls the contract's `buy` circuit.
4. **Witness Context**: The `witnesses.ts` provider fetches the current ledger state and performs the exact $C(q)$ calculation.
5. **Ledger Update**: If valid, the Midnight nodes update the ledger state with new share counts and contract volume.
