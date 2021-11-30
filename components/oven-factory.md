# OvenFactory Contract

## Overview

The `OvenFactory` provides an on-chain factory to originate `Oven`s and registers the originated `Oven`s in the `OvenRegistry`.  `Oven` code must be trusted and thus it needs to originate from a factory. 

### ACL Checking

Anyone may originate an `Oven` via the `makeOven` entrypoint.

However, only the `Minter` contract can return price data via the `makeOven_minterCallback` entrypoint.

### Upgrade Path
`
Any contract that points to the `OvenFactory` should have a governable `address` for the `OvenFacotry`. In case of upgrades, a new `OvenFactory` could be deployed, and governance could upgrade contracts to point to the new factory.

### State Machine

`OvenFactory` maintains a simple state machine with two states: (1) `IDLE` and (2) `WAITING_FOR_INTEREST_INDEX`. 

From the `IDLE` state, the `OvenFactory` can only transition to the `WAITING_FOR_INTEREST_INDEX` state. The `makeOven` can only be invoked when in the `IDLE` state. Invoking the entrypoint transitions the `OvenFactory` to the `WAITING_FOR_INTEREST_INDEX` state. In the `IDLE` state, `makeOvenOwner` is always `none`.

From the `WAITING_FOR_INTEREST_INDEX` state, the `OvenFactory` can only transition to the `IDLE` state. The `makeOven_minterCallback` can only be invoked when in the `WAITING_FOR_INTEREST_INDEX` state. Invoking the entrypoint transitions the `OvenFactory` to the `IDLE` state.  In the `WAITING_FOR_INTEREST_INDEX` state, `makeOvenOwner` is always `some`.

## Storage

The `OvenFactory` stores the following:
- `governorContractAddress` (`address`): The address of the `Governor` contract, used for ACL checking.
- `ovenRegistryContractAddress` (`address`): The address of the `OvenRegistry` contract, used for ACL checking and to register `Oven`s
- `ovenProxyContractAddress` (`address`): The address of the `OvenProxy` contract, used to originate `Oven`s with a proper configuration.
- `minterContractAddress` (`address`): The address of the `Minter` contract, used for ACL checking and to retrieve interest indices.
- `state` (`nat`): The current state of the contract; see 'State Machine' above.
- `makeOvenOwner` (`option(contract(address))`): A register that stores the owner of the new `Oven` when the `OvenFactory` is waiting for the interest index.
- `initialDelegate` (`option key_hash`): The address of the initial delegate for all ovens.

All contract pointers (`governorContractAddress`, `ovenRegistryContractAddress`, `ovenProxyContractAddress`, `minterContractAddress`) are governable and can be set by the `Governor`. `initialDelegate` is also governable by the `Governor`.

## Entrypoints

The `OvenFactory` has the following entrypoints:
- `makeOven`: Create a new oven.
- `makeOven_minterCallback`: Callback to recieve interest data needed to originate an oven.. This entrypoint may only be called by the `minterContractAddress.`
- `default`: No-op. Always fails to prevent transfer of XTZ into contract.
- `setGovernorContract`: Update the `governorContractAddress`. This entrypoint may only be called by `governorContractAddress`.
- `setOvenRegistryContract`: Update the `ovenRegistryContractAddress`. This entrypoint may only be called by `governorContractAddress`.
- `setOvenProxyContract`: Update the `ovenProxyContractAddress`. This entrypoint may only be called by `governorContractAddress`.
- `setMinterContract`: Update the `minterContractAddress`. This entrypoint may only be called by `governorContractAddress`.
- `setInitialDelegate`: Update the `initialDelegate`. This entrypoint may only be called by `governorContractAddress`.