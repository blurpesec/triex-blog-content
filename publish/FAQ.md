---
title: FAQ for Trinary Exchange - EVE Frontier Market
excerpt: How trading works in EVE Frontier - buy and sell orders, player-run trade hubs, asset safety, and CRED - answered for the Trinary Exchange marketplace.
tags:
  - faq
  - marketplace
  - trading
date: April 28th, 2026
author: Hecate
image: /trilith.svg
---
## What is Trinary Exchange (TriEx)?
Trinary Exchange is a protocol for running player-driven item marketplaces in EVE Frontier.

It allows players to buy and sell items at [local trade hubs](/marketplace/discovery) while keeping control of their own items and currency at all times.

---

## Is TriEx an in-game tribe or organization?
No.
TriEx is neutral infrastructure - not a tribe, faction, or gated group.

Anyone can use it, and anyone can operate a trading hub.

---

## How do I set up a trade hub in EVE Frontier?
Point your storage unit to `https://trinary.exchange` and follow the directions to get started.

**OR:**
You can watch this short (2.5-minute) tutorial:
<iframe width="560" height="315" src="https://www.youtube.com/embed/d177AMwPkw8"  
title="YouTube video player" frameborder="0"  
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"  
allowfullscreen></iframe>

---

## Do I need to trust the exchange operator?
Minimally.

The operator provides the interface and runs the TriEx trade hub, but **they cannot access or withdraw your items or currency without destroying their storage unit running TriEx**.

**That said:** Exchange operators can un-anchor their structures, and the items within will be dropped into space.

---

## What happens if a TriEx hub is destroyed while I have buy or sell orders listed?
If the storage unit hosting TriEx is destroyed, some of the items at the trade hub will likely drop to the attackers.

* **Sell orders** become null and void since the items backing them no longer exist and cannot be traded.
* **Buy orders** can still be canceled by their owner, allowing you to reclaim your currency from the wreckage of a destroyed trade hub.

---

## What happens to my items when I list them for sale?
Your items are either:

* In your possession (when not listed), or
* Locked while an order is active

They can only move if:

* The order is completed, or
* You cancel it

No one else can move them.

---

## Is Trinary Exchange safe?
Your items stay in the Storage Unit at the trade hub, and only your own wallet can move them or your CRED: the exchange's contracts give neither the hub's owner nor Trinary Exchange a way to withdraw them.

* **Items** stay in the hub's Storage Unit the whole time. While a sell order is open they are locked to that order, and they only move when the order fills or you cancel it.
* **CRED** for an open buy order is locked in the on-chain market until the order fills or you cancel it. The rest sits in your wallet or your trading account, which only your wallet can withdraw from.
* **Every action** is a transaction you approve in your own wallet.

What you do trust Trinary Exchange with is the code: the operator can upgrade the exchange's smart contracts. The [capabilities list](https://github.com/loash-industries/trinary-exchange/blob/main/CAPABILITIES.md) sets out exactly what the operator can and cannot do.

The other risks are in the game and on the web: if a hub's Storage Unit is destroyed or un-anchored, the items inside are lost or dropped into space, and phishing sites can imitate trinary.exchange, so check the address before you sign.

---

## Do I need SUI for transaction fees?
**No** - we currently pay those transaction fees for you.

---

## What is CRED?
CRED (Inter-Galactic Credits) is the currency Trinary Exchange prices and settles EVE Frontier trades in: a token on the Sui testnet that Trinary Exchange created and operates.

You get CRED by selling items, by swapping EVE or SUI for it on [Currencies](/currencies), or from the one-time starter claim on the trade screen. It has 6 decimal places and carries no voting or staking rights. Full explainer: [What is CRED in EVE Frontier?](/guides/what-is-cred-in-eve-frontier)

In the future, trade hub operators (individual players or player groups) may create and operate their own currencies to add more financial depth to the game.

We intend for player-operated currencies to be integrated into TriEx in a trustless way. This means the protocol does not prevent players from creating or using their own currencies, and no permission from TriEx is required.

---

## How is Trinary different from other EVE Frontier markets?
Trinary Exchange prices every item in CRED, pays the Sui network fee for your trades, and connects many player-owned trade hubs into one place where you can compare prices and distances.

On top of the market it adds tribe organizations with shared storage and treasuries, currency markets for EVE and SUI, a hauling job board, and a developer API. EF-Map's Open Market, for example, settles in EVE and has traders pay their own gas in SUI. See the [side-by-side comparison](/guides/trinary-exchange-vs-ef-map-open-market).

---

## Can someone steal my assets through the exchange?
The system is designed to prevent that.

There is no shared custody and no administrative access that would allow an operator to withdraw user assets.

---

## What happens if the website or interface goes offline?
Your assets remain safe.

The interface is only a way to interact with the system - your orders and items continue to exist independently and can be recovered. To recover assets if TriEx goes offline, players will need to use or build tools that interact directly with the on-chain contracts.

---

## Do I need a specific crypto wallet?

* If you want to trade in-game: **No**, just use the storage unit interaction window.
* If you want to trade or manage trade hubs in Chrome or Brave: you will need to install the CCP Games EVE Vault Chrome extension and log in with your EVE Frontier credentials.

---

## Where do trades take place?
Trades are executed on-chain on Sui.

This ensures transparency and that all trades follow predefined rules.

---

## Why not just trade directly with other players?
Direct trading works, but has limitations:

* Both players must be available at the same time
* It relies on trust or manual coordination
* It does not scale well into a broader market

TriEx enables asynchronous trading - you can list items and others can fill those orders later.

---

## Can anyone create a TriEx hub?
Yes.
It is fully permissionless - any player, [tribe, or organization](/features/organizations) can set up and operate a trading hub.

---

## Is there a single global market?
No.
Markets are local to the storage unit where the TriEx trade hub is deployed and reflect local conditions such as supply, demand, and logistics.

However, you **can see** buy and sell orders at nearby public trade hubs. This allows you to [identify better prices elsewhere](/items) and supports transportation as a viable in-game career path.

---

## Can players manipulate prices?
Prices are determined by market activity.

While players can influence prices through trading, they **cannot access or interfere with other players’ assets or orders**.

---

## Can TriEx be used for more than trading?
Yes.
It can serve as a foundation for additional systems that require item trading primitives such as:

* Loans
* Contracts
* [Tribe supply chains](/features/shared-storage)
* Logistics coordination

---

## Will the code be open source and publicly auditable?
Yes.

It is not currently open source, but the intent is to release the contracts, interface, and some developer tooling so players can build their own trading interfaces on top of TriEx.

---

## Are there any tradeoffs?
Yes:

* Early markets may have limited liquidity
* Each hub builds its own local activity
* Users must still follow standard online safety practices

However, the key benefit remains: **you do not need to trust anyone else with your assets**.
