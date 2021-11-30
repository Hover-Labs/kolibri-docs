# DevFund Contract

## Overview

A `DevFund` contract collects funds in `kUSD` and `XTZ` for later use. 

The `DevFund` can disburse any `XTZ` and `kUSD` it contains.

The `DevFund` contracts have two permissions on them:
(1) `Administrator`: Can manage the `Fund`'s baker
(2) `Governor`: Can swap `Governor`s, `Administrator`s and transfer `XTZ`

Long term, `Administrator` should be a multi-sig or governance function controlled without a time delay, while `Governor` should be a higher privileged multi-sig or DAO with a time lock.

### ACL Checking

Anyone may deposit `XTZ` into a `DevFund`.

The `Administrator` can choose the baker for the `DevFund`.
The `Governor` can change the administrator and other contract references. The `Governor` can also disburse `kUSD` and `XTZ` funds.

### Core Upgrade Path

Any contract which needs to interact with a `DevFund` should have a governable reference to the `DevFund`.

If a new `DevFund` contract is needed then:
(1) A new `DevFund` contract would be deployed
(2) The `Governor` would update every contract that interacts with the `DevFund` to point to the new `DevFund`.
(3) The `Governor` would transfer existing `kUSD` and `XTZ` to the new `DevFund`

### State Machine

The `DevFund` contract does not have a state machine.

## Storage

The `DevFund` contract stores the following:
- `governorContractAddress` (`address`): The `Governor`. Used for ACL checking.
- `administratorContractAddress` (`address`): The `Administrator`. Used for ACL checking.

All storage parameters are governable and can be changed by `governorContractAddress`.

## Entrypoints

The `DevFund` contract has the following entrypoints:
- `default`: No-op. Implemented so the contract can receive XTZ.
- `setDelegate`: Set the baker for the contract. Can only be called by the `administratorContractAddress`.
- `send`: Send `XTZ` to a recipient. Can only be called by the `governorContractAddress`.
- `setGovernorContract`: Update the `governorContractAddress`. Can only be called by the `governorContractAddress`.
- `setAdministratorContract`: Update the `administratorContractAddress`. Can only be called by the `governorContractAddress`.
