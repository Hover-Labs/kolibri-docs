## Overview

A **Liquidity Pool** is a shared pool where users contribute funds in order to liquidate under-collateralized ovens.

Users deposit **kUSD tokens** in the pool and receive **QLkUSD** (**Q**uipuswap **L**iquidating **kUSD**) tokens in return. The QLkUSD tokens entitle the holder to the original amount of kUSD tokens deposited, plus any additional kUSD tokens earned from liquidating ovens. QLkUSD tokens are liquid and may be sent to other users or used in other applications. Any user who holds a QLkUSD may return the token to the pool and receive the associated kUSD at any time.

## How It Works
The pool contains kUSD from many users who have pledged their kUSD to liquidate ovens on Kolibri. When an oven is undercollateralized, anyone may liquidate it, provided they have sufficient kUSD to repay the loan. When an oven is liquidated, the liquidator receives the collateral (XTZ) in the oven.

Any user can act as a **liquidator** and can initiate a transaction that will use the pooled kUSD to liquidate an oven. When an oven is liquidated, the pool receives XTZ, which should be worth more than the kUSD repaid.

The pool also pays a percentage of the received XTZ directly to the liquidator. This payment reimburses the liquidator for their transaction fees, and it rewards the liquidator for using the pool, rather than selfishly liquidating the oven themselves.

The pool then trades the remaining XTZ for kUSD on  [Quipuwap](https://quipuswap.com/), a decentralized exchange on Tezos. The kUSD received in the trade are distributed ratably to all holders of the QLkUSD tokens, who will receive their portion of the new kUSD when they redeem their QLkUSD tokens.

## Benefits
- **kUSD Contributors**: Users contributing kUSD receive several benefits for using the pool. First, they can work together to liquidate ovens larger than the collateral than any individual holder may have on hand.
  Secondly, if users all competed to liquidate the oven individually they would be competing on who was the fastest and had the highest transaction fees. The fastest user with the highest fees would receive everything, while all other users received nothing. By aligning all users together, users are likely to receive a portion of liquidation rewards more consistently.
- **Liquidators**: Users who use the pool to liquidate are able to receive a portion of liquidated rewards without having to buy or hold kUSD.
- **Kolibri Protocol**: The Kolibri Protocol requires ovens to be liquidated when they go underwater in order to maintain peg. The Kolibri protocol benefits from the liquidity pool because systemic risks from ovens that are too large for any individual to liquidate are more easily liquidated.

## Risks

**Smart Contract Risk**
All smart contract based systems carry smart contract risk, the risk that smart contracts do not work as intended. Please see the  [risks page ](http://localhost:8080/docs/security/risks) as part of the Kolibri documentation for more info about the general risks with using smart contracts on Tezos.
There is no way to mitigate smart contract risk completely, however, good software engineering practices can help to build confidence in the security and correctness of implementation. The Kolibi Liquidity Pool contracts are  [open sourced](https://github.com/hover-labs/liquidation-pool)  and well tested, however, they **have not undergone a security audit**.

**Market Manipulation Risks**
When an oven is liquidated, the pool receives XTZ which it trades for kUSD on Quipuswap. Sophisticated users with significant capital may be able to manipulate prices on Quipuswap prior to using the pool to liquidate an oven in order to gain outsized profit.
Any market is subject to manipulation, and there is no way to mitigate this risk completely. However, healthy and liquid markets generally prove harder to manipulate. Thus, the less healthy the Quipuswap market for kUSD is, the more risky the Liquidity Pool contracts behavior will be.
