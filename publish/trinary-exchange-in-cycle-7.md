---
title: "Trinary Exchange in Cycle 7: Hub Revenue, Fee Tiers, and Shared Intel"
excerpt: What's new on Trinary Exchange for EVE Frontier Cycle 7 - trade hub operators earn a share of fees, frequent traders pay less, org storage gets audit logs, and orgs can share locations with allies.
tags:
  - announcement
  - marketplace
  - trade-hubs
  - organizations
  - fees
date: October 11th, 2026
author: Hecate
image: /cycle-7-hub-page.png
---
## What's New on Trinary Exchange in Cycle 7

**Cycle 7 is live, and the marketplace is now worth more to the players who keep it running.**

---

Every cycle we take what players told us about the last one and build on it. Cycle 6 showed us three things. The players running trade hubs were doing real work with no direct reward. The most active traders were paying the same rate as someone placing their first order. And organizations were running serious logistics with nothing but tribe chat to track who moved what.

Cycle 7 fixes all three. Here's what changed.

## Run a Trade Hub, Earn a Cut

Every trade hub on Trinary Exchange is a player-owned storage unit. Somebody anchored it, keeps it fueled and online, and hauls stock in and out. Starting in Cycle 7, **that operator earns a share of the exchange's fee revenue from every item market at their hub - 20% at launch.**

The important part: **this never changes what traders pay.** The operator share comes out of the exchange's side of the fee, not out of anyone's trade. Traders see the same rates at every hub, so the only competition between hubs is on location, stock, and price.

