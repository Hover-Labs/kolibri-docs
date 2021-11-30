## Oracle Contract

The `Oracle` contract provides price data to the Kolibri ecosystem. It is implemented using a [Harbinger Price Feed](#/project-info/oracle/harbinger).

The `Oracle` contract calls to a `Harbinger Normalizer` contract. It sanity checks the return values, normalizes them for the Kolibri system, and returns them to the client. 

The current implementation uses a Coinbase Data Feed normalized by volume weighted average price (VWAP) with `n=6`.

The `Oracle` performs the following checks on data returned from Harbinger:
- Was the asset returned "XTZ-USD"
- Was the asset updated in the last 30 minutes

### Callbacks

`Oracle` has a single callback function that returns the price to the caller. The caller includes a callback contract/entrypoint in the request and data is called back to that location.

The callback is a single `nat` which is the XTZ-USD price, specified with 10^-18 precision.

### ACL Checking

Anyone may request a price via the `getXtzUsdPrice` entrypoint.

However, only the `Harbinger` contract can return price data via the `getXtzUsdPrice_callback` entrypoint.

### Upgrade Path

Any contract that points to the `Oracle` should have a governable `address` for the `Oracle`. In case of upgrades, a new Oracle could be deployed, and governance could upgrade contracts to point to the new contract.

### State Machine

`Oracle` maintains a simple state machine with two states: (1) `IDLE` and (2) `WAITING_FOR_HARBINGER`. 

From the `IDLE` state, the `Oracle` can only transition to the `WAITING_FOR_HARBINGER` state. The `getXtzUsdPrice` can only be invoked when in the `IDLE` state. Invoking the entrypoint transitions the `Oracle` to the `WAITING_FOR_HARBINGER` state. In the `IDLE` state, `clientCallback` is always `none`.

From the `WAITING_FOR_HARBINGER` state, the `Oracle` can only transition to the `IDLE` state. The `getXtzUsdPrice_callback` can only be invoked when in the `WAITING_FOR_HARBINGER` state. Invoking the entrypoint transitions the `Oracle` to the `IDLE` state.  In the `WAITING_FOR_HARBINGER` state, `clientCallback` is always `some`.

## Storage

The `Oracle` stores the following:
- `harbingerContractAddress` (`address`): The address of the `Harbinger Price feed` contract. Used to request data and for ACL checking.
- `state` (`nat`): The current state of the contract; see 'State Machine' above.
- `clientCallback` (`option(contract(nat))`): A register that stores state when the `Oracle` is waiting for Harbinger data.
- `governorContractAddress` (`address`): The address of the `Governor` contract. Used for ACL checking.
- `maxDataDelaySec` (`nat`): The maximum acceptable age of data received from Harbinger, in seconds.

`governorContractAddress` and `maxDataDelaySec` are governable.

## Entrypoints

The `Oracle` has the following entrypoints:
- `getXtzUsdPrice`: Retrieve, sanity check, and normalize Harbinger data.
- `getXtzUsdPrice_callback`: Callback to receive data from Harbinger. This entrypoint may only be called by the `harbingerContractAddress.`
- `default`: No-op. Always fails to prevent transfer of XTZ into contract.
- `setMaxDataDelaySec`: Set the maximum acceptable age of data recieved from Harbinger. May only be called by the `Governor`.
- `setGovernorContract`: Set the `Governor` address. May only be called by the `Governor`.

## Possible Future Work

A future version of `Oracle` may choose to use a custom view on the `Harbinger` data, or another oracle solution.

If a new version of the `Oracle` is required to be deployed to accomodate updates to `Harbinger`, then `harbingerContractAddress` could become governable.
