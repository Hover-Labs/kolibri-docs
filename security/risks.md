# Kolibri Risks
All smart contracts deployed (as with all software) run the risk of software bugs, some of which can compromise the overall security of a system. When systems have administrative controls there is also the risk of malicious actors abusing those controls for nefarious purposes.

Kolibri is no different, and has a number of risks associated with it. We’ve tried to keep things as simple as possible, had a security review done by a reputable company, and written extensive tests (unit + integration) to gain confidence that Kolibri operates as we expect it to, but with security there is no guarantee that it’s 100% safe to use.  

Below we outline some of the risks that exist in the system. It’s an ever-growing document and likely not 100% inclusive of every risk within the system. If you think something is missing please ping us on [twitter](https://twitter.com/hovereng) or come [join our discord](https://discord.gg/pCKVNTw6Pf)!

### Protocol-level Risks
Tezos is an [ever-evolving protocol](https://medium.com/tezoscommons/tezos-blockchains-fast-pace-of-evolution-delivers-new-features-with-edo-upgrade-fec6a62a4b8b), which brings its own risks with it. Writing a fault tolerant and secure blockchain is no simple task, and where there is complexity there is defects. 

With these upgrades, the protocol can do or change literally anything about how the system operates. This is a feature not a bug, but if the protocol designers break assumptions it may lead to a loss of security for the system (imagine a scenario where the address of the current caller suddenly can be set arbitrarily somehow, that’d break our authorization checks immediately).

Likewise, there have been real scenarios where the Nomadic Labs protocol development team has uncovered [unexpected things with new features](https://forum.tezosagora.org/t/baking-accounts-proposal-contains-unexpected-breaking-changes/2844) which would break parts of Kolibri. We try to be in constant contact with them to hedge against this risk, but if things like this baking account feature that has these unexpected consequences were to get elected and go live, it would cause issues in the system.

### Economic Risks
Until the stability fund grows in size, there’s a scenario where the economic model breaks if a black swan event were to occur within the Tezos ecosystem and the price crashes significantly. We have a very conservative 200% collateralization ratio per oven, and use a VWAP price feed to hedge against these risks but if the underlying collateral that backs the outstanding kUSD were to suddenly (and continue to be) worth significantly less, it’d remove the economic incentive to liquidate ovens and destroy any ability to stay at peg. In this black swan event we’d need to use the stability fund as a liquidator of last resort, but that will only be successful if we have enough funds to liquidate enough ovens to restore stability.

### Liquidation Risk
If your oven becomes under-collateralized then it’s vulnerable to liquidation. When liquidated, you keep all existing kUSD, but your collateral (the XTZ in your oven) can be “bought” by closing out your loan, which helps ensure that the system is always fully collateralized (there is more value in the system than created in kUSD value). 

We use a VWAP price feed ([Harbinger Oracle](https://github.com/tacoinfra/harbinger#harbinger)), so we have some protections against a sudden flash-crash style drop in the price of XTZ (a flash crash would need to be sustained for 6 Harbinger updates, which is *at minimum* 6 minutes if someone is submitting a new price update every block). In practice the price oracle is updated every 15 mins or so currently so it’d take ~1.5 hours for it to feel 100% of the effects of this black swan event (note: anyone can push oracle updates using https://harbinger.live).

### Risks In The Tooling
Kolibri was built and compiled to Michelson (the native smart contracting language of Tezos) using [SmartPy](https://smartpy.io/). SmartPy is currently [open source](https://gitlab.com/SmartPy/smartpy) (though not with the most recent versions hosted on GitHub/gitlab), but is pretty constantly changing and has not (as of writing this) undergone a formal security audit. 

We’ve written extensive tests, but they all depend on SmartPy operating correctly, and if it’s doing something like optimizing out security checks or creating invalid Michelson, it’s possible that those issues could impact the security of the system as a whole. Writing compliers is difficult to do and one of the assumptions we’ve had to make about this system is that SmartPy is operating correctly and would also inform us of any major breakage they discover. We’re also in contact with them to hopefully help hedge against this risk, but this is largely an unpredictable factor.

### Risk Of Software Bugs/Logic Issues
Kolibri is software not unlike any other software, and may have hidden bugs or defects that may impact the security of the system. We’ve taken every precaution we could to try to make things as simple and straightforward as possible, and [gotten a security review done from a reputable firm in the space](https://kolibri.finance/project-info/security/security-audit), but there may still be bugs that exist that break assumptions about how these smart contracts work or operate. 

Our tests also give us some assurances that things are working as we expect them to, but if our tests are operating on false premises (specifically that they’re running in the same way as the contracts run on-chain), then our tests could be giving us improper assurances. 

### Risk Of Malicious Operators
We’re working hard to remove ourselves as arbiters of the system, but until we do things are controlled by a multi-sig contract (“the governor”) that has an 8 hour time-lock. We currently hold signaling votes on our Discord for people to be able to participate in governance over any parameter changes, but if 2 of the multi sig keys were compromised it’d allow an arbitrary actor to change parameters of the system. 

You can find the current multi-sig contract [here](https://better-call.dev/mainnet/KT1JBmbYxTv3xptk2CadgEdMfjUCUXKEfe5u/storage), and can see any parameter changes queued in the time lock [here](https://better-call.dev/mainnet/big_map/421/keys). If you see a change there that’s not expected, please come to [the discord](https://discord.gg/pCKVNTw6Pf), [tweet at us](https://twitter.com/hovereng), or email security@hover.engineering to let us know. 

Once we have a governance solution deployed and working in production this risk is reduced significantly, but also traded for other risks like a coordinated and hostile takeover of the system. 

## FAQ
**Is storing funds in a Kolibri oven just as secure as storing it on a ledger?**

**Answer**: No, absolutely not. Smart contracts *all* have risks to them, and storing funds in a Kolibri oven is much more complicated and exposes your XTZ to many more risks than just holding it on a ledger. We’ve taken as many reasonable precautions as we felt was reasonable but most of the risks above don’t exist with just holding funds on a Ledger. Put bluntly, you are increasing your risks to your funds 