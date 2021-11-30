# Developer and Stability Funds

Accrued stability and liquidation fees are deposited in two funds. The split is determined by the **developer fund split**.

The role of the **stability fund** is to be a liquidator of last resort. If the price of the outstanding `kUSD` is greater than the `XTZ` value locked in the `Oven`s then a rational economic actor would not liquidate, and the `Oven` is then considered **underwater**. This event should not occur since it is unlikely that `Oven` values would go from undercollateralized to underwater fast enough that rational actors would not liquidate. However, in some sort of systemic black swan event, the **stability fund** acts as a powerful counter-balance and can liquidate **underwater** `Oven`s to restore collateralization (and thus stability) to the system.

The **developer fund** is a discretionary fund used to fund future developments to Kolibri. Distributions are determined via a planned governance mechanism.
