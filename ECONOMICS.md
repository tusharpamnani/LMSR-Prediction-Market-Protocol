# Economic Model

The LMSR protocol is designed to aggregate information from market participants.

## Information Aggregation

Participants trade when their belief differs from the market price.

As trades occur, the price converges toward the consensus probability.

## Liquidity Parameter

The parameter b controls market depth.

Low b:
- prices move quickly
- high volatility

High b:
- prices move slowly
- deep liquidity

## Bounded Loss

The maximum loss for the market maker is:

L_max = b ln(n)

Where n is the number of outcomes.

## Manipulation Cost

Changing the market price requires purchasing shares.

The capital required increases with b and the magnitude of the price shift.
