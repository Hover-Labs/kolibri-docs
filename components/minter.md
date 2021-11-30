# Minter Contract

## Overview

The `Minter` contract provides core functionality to the Kolibri ecosystem. 

The `Minter` has three main functions:
1) Track the `Global Interest Index` for Kolibri
2) Provide implementation logic for deposit/withdraw/borrow/repay/liquidate operations.
3) Act as the `Minter` and `Burner` for the `Token` contract.

The different `Interest Indices` and the interest accrual methodology are documented in the "Interest Calculation" section of the [System Overview](#/project-info/general/intro).

The `Minter`'s functions can only be called by the `OvenProxy`. The `Minter` trusts all values that are passed in. 

### ACL Checking

Anyone may request a price via the `getXtzUsdPrice` entrypoint.

However, only the `Harbinger` contract can return price data via the `getXtzUsdPrice_callback` entrypoint.

### Upgrade Path

Any contract that points to the `Minter` should have a governable `address` for the `Minter`. In case of upgrades, a new `Minter` could be deployed, and governance could upgrade contracts to point to the new contract.

Additionally, the `Minter` acts as the `administrator` for the token contract. The `Minter` should always expose a function that allows the `Token` contract's administrator to be updated. If the `Minter` were to be replaced, the old `Minter` would set the administrator to the new `Minter`.

## Storage

The `Minter` stores the following:
- `tokenContractAddress` (`address`): The address of the `Token` contract. Used to mint and burn tokens.
- `governorContractAddress` (`address`): The address of the `Governor` contract. Used for ACL checking.
- `ovenProxyContractAddress` (`address`): The address of the `OvenProxy` contract. Used for ACL checking.
- `stabilityFundContractAddress`(`address`): The address of a `Fund` contract that represents the Stability Fund. Used to send fees to.
- `developerFundContractAddress`(`address`): The address of a `Fund` contract that represents the Developer Fund. Used to send fees to.
- `stabilityFee` (`nat`): The stability fee that is applied every 60 seconds. Represented as an integer with 10^-18 precision, ex: 100_000_000_000_000_000 = 10% interest / 60 secsonds.
- `lastInterestIndexUpdateTime` (`timestamp`): The last time the `Global Interest Index` was updated.
- `interestIndex` (`nat`): The value of the `Global Interest Index`.
- `stabilityDevFundSplit` (`nat`): The percentage of fees that go to the Developer Fund. Represented as an integer with 10^-18 precision, ex: 100_000_000_000_000_000 = 10% of all fees are given to the Developer fund, and the other 90% are given to the Stability Fund.
- `liquidationFeePercent` (`nat`): An additional percentage fee collected for liquidations. Represented as an integer with 10^-18 precision, ex: 80_000_000_000_000_000 = 8% fee on liquidations.
- `collateralizationPercentage` (`nat`): The required collateralization for a vault. Represented as an integer with 10^-18 precision, ex:
200_000_000_000_000_000_000 = 200% = there must be $2 of `XTZ` backing every $1 of `kUSD`.
- `ovenMax` (`option mutez`): The maximum amount of `XTZ` an oven may hold if the value is `Some`. If the value is `None` then there is no limit.

All parameters are governable.

## Entrypoints

The `Minter` has the following entrypoints:
- `getInterestIndex`: Calculate the `Global Interest Index` and return it to the caller.
- `default`: No-op, implemented to avoid stranding of funds.
- `borrow`: Borrow `kUSD` tokens against an `Oven`'s collateral. May only be called by the `OvenProxy`.
- `repay`: Repay `kUSD` tokens borrowed against an `Oven`'s collateral. May only be called by the `OvenProxy`.
- `deposit`: Deposit `XTZ` tokens into an `Oven`. May only be called by the `OvenProxy`.
- `withdraw`: Withdraw `XTZ` tokens from an `Oven`. May only be called by the `OvenProxy`.
- `liquidate`: Liquidate an under-collateralized `Oven`. May only be called by the `OvenProxy`.
- `updateParams`: Update the stability fee, liquidation fee, collateralization percentage or oven max. May only be called by the `Governor`.
- `updateContracts`: Update addresses for the `Governor`, `Token`, `OvenProxy`, `Stability Fund` or `Developer Fund` contracts. May only be called by the `Governor`.
- `updateTokenContractAdmin`: Update the administrator for the `Token` contract. May only be called by the `Governor`.
