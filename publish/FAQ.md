---
title: FAQ
excerpt: Frequently Asked Questions about Trinary Exchange - an item marketplace protocol for EVE Frontier
tags:
  - faq
date: January 25th, 2026
author: Hecate
image: /foam_block.svg
---
## What is it?
Trinary Exchange (TriEx) is a distributed exchange protocol that allows for tribes, syndicates, or players within Eve Frontier to spin up their own trading hubs by exposing a low-level trading system on-chain that is trust-less by design. The Trinary Exchange service and the operators of trade hubs will have no ability to take player items or currency; they will only have the ability to facilitate trades initiated by players. 

### Is the Trinary Exchange an in-game tribe?
No. TriEx aims to be a neutral service that builds trading infrastructure for other tribes to use. There will be no gating to deny anyone the ability to become a trading hub service operator, nor on building second-order services on top of it (tribe mission systems, loan systems, etc).

## Is it safe?
Generally, yes - though this is a bit of a complex question to answer, so we'll break it down into different types of risk (these aren't all-encompassing, but should give a good indication of how we're thinking about this).
### Item ownership
While trading within TriEx, items and currency will be either owned by the player (if they're not actively being traded) or by a vault module that custodies assets (if they're are part of an open order). The vault module has no ability to "withdraw" other than through the mechanism of trade execution or order cancellation. At no point in time will the owner of an SSU have access to your player-owned assets, nor the module-"owned" assets. Neither will we as the operators of the exchange. The modules, while managed by TriEx, are "trust-less" by design. This means that they operate fully within the bounds of the module code which defines the behavior of the trading system.

### On-chain hack risk
The module design of the Trinary Exchange is as a fork (a copy) of [DeepBook v3](https://docs.sui.io/standards/deepbook) with some functionality stripped out. The on-chain code here is (as of November 8th, 2025) custodying assets in excess of $10m and has been for more than 1-year. That's not to say that the TriEx contracts have the same security, but they do inherit some of that security by nature of being almost exactly the same implementation and design. 

DeepBook v3 has undergone a security audit in August 2024 which you can read [here](https://github.com/sui-foundation/security-audits/blob/main/docs/mysten_deepbook_audit_final.pdf) . If the Trinary Exchange gains significant traction, we can pursue our own audit of our slightly-altered version of the contract code.

### Trading interface hack risk
Since transactions are created through software running on a website, there is a risk of various types of hacks associated with the trading interface. Software supply chain hacks, DNS spoofing, phishing, etc are all forms of hacks that affect the trading interface. There is nothing that can be done to fully alleviate the risk of hack, but standardizing solid security practices can be used to reduce the risk drastically. A few of these mechanisms that we'll be pursuing are:
1. Package version pinning
2. Automated vulnerability detection
3. DNSSEC, and HSTS
4. Bring-your-own-wallet only - We never handle your private keys. All transactions must be signed through your existing Sui wallet (like Slush SUI Wallet, Nightly wallet, etc.), leveraging their established security models rather than implementing our own wallet solution.

In addition - we'll be looking into the viability of periodically publishing a static (client-side) content-addressed version of the trading website using IPFS for software supply-chain determinism.

## Will the on-chain orderbook code be open sourced?
Yes.

## **When can I start using Trinary Exchange?**
We're aiming to have a beta using mocked on-chain data sometime in Feb/March 2026. As for main game data - it will likely be some time after CCP has finished migrating the underlying state to SUI (currently targeting March, 2026).

