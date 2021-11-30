# Kolibri

Kolibri is an Tezos based stablecoin built on Collateralized Debt Positions (CDPs) known as `Oven`s.

### Overview

Kolibri uses CDPs (referred to as an `Oven`) to collateralize a soft pegged USD-stable value asset, `kUSD`.

Each `Oven` has four functions:
- `Deposit`: Place `XTZ` into the `Oven`
- `Withdraw`: Remove `XTZ` from the `Oven`
- `Borrow`: Borrow `kUSD` against the `Oven` using XTZ as collateral
- `Repay`: Repay `kUSD` that was borrowed against the `Oven`.

### Stability Fee

A stability fee is applied to borrowed `kUSD`. It is accrued in terms of `kUSD`. It is percentage based fee applied to all outstanding `kUSD` (borrowed `kUSD` + stability fee). Interest is assessed every minute (about every block on the Tezos chain). The stability fee is adjusted via governance to increase or decrease the incentives to borrow or repay `kUSD` if the asset loses a peg.

Negative stability fees are not supported in Kolibri but may be added in the future via a contract upgrade.

### Collateralization Ratio

In order to remain solvent, an `Oven` must maintain a minimum **collateralization ratio**. The collateralization ratio is computed as:
```
Collateralization Ratio = ((XTZ in Oven * Price of XTZ/USD) / (Borrowed kUSD + Stability Fees)) * 100 
```

If a `Oven` drops below the **collateralization ratio**, then it is said to be **under collateralized**. `Oven` owners should take care to keep their position above the collateralization ratio, by either locking more `XTZ` or repaying `kUSD` when the collateralization ratio drops.

The Kolibri system will prevent users from borrowing `kUSD` such that an `Oven` becomes under collateralized, or withdrawing `XTZ` to cause the `Oven` to become undercollateralized. However, the price of `XTZ` still fluctuates, which means an `Oven` can become undercollateralized without user action. At that point, a liquidation process kicks in to restore stability to the system.

### Compound Interest Calculations

Kolibri maintains a constant, called `stabilityFee` which calculates interest. The stability fee is a fixed interest amount, calculated on a 60 second basis. GUIs should choose to display this interest amount as an APY.

Rather than calculate compound interest, Kolibri uses a linear approximation to calculate interest. That is to say that if two time periods have elapsed
then the interest calculation will be:
```
newInterest = oldInterest * (1 + (numPeriods * stabilityFee))
```

Kolibri compounds interest within every call to the system. Given high enough usage, the system will be called at least every 60 seconds, which means interest will compound as expected. In periods of low usage, the interest will be linearly approximated, which is believed to be negligible in a system which represents numbers as small as `10^18`.

### Interest Calculations on `Oven`s

Calculating interest on a potentially unbound number of `Oven`s is not feasible in a system like Tezos where you pay per unit of computation. To solve this, Kolibri introduces the idea of an `Interest Index`.

The `Minter` contains a `Global Interest Index`. This value is initially set to `1`.
Each `Oven` contains its own `Interest Index`. This value is set to the value of the `Global Interest Index` at origination time for the `Oven`.

Whenever the `Minter` is invoked, it compounds the `Global Interest Index`, using the elapsed time, stability fee, and linear approximation technique described above.

When an `Oven` is originated, the `Minter` calculates the current `Global Interest Index` and gives it to the `Oven`. Whenever an `Oven` interacts with the `Minter`, the following process occurs:
- `Minter` recalculates `Global Interest Index`
- A ratio is calculated between the `Global Interest Index` and the `Oven`'s `Interest Index` to determine the interest the `Oven` has accrued
- The `Oven`'s `Interest Index` is updated to be the same as the `Global Interest Index`

## Oracle

Kolibri needs accurate data to function. Data is provided via the [Harbinger Price Feed](https://github.com/tacoinfra/harbinger) via the `Oracle` contract. The `Oracle` contract is replaceable and can be replaced to pull from another data source if needed.
