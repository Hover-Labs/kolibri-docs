# Oven Registry Contract

## Overview

The `OvenRegistry` contract provides a registry of `oven` contracts in the system. The root of trust in most Kolibri calls is that the call originated in an `oven` which is running code that `OvenFactory` deployed. `OvenRegistry` keeps track of those originated contracts to verify trust. 

At it's core, `OvenRegistry` is a map of `address` to `address`. The key is the `oven`'s address, and the value is the owner of the oven. Thus to determine if an contract is an `oven`, `OvenRegistry` simply checks if the contract's address is present as a key in the map.

### Callbacks

`OvenRegistry` does not attempt to return data to the caller. Instead, it will simply fail the call if an `oven` is not whitelisted. This will cause all related contract calls to fail atomically in Tezos. This is in line with Kolibri's data flow priniciples.

### Data Duplication

Astute readers will realize that the `owner` field exists in both `oven` contracts and in the registry. Kolibri duplicates this data to provide maximum flexibility in the future for the core logic to be able to override ACL checks or perform their own ACL checks. Since both contracts maintain `owner` as immutable, it is not troublesome to keep them in sync. 

#### Possible Future Work

In the future, ACL checks could be removed from the `oven` contract in a governance upgrade and performed by the core. This change would not require upgrades to the `OvenRegistry` contract. 

Further in the future, `oven` owners may choose to allow others to manage their `oven` on their behalf. An upgrade to `OvenRegistry` to keep track of `delegated` owners is possible. 

### ACL Checking

Anyone may query the `OvenRegistry` to see if a contract is a known `oven`.

However, only the `OvenFactory` can add an `oven` to the registry.

### Core Upgrade Path

`OvenRegistry` is implemented outside of the core so that upgrades to the core do not require a map to be migrated. By prioritizing modularity, upgrades to `OvenFactory` or core logic can be performed by simply updating contract pointers.

In the event `OvenRegistry` needs to be updated, the system will likely need to be temporarily paused via PauseGuardian or Governance, and the state of the `OvenRegistry` contract will be migrated to a new registry contract and the system will need to be restarted. If this becomes a frequent problem, an atomic upgrade path could be baked into `OvenRegistry`.

### State Machine

`OvenRegistry` does not maintain a state machine.

## Storage

The `OvenRegistry` stores the following:
- `governorContractAddress` (`address`): The address of the `Governor` contract, used for ACL checking.
- `ovenFactoryContractAddress` (`address`): The address of the `OvenFactory`, used for ACL checking.
- `ovenMap` (`BigMap<address, address>`): A map of known `oven`s. Keys are `oven` addresses, values are `oven` owners.

Both the `governorContractAddress` and `ovenFactoryContractAddress` are governable by the `governorContractAddress`.


## Entrypoints

The `OvenRegistry` has the following entrypoints:
- `isOven`: Verify the given address is an `oven`. If it is not, atomically fail the transaction.
- `addOven`: Add an `oven` and owner address to the registry. This entrypoint may only be called by the `OvenFactory.`
- `setGovernorContract`: Set a new `Governor` contract. This method may only be called by the `Governor`.
- `setOvenFactoryContractAddress`: Set a new `OvenFacotry`. This method may only be called by the `Governor`.
- `default`: No-op. Always fails to prevent transfer of XTZ into contract.
