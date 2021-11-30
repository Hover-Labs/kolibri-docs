# Token Contract

## Overview

The `Token` contract is an FA1.2 Token Contract.

The FA1.2 token standard is available in the [Tezos Improvement Proposal Repository](https://gitlab.com/tzip/tzip/blob/master/A/FA1.2.md).

The token contract was copied verbatim from [SmartPy's implementation](http://smartpy.io/dev/?template=fa12.py) on 11/02/2020 with some minor changes noted below.

## Overview

Details of the `token` contract are omitted since the FA1.2 standard is well understood and described extensively elsewhere. Instead, we opt to discuss specifics in the Kolibri system. 

The `token` contract controls the `kUSD` tokens. Tokens are specified in 10^18 precision, as are all values in Kolibri. The administrator of the token contract is the `minter` contract, who controls minting and burning of `kUSD` tokens based on it's own internal logic. 

# Upgrade Path

Any contract which interacts with the `Token` contract should have a governable reference. When a new `Token` contract is deployed, then the references should be updated. Additionally, if the admin needs to be changed, then the `Governor` should call `setAdmin` on the `Token` contract.

### Changes

Kolibri tries to use this contract without modification to preserve security. Kolibri makes the following modifications:
- Add a `default` entrypoint and disallow transfers if they contain value. This behavior is explicitly defined to make the restriction obvious to future engineers.
- Add a "debt ceiling" that is configurable through governance.
- Add a `governor` which can change the debt ceiling.

### ACL Checking

The `Token` contract exposes the following new methods, above the FA1.2 standard contract:
- `setGovernorContract` and `setDebtCeiling` may only be called by the `Governor`.
- `setAdministrator` may only be called by the `Governor` (which differs from the the standard implementation, where `setAdministrator` could only be called by an administrator)

## Storage

The `Token` contract exposes the following new storage fields, above the FA1.2 standard contract:
- `governorContractAddress` (`address`): The address of the `Governor` contract, used for ACL checking
- `debtCeiling` (`nat`): The debt ceiling. The token's `totalSupply` can never exceed the debt ceiling.

Both storage items are governable. 

## Entrypoints

The `Token` contract exposes the following new methods, above the FA1.2 standard contract:
- `setGovernorContract`: Update the `Governor` contract. May only be called by the `Governor`.
- `setDebtCeiling`: Update the debt ceiling. May only be called by the `Governor`.
