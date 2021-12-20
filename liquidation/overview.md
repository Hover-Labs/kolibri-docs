### Liquidation

In Kolibri a liquidation occurs when an `Oven` drops below the collateralization threshold. 

There are three types of users who can perform a liquidation:

1) Liquidity Pool: A shared pool of `kUSD` pledged to help liquidate undercollateralized ovens. When the pool liquidates an `Oven`, it repays all outstanding `kUSD` tokens, plus an additional percentage based **liquidation fee** assessed on the loan amount, receives `XTZ` as collateral and immediately swaps it to `kUSD` on Quipuswap. Anyone can direct the pool to liquidate an `Oven` and receive a **liquidation reward** (currently 10% of the collateral). Liquidity Providers in the Liquidity Pool will ratably split the remaining 90% of the rewards. 
After adopting KIP-006 the Liquidity Pool is prioritized to liquidate an `Oven` below the `collateralizatizationRequirement`(currently set to 200%).
2) Private Entity: A private entity may choose to liquidate an `Oven` by repaying the loan plus an additional percentage based **liquidation fee** assessed on the loan amount and receiving the collateral in the `Oven` in return. 
After adopting KIP-006 a new parameter `privateLiquidationThreshold`(currently set to 20%) was introduced. A Private Entity can only liquidate an `Oven` below `collateralizatizationRequirement - privateLiquidationThreshold` collateralization.
3) Stability Fund: The protocol maintains an emergency backstop that can be used to liquidate ovens that are either underwater or too large for the Liquidity Pool or private entities to liquidate. Although the Stability Fund can liquidate an `Oven` below the `collateralizatizationRequirement`, in practice it should only step in as a liquidator of last resort

This is economically beneficial to the **liquidator** to liquidate an `Oven`, since they are able to acquire `XTZ` at below market prices. Meanwhile, the `Oven` owner is penalized heavily for letting their `Oven` become undercollateralized, with the penalty funds being split between the Stability fund and the Development fund (currently split in a 90/10 ratio, the `devStabilityFundSplit` value in the `Minter` contract set to `100000000000000000` or 10%).

The following graphics depicts the set of rules for liquidations in the Kolibri Protocol with the `collateralizatizationRequirement` set to 200% and the `privateLiquidationThreshold` set to 20%:

![kolibri liqs](https://user-images.githubusercontent.com/69350535/146745446-1bd852bf-a45f-454f-92fe-784f6128ae59.png)

#### An example liquidation flow
<table class="table is-bordered">
    <thead>
        <th colspan="2">Assumptions</th>
    </thead>
    <tbody>
        <tr>
            <td><b>XTZ/USD Pair Price</b></td>
            <td><b>$4.00</b></td>
        </tr>
        <tr>
            <td><b>Oven Collateralization Ratio</b></td>
            <td><b>200%</b></td>
        </tr>
        <tr>
            <td><b>Private Liquidation Threshold</b></td>
            <td><b>20%</b></td>
        </tr>
         <tr>
            <td><b>Liquidation Reward</b></td>
            <td><b>10%</b></td>
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

1. Alice creates an `Oven` and deposits 10 `XTZ` (worth $40). She borrows 20 `kUSD` (worth $20) against the collateral she has in the `Oven`. Since the oven has a collateralization ratio of 200%, it's the maximum she's able to borrow, and puts her at significant liquidation risk if the price of `XTZ` were to drop.
2. Some time later, the price of `XTZ/USD` drops to $3.90. The 10 `XTZ` in her `Oven` is now worth only $39, the 20 `kUSD` loan is still worth $20, and the `Oven` is now collateralized at 195% and can be liquidated by the Liquidity Pool 
3. Bob notices the `Oven` is liquidatable by the Liquidity Pool and calls the `liquidate` entrypoint.
4. The Liquidity Pool pays all outstanding `kUSD`, along with a 10% **liquidation fee** (20 `kUSD` + 2 `kUSD` = 22 `kUSD`), Bob recieves a 1 `XTZ` **liquidation reward** (10% of the collateral), the rest of the collateral 9 `XTZ` (worth $35.1) is sold on Quipuswap to `kUSD` to be distributed to the Liquidity Pool.
5. The `Oven` transitions to a liquidated state, the 20 `kUSD` is burned/destroyed, and the 2 `kUSD` penalty is split between the `Stability fund`, and the `Development fund` in a 90/10 ratio (1.80 `kUSD` goes into the `Stability fund` and 0.20 `kUSD` goes into the `Development fund`)
3*. For some reason the `Oven` hasn't been liquidated by the Liquidity Pool. Some time later the price of `XTZ/USD` drops to $3.50. The 10 `XTZ` in her `Oven` is now worth only $35, the 20 `kUSD` is still worth $20, the `Oven` is now collateralized at 175%, the collateralization ratio dropped below `collateralizatizationRequirement - privateLiquidationThreshold` (175% < (200 - 20)%) and it can be liquidated by a private liquidator.
4*. Bob notices he can liquidate the `Oven`. 
5*. Bob pays all outstanding `kUSD`, along with a 10% **liquidation fee**, 20 `kUSD` + 2 `kUSD` = 22 `kUSD`, and receives $35 worth of `XTZ`
6*. The `Oven` transitions to a liquidated state, the 20 kUSD is burned/destroyed, the 2 `kUSD` penalty is split between the `Stability fund` and the `Development fund` in a 90/10 ratio (1.80 `kUSD` goes into the `Stability fund` and 0.20 `kUSD` goes into the `Development fund`)
