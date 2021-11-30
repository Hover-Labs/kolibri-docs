# StabilityFund Contract

## Overview

A `StabilityFund` contract collects funds in `kUSD` and `XTZ` for later use. It acts as the "Liquidator of Last Resort" and can be instructed to liquidate undercollateralized `Oven`s.

`StabilityFund`s can liquidate an oven (and acquire `XTZ` from the liquidation), and they can also disburse their `kUSD` and liquidated `XTZ`. 

`StabilityFund` is an extension of [`DevFund`](dev-fund.md), and all properties of a `DevFund` also apply to the `StabilityFund`.

### ACL Checking

In addition to the functions of the `DevFund`, the `Administrator` of the `StabilityFund` may direct it to liquidate ovens.

### Core Upgrade Path

Any contract which needs to interact with a `StabilityFund` should have a governable reference to the `StabilityFund`.

If a new `StabilityFund` contract is needed then:
(1) A new `StabilityFund` contract would be deployed
(2) The `Governor` would update every contract that interacts with the `StabilityFund` to point to the new `StabilityFund`.
(3) The `Governor` would transfer existing `kUSD` and `XTZ` to the new `StabilityFund`

### State Machine

The `StabilityFund` contract does not have a state machine.

## Storage

The `StabilityFund` contract stores the following, in addition to the storage inherited from [`DevFund`](dev-fund.md):
- `ovenRegistryContractAddress` (`address`): The address of the `OvenRegistry`. Ensures that `liquidate` is only called on registered `Oven`s.

All storage parameters are governable and can be changed by `governorContractAddress`.

## Entrypoints

The `StabililtyFund` contract has the following entrypoints, in addition to the storage inherited from [`DevFund`](dev-fund.md):
- `liquidate`: Liquidate an oven. Can only be called by the `administratorContractAddress`.
- `setOvenRegistryContract`: Update the `ovenRegistryContractAddress`. Can only be called by the `governorContractAddress`.
