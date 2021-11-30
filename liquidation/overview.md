### Liquidation

If an `Oven` becomes under collateralized, then anyone can liquidate it. This user is referred to as the **liquidator**.

During liquidation, the following happens:
1) The **liquidator** receives all `XTZ` in the `Oven`.
2) The **liquidator** repays all outstanding `kUSD` tokens, plus an additional percentage based **liquidation fee** assessed on the assets.
3) The `Oven` is marked as **liquidated** and no longer accepts deposits, withdrawals, borrows or repayments.

This is economically beneficial to the **liquidator**, since they are able to acquire XTZ at below market prices. Meanwhile, the `Oven` owner is penalized heavily for letting their `Oven` become undercollateralized, with the penalty funds being split between the stability fund and the development fund (the `devStabilityFundSplit` value in the `Minter` contract).

#### An example liquidation flow
<table class="table is-bordered">
    <thead>
        <th colspan="2">Assumptions</th>
    </thead>
    <tbody>
        <tr>
            <td><b>XTZ/USD Pair Price</b></td>
            <td><b>$2.00</b></td>
        </tr>
        <tr>
            <td><b>Oven Collateralization Ratio</b></td>
            <td><b>200%</b></td>
        </tr>
        <tr>
            <td><b>Liquidation Fee</b></td>
            <td><b>10%</b></td>
        </tr>
        <tr>
            <td><b>Dev Fund/Stability Fund Split</b></td>
            <td><b>10%</b></td>
        </tr>
    </tbody>
</table>

1. Alice creates an `Oven` and deposits 10 `XTZ` (worth $20). She borrows 10 `kUSD` (worth $10) against the collateral she has in the `Oven`. Since the oven has a collateralization ratio of 200%, it's the maximum she's able to borrow, and puts her at significant liquidation risk if the price of `XTZ` were to drop.
2. Some time later, the price of `XTZ/USD` drops to $1.50. The 10 `XTZ` in her `Oven` is now worth only $15, the 10 `kUSD `is still worth $10, and the `Oven` is now collateralized at 150%.
3. Bob notices the `Oven` is under collateralized and liquidates it.
4. Bob pays all outstanding `kUSD`, along with a 10% penalty (the liquidation fee), ($10 + $1 = $11) and receives $15 worth of `XTZ`
5. The `Oven` transitions to a liquidated state, the $10 in kUSD is burned/destroyed, and the $1 in fees is split between the `stability fund`, and the `development fund` in a 90/10 ratio (0.90 kUSD goes into the `stability fund` and 0.10 kUSD goes into the `development fund`)
