# Silo Core Design

## Overview

Silo Core is the lending engine for Silo V2. It lets anyone create isolated
two-asset lending markets. Each market keeps its assets, positions, and risk
parameters separate from every other market, so risk is contained within that
market rather than shared protocol-wide.

A market consists of one vault for each asset and a shared immutable
configuration. Users can provide one asset as collateral and borrow the paired
asset, subject to the market's risk rules.

## Position types

For each asset, the system represents three kinds of user position:

- **Protected collateral**: supplied assets that cannot be borrowed by other
  users.
- **Borrowable collateral**: supplied assets that may be lent out and can earn
  lending yield.
- **Debt**: a user's outstanding borrowed amount.

These positions are represented by transferable tokens, allowing integrations
to observe and use positions while Silo Core retains responsibility for the
underlying lending and risk rules.

## Core functionality

Silo Core provides the following market operations:

- Depositing assets as protected or borrowable collateral.
- Borrowing against collateral held in the paired asset vault.
- Repaying debt and withdrawing collateral.
- Moving a position between protected and borrowable collateral.
- Accruing interest and distributing the resulting value to lenders and fee
  recipients.
- Checking whether a position remains adequately collateralized.
- Liquidating positions that no longer meet the market's solvency requirements.
- Providing standardized vault and flash-loan interfaces.

## Risk model

Each market has its own fixed risk configuration. This includes limits for
borrowing, the point at which a position becomes liquidatable, fees, pricing
sources, and interest-rate behavior.

Price-oracle interfaces supply the values needed for borrowing and solvency
decisions. The core deliberately depends on interfaces rather than a specific
price-feed implementation, so a market can use an appropriate oracle
integration without changing the lending engine.

## Interest and fees

Borrowing rates are determined independently for each market asset through a
pluggable interest-rate model. Interest accrues as market activity changes and
is reflected in lender value, with configured portions available to protocol
and market-creator fee recipients.

## Market creation

The factory creates markets permissionlessly from reusable implementations and
records their shared configuration. Once a market is created, its asset pair
and risk setup are fixed.

The deployer provides a higher-level creation path for assembling a market's
supporting configuration, such as rate models and optional integrations, before
creating the market.

## Integration points

Silo Core is designed to be extended through well-defined interfaces:

- **Oracles** provide asset pricing.
- **Interest-rate models** define borrowing-rate behavior.
- **Hooks** let external systems react to position-token balance changes, such
  as for incentives or accounting.
- **Leverage and flash-loan callbacks** let external contracts compose
  advanced operations with the core.
- **Router and lens contracts** provide convenient write batching and read-only
  views for applications and integrations.

These integrations add functionality around the market while the core remains
the authority for balances, debt, interest, and risk enforcement.

## Scope

This directory contains the core lending-market contracts and their public
interfaces. Concrete oracle providers, incentive systems, and application
front ends are intentionally outside this scope.
