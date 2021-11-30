# Oven Contract

## Overview

An oven contract is a simple key-value store which holds data about a CDP and the CDP's collateral. Oven's provide their data and collateral into the Kolibri system and receive outputs and collateral from the Kolibri system in a callback to update their state.

### Trusted Code
Oven's are **not** upgradeable via governance, and their code is trusted in the system. Allowing a governance upgrade to the Oven could modify behavior maliciously to steal funds. 

The only logic in `Oven`s is:
1) Convert `mutez` (specified in 10^-6) to Kolibri's 10^-18 precision
2) ACL Checking

### ACL Checking

Only the `Oven`'s owner can invoke the `withdraw`, `borrow` or `repay` functions. This prevents other user's from performing actions that affect the owner's collateral. Additionall, the `setDelegate` operation in the `Oven` is also protected behind this `ACL`. 

The `Oven` allows anyone to call the `default` (aka `deposit`) endpoint. This is because `Oven`s can be delegated to bakers who will want to pay out funds. 

### Core Upgrade Path

All `Oven`s must point to the core contract in order to operate with the Kolibri system. The core is upgradeable via governance, where a new core contract is deployed and other contracts point to it. Since `oven`s are immutable, they will always point to the same contract. 

To solve this, an `OvenProxy` sits between all `Oven`s and the core contract. The `OvenProxy` can be upgraded to point at a new core contract, if needed. `Oven`s still point to the same `OvenProxy` and don't need to be modified. 

If the `OvenProxy` needed to be upgraded, then the existing `OvenProxy` could be wired to attach to a new `OvenProxy`. 

### Oven Upgrade Path

`OvenFactory` originates `Oven` contracts. If a new `Oven` contract is needed, then `OvenFactory` will need to be upgraded via governance. Old `Oven`s cannot be forced to be upgraded. Instead, the `OvenProxy` could be changed via governance to point to an alternative core contract which could levy additional fees or taxes on old `Oven` contracts to incentivize upgrades.

This is a complicated and work intensive process. `Oven` contracts are designed to be as simple as possible and upgradees are not expected to be required. 

### State Machine

`Oven`s exist in two states:
- Not Liquidated
- Liquidated

`Oven`s defer the meaning of this state to the core contracts and simply update the state according to the callback.

## Storage

The `Oven` stores the following:
- `owners` (`address`): The owner of the `oven`, used for ACL checking. 
- `borrowedTokens` (`nat`): The number of tokens borrowed against this `oven`.
- `stabilityFeeTokens` (`int`): The number of tokens accrued in stability fees against borrowed tokens.
- `interestIndex` (`int`): The local interest index for this oven.
- `isLiquidated` (`bool`): Indicates if the oven is liquidated.
- `ovenProxyContractAddress` (`address`): The address of the `OvenProxy` which proxies all calls into the core.

No storage parameters are governable.

## Entrypoints

The `Oven` has the following entrypoints:
- `default`: Deposit collateral into the `Oven`.
- `withdraw`: Remove collateral from the `Oven`.
- `borrow`: Borrow tokens against the collateral in the `Oven`
- `repay`: Repay tokens borrowed against the collateral in the `Oven`.
- `setDelegate`: Set (or clear) the delegate for the `Oven`.
- `liquidate`: Initiate liquidation of the oven if it is under collateralized.
- `updateState`: Receives a callback from Kolibri's core. This entrypoint may only be called by `ovenProxyContractAddress`.

