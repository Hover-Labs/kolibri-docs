# OvenProxy Contract

## Overview

The `OvenProxy` has two responsibilities:
1) Provides a layer of abstraction between `Oven`s and the `Minter`
2) Fetch auxilary data and validate system state before forwarding calls to `Minter`

In the case of (1), `Oven` contracts do not have mutable code. Thus they always will call to the same contract. Without an abstraction, the `Minter` logic would not be upgradeable. Thus, `OvenProxy` serves as a consistent target for calls from `Oven`s and can redirect to a new `Minter` if required. 

In the case of (2), separating logic out of the `Minter` makes the system more maintainable and easy to reason about. It also makes the `Minter`'s code size smaller, and without this separation, the `Minter` contract would become too large to deploy on chain. The `OvenProxy` has two responsibilities:
- Fetch data from the `Oracle` if the `Oven`'s call requires it
- Make sure that the system is not paused by the Pause Guardian.

### ACL Checking

The `OvenProxy` does not have a public interface. 

The `deposit`, `withdraw`, `borrow`, `repay` and `liquidate` entrypoints must only be called by an `Oven` contract. 
The `withdraw_callback`, `borrow_callback` and `liquidate_callback` entrypoints must only be called by the `Oracle` contract.
The `updateState` entrypoint may only be called by the `Minter` contract.
The `pause` entrypoint may only be called by the `Pause Guardian`.
The `unpause`, `setGovernorContract`, `setMinterContract`, `setOvenRegistryContract`, and `setOracleContract` may only be called by the `Governor`.

### Upgrade Path

The `OvenProxy` is not meant to be upgraded regularly. If changes to the system are required, it is much prefered to originate a new `Minter` contract and change the `OvenProxy` to point at the new `Minter`.


If an upgrade is needed, a several stage process could be deployed:
1) A new `OvenProxy` is deployed
2) The `OvenFactory` contract points new `Oven` contracts to the new `OvenProxy`.
3) The old `OvenProxy` has the `minterAddress` set to the new `OvenProxy`.

In effect, this adds an additional layer of indirection on older ovens. This will cost additional fees in gas which acts as a tax for using older `Oven` contracts. We assume that as this tax increases, users will opt to open newer `Oven`s and retire older ones.

### State Machine

`OvenProxy` is a state machine of moderate complexity to accomodate multiple entrypoints which require a callback from the `Oracle`. 

In short, the `OvenProxy` is in two states:
(1) `IDLE`: Not waiting on anyone
(2) Waiting for an Oracle callback (`BORROW_WAITING_FOR_ORACLE`, `WITHDRAW_WAITING_FOR_ORACLE`, `LIQUIDATE_WAITING_FOR_ORACLE`)

The `OvenProxy` moves from the `IDLE` state to one of the "waiting" states. From the "waiting" state it moves back to the idle state.

Specifically:
From the `ILDE` state, the `OvenProxy` can transition to the `BORROW_WAITING_FOR_ORACLE`, `WITHDRAW_WAITING_FOR_ORACLE`, or `LIQUIDATE_WAITING_FOR_ORACLE` state when the `borrow`, `withdraw` or `liquidate` entrypoints are called respectively. In the `IDLE` state, `borrowParams`, `withdrawParams` and `liqudiateParams` are all `none`.

In the `BORROW_WAITING_FOR_ORACLE` state, the `OvenProxy` can only transition back to the `IDLE` state when `borrow_callback` is called. In the `BORROW_WAITING_FOR_ORACLE` state, `borrowParams` is always `some`.

In the `WITHDRAW_WAITING_FOR_ORACLE` state, the `OvenProxy` can only transition back to the `IDLE` state when `withdraw_callback` is called. In the `WITHDRAW_WAITING_FOR_ORACLE` state, `withdrawParams` is always `some`.


In the `LIQUIDATE_WAITING_FOR_ORACLE` state, the `OvenProxy` can only transition back to the `IDLE` state when `liquidate_callback` is called. In the `LIQUIDATE_WAITING_FOR_ORACLE` state, `liquidateParams` is always `some`.

## Storage

The `OvenProxy` stores the following:
- `minterContractAddress` (`address`): The address of the `Minter` contract, used for ACL checking and to forward calls
- `governorContractAddress` (`address`): The address of the `Governor` contract, used for ACL checking
- `ovenRegistryContractAddress` (`address`): The address of the `OvenRegistry` contract, used for ACL checking
- `pauseGuardianContractAddress` (`addresss`): The address of the `Pause Guardian`, used for ACL checking
- `oracleContractAddress` (`address`): The address of the `Oracle` contact, used for ACL checking and the price feed
- `paused` (`boolean`): Whether the `Pause Guardian` has paused the system.
- `state` (`nat`): The state machine's state, see descriptions above. 
- `borrowParams` (`option(tuple)`): An  set of parameters that were sent to the `borrow` entrypoint. Stored while waiting for `Oracle` callback.
- `withdrawParams` (`option(tuple)`): An  set of parameters that were sent to the `withdraw` entrypoint. Stored while waiting for `Oracle` callback.
- `liquidateParams` (`option(tuple)`): An  set of parameters that were sent to the `liquidate` entrypoint. Stored while waiting for `Oracle` callback.

All contract pointers (`minterContractAddress`, `governorContractAddress`, `ovenRegistryContractAddress`, `pauseGuardianContractAddress`, `oracleContractAddress`) are governable and can be set by the `governorContractAddress`. Additionally, `paused` boolean can be set to `false` by the `governorContractAddress`.

The `paused` boolean can be set to true by the `Pause Guardian`.

## Entrypoints

The `OvenProxy` has the following entrypoints:
- `default`: No-op. Always fails to prevent transfer of XTZ into contract.
- `borrow`: Borrow `kUSD` against collateral in an `Oven`. May only be called by an `Oven` contract.
- `repay`: Repay borrowed `kUSD`. May only be called by an `Oven` contract.
- `deposit`: Deposit `XTZ` into an `Oven`. May only be called by an `Oven` contract.
- `withdraw`: Withdraw `XTZ` from an `Oven`. May only be called by an `Oven` contract.
- `liquidate`: Liquidate an `Oven`.  May only be called by an `Oven` contract.
- `borrow_callback`: A callback that receives data from the `Oracle` contract in the `borrow` flow. May only be called by the `Oracle` contract.
- `withdraw_callback`: A callback that receives data from the `Oracle` contract in the `withdraw` flow. May only be called by the `Oracle` contract.
- `liquidate_callback`: A callback that receives data from the `Oracle` contract in the `liquidate` flow. May only be called by the `Oracle` contract.
- `updateState`: Proxies new state from the `Minter` back to the `Oven`. May only be called by the `Minter` contract. 
- `pause`: Temporarily pauses the system. May only be called by the `Pause Guardian`.
- `unpause`: Unpauses the system. May only be called by the `Governor`.
- `setGovernorContract`: Update the `Governor` contract. May only be called by the `Governor`.
- `setOvenRegistyContract`: Update the `OvenRegistry` contract. May only be called by the `Governor`.
- `setOracleContract`: Update the `Oracle` contract. May only be called by the `Governor`.
- `setMinterContract`: Update the `Minter` contract. May only be called by the `Governor`.
- `setPauseGuardian`: Update the `PauseGuardian` contract. May only be called by the `Governor`.
