# Kolibri Savings Rate Contract

## Overview

The Kolibri Savings Rate (KSR) is a component of the Kolibri protocol that allows users to lock up kUSD and receive a portion of the protocol fees accrued through the stability fee, similar to the [DAI Savings Rate](https://makerdao.world/en/learn/Dai/dsr/)

Users deposit kUSD into the KSR contract, and are given KSR tokens that track their deposits. Interest is then accrued at a variable rate compounded every minute. The variable rate is set by the Kolibri DAO and should never exceed the stability fund.

## How It Works

As the protocol accrues stability fees, they're accumulated in the [stability fund](stability-fund) and [developer fund](developer-fund). The KSR contract is white-listed in the [stability fund](stability-fund) to be able to request funds to fill the pool. 

At every deposit/withdraw, a global interest accumulator is updated and the required interest to fill the pool is requsted from the [stability fund](stability-fund) and deposited into the pool. 

## Benefits

- **kUSD Contributors**: Users depositing kUSD can get a variable rate of return (paid in kUSD) automatically without any risk of liquidation/impermanent loss. 
- **Kolibri Protocol**: The protocol gains a new lever to help control the **demand** for kUSD, instead of relying solely on supply-side economics to maintain peg. 

## Risks

**Smart Contract Risk**
All smart contract based systems carry smart contract risk, the risk that smart contracts do not work as intended. Please see the  [risks page](../security/risks) as part of the Kolibri documentation for more info about the general risks with using smart contracts on Tezos. There is no way to mitigate smart contract risk completely, however, good software engineering practices can help to build confidence in the security and correctness of implementation. The Kolibi Liquidity Pool contracts are [open sourced](https://github.com/Hover-Labs/kolibri-contracts/blob/master/smart_contracts/savings-pool.py) and well tested, however, they **have not yet undergone a security audit**. A security review is slated for March, 2022 to review these contracts. 