![A trade hub's Settings & Treasury tab, showing an unclaimed operator share of 403,422 CRED across 11 markets and the payout address](/cycle-7-operator-share.png)

How it works:

- **Set a payout address once** - From your hub's page, open **Settings & Treasury** and register a beneficiary. It covers every item market at your hub, including ones created later. It doesn't have to be you: point it at a partner or your organization's treasury and the hub pays for the whole tribe.
- **Nothing is lost while you set up** - Fees earned before you register a payout address build up and wait for you. Your first claim pays out everything owed.
- **Claim whenever you want** - Hit **Claim operator share** and your cut goes to your payout address. Anyone can trigger a claim, but the payment always goes to the address you registered.

Want the details on what's shared and when? Read [how operator fees work](https://docs.trinary.exchange/docs/concepts/operator-fees?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7).

## Trade More, Pay Less

Cycle 7 introduces **fee tiers**. The more you trade, the lower your rates. Every trading account sits on an eight-step ladder, and we've named the steps by mass. Everyone starts as **Dust**. Keep trading and you'll grow into **Rubble**, **Moonlet**, **World**, **Giant**, **Star**, **Cluster**, and finally **Galaxy**.

![The account menu for an organization trading account at fee tier Class III, World: 1.83% taker and 1.49% maker, about 40.6M CRED of trading away from Giant](/cycle-7-fee-tier.png)

| Tier | Fees paid in the last 30 days | Taker | Maker |
|---|---|---|---|
| 0 · Dust | 0 CRED | 2.20% | 1.80% |
| I · Rubble | 20,000 CRED | 2.09% | 1.71% |
| II · Moonlet | 100,000 CRED | 1.98% | 1.62% |
| III · World | 500,000 CRED | 1.83% | 1.49% |
| IV · Giant | 2,000,000 CRED | 1.65% | 1.35% |
| V · Star | 10,000,000 CRED | 1.43% | 1.17% |
| VI · Cluster | 50,000,000 CRED | 1.21% | 0.99% |
| VII · Galaxy | 200,000,000 CRED | 1.10% | 0.90% |

_Rates for item markets at launch._

A few things worth knowing:

- **Both sides of a trade now pay a fee.** In earlier cycles only buyers paid. Now sellers do too, with the fee taken out of their proceeds, so nothing extra gets locked up when you list.
- **Makers always pay less.** If you post an order and let it sit on the book, you pay the lower maker rate. Resting orders are what give everyone else something to trade against.
- **One tier across every market.** Fees you pay in any CRED market count toward your tier in all of them. Buy fuel at one hub, sell ore at another, and both push you up the ladder.
- **Your tier lives on your trading account.** That includes your organization's. When officers trade through the org's account, they all build the same tier together.
- **Use it or lose it.** Your tier is based on the last ~30 days. Stop trading and it steps back down as old fees age out.

Click your name in the top-right corner to see your current tier and how far you are from the next one. The full breakdown, including what happens when you cancel a buy order, is in [the fees docs](https://docs.trinary.exchange/docs/concepts/fees?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7).

## Audit Logs for Shared Storage

[Shared storage](https://trinary.exchange/features/shared-storage?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7) only works if people trust it. When fifty players can reach into the same vault, "who took the Kerogen Tar?" becomes an organizational problem.

Cycle 7 adds **Audit Logs** to your inventory. Open **Inventory**, expand your organization in the sidebar, and pick **Audit Logs**. You'll see every item that moved into, out of, or between your org's vaults, at every hub and every tier.

![The Algorithmic Warfare audit log: 68 item movements across 13 items and 4 members, each showing item, direction, quantity, source and destination storage, and the member who moved it](/cycle-7-audit-logs.png)

Each row tells you:

- **What moved, and how much**
- **Where it came from and where it went** - org vault, trading account, or a member's hangar, with the tier badge (Officers, Members, and so on) on each side
- **Who did it** - the member who signed the transaction, plus a link to the transaction itself

Filter by vault, by item, or by direction - **In**, **Out**, or **Internal** for shuffles between your own vaults. Flip on **My actions only** to check your own history. Your personal storage gets the same treatment: an Audit Logs view under your Hangar, Warehouse, and Trading Account.

No more digging through tribe chat to reconstruct who withdrew what. It's all on-chain, and now it's readable too.

## Share Locations Across Organizations

[Private location sharing](https://trinary.exchange/posts/build-in-the-shadows?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7) let you hand your hub coordinates to individual players. In Cycle 7, **location lists can belong to an organization, and you can share them with individual players or with a role in any organization - yours or someone else's.**

![A location list's Permissions tab: Algorithmic Warfare Officers hold Grant, Algorithmic Warfare Members can View, and the Rift Riders Member role is being added with View access](/cycle-7-location-sharing.png)

Open **Org Locations** from the sidebar, pick a list, and go to the **Permissions** tab. Every entity with access shows up on one screen, with three levels of permission:

- **Grant** - can manage who has access
- **View** - can decrypt and read the locations
- **Edit** - can add and update locations

Hit **Edit permissions**, search for a player or an organization, and pick who gets in - a single player, or another org's Members, Officers, or Admins. Stage as many changes as you need. They all go out together in a single transaction. Removing someone works the same way.

This is where it gets interesting. Some of the things you can now do:

- **Joint operations** - Two organizations working toward the same goal can share a list of staging points, fuel caches, and forward hubs without merging their member rosters.
- **Private trading networks** - Share your hub locations with a trusted partner org's Members role. Their traders can find your hubs and trade at them. Everyone else sees nothing.
- **Need-to-know logistics** - Give your Officers a list with your reserve depots and give Members the list with the spoke hubs. The [hub-and-spoke model](https://trinary.exchange/posts/trading-in-the-fog-of-war?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7) is now a few clicks.

Locations are still end-to-end encrypted with [Keyspace](https://github.com/loash-industries/keyspace). TriEx servers never see your coordinates, and when you revoke access, it's revoked. If your account can see a lot of lists and you'd rather not sign a decrypt prompt on every page load, you can turn off **Auto-decrypt org locations** in **Settings** and decrypt on demand instead.

Step-by-step instructions are in [the location sharing guide](https://docs.trinary.exchange/docs/how-to/share-a-location?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7).

## Every Trade Hub Gets Its Own Page

Before you haul cargo across the cluster, you want to know if the trip is worth it. Every trade hub now has **its own page**, and you don't need to log in to see it.

![The Eimur Expedition Outfitter trade hub page: 58.7M CRED of volume and 37 trades in the last 7 days, 26 items with orders, and its top items by volume](/cycle-7-hub-page.png)

At a glance you get:

- **7-day volume and trade count** - how busy the hub actually is
- **Items with orders** - how many markets are open there right now
- **Last activity** - whether anyone has traded there recently
- **Top items by volume** - what the hub is really for, whether that's fuel, ore, or Building Foam
- **The latest trades** - with a link through to the hub's full trade history
- **Structure details** - the storage unit behind the hub

Each page has a permanent link, so hub operators can post it in their tribe's Discord or put it on a recruiting post. When someone likes what they see, **Trade at this hub** takes them straight to the market. For owners, the page is also where you manage your operator share.

Find any hub from the **Trade Hubs** list and click the info icon next to it.

## Get Started

Everything above is live on [trinary.exchange](https://trinary.exchange?utm_source=blog__cycle_7_announcement&utm_campaign=cycle_7) now.

1. **Run a hub?** Open your hub's page, go to **Settings & Treasury**, and register a payout address. Then claim what you've earned.
2. **Check your tier** - Click your name in the top-right corner and see where you sit between Dust and Galaxy.
3. **Running an org?** Open **Inventory**, expand your organization, and check **Audit Logs**. Then head to **Org Locations** and decide who your allies are.

_Have feedback on Cycle 7, or want to show off your hub? Join the conversation in our [Discord community](https://discord.gg/VCmR6xrxmj)._

---

_Trinary Exchange | Rebuild Civilization Through Trade_
