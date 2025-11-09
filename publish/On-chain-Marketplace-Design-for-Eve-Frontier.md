---
title: On-chain Marketplace Design
excerpt: Outlining the structure of on-chain marketplaces
tags:
  - ecosystem
  - onchain-markets
  - decentralized-exchanges
date: November 9th, 2025
author: Hecate
image: /currencies.svg
---
# 1. Introduction

## 1.1. Context & Motivation

In _Eve Frontier_, players operate in a vast, abandoned galaxy where intelligent life has vanished. Traditional in-game trading infrastructure is scarce, especially in remote systems. As a result, players exploring the frontier often find themselves without access to marketplaces, making coordination difficult and forcing them to transport goods back to distant population centers.

On-chain marketplaces offer a solution by making trade infrastructure **universally accessible**. By replacing NPC trading hubs with smart contract-powered structures, players can establish distributed, self-sustaining markets anywhere in the galaxy.

Even more critically, on-chain systems enable real-time _detection of demand_. Since all order flow is published on a global, transparent ledger, players can observe where demand is emerging across the entire universe - thus facilitating a marketplace for _transportation_. This stream of information allows more efficient routing of goods, better pricing, and strategic arbitrage, even in regions with no NPC market to anchor to.

## 1.2. Goals of This Post

In making this post, I intend to outline a technically-feasible approach to enabling on-chain marketplaces in Eve Frontier.

# 2. Background & Core Concepts

## 2.1. What “On-Chain” Means

In the context of Eve Frontier “on-chain” means "existing on a blockchain". Specifically - this is referring to assets (currently - indivisible digital representations denoting your character, anchored smart assemblies, in-game items stored within those assemblies, and various object metadata like "character's corporation id"). A **blockchain** is a data structure combined with economic incentives that lets parties who don’t trust one another interact under a common set of rules. Those rules, enforced by code and cryptography, ensure the system’s state is transparent and verifiable.

## 2.2. Key Marketplace Primitives

On-chain marketplaces, whether for real-world assets or in-game goods, revolve around a few fundamental primitives. These serve as the building blocks of any trading system, regardless of whether it’s centralized or decentralized.

### Listings (Limit vs. Market Orders)

A _listing_ is the expression of intent to trade. In the most common designs of marketplaces, this takes the form of a _limit order_ - an offer to buy or sell a given asset at a specified price. In contrast, _market orders_ execute immediately at the best available price against existing limit orders. In Eve Frontier, listings must encode location in addition to price and quantity. Each listing is (in it’s current form) a message to the rest of the galaxy: “Here is what I have, and what I want for it.” CCP has signaled that this might change in the future by way of [PODs](https://pod.org/).

### Settlement & Custody

Settlement is the process of finalizing a trade by transferring custody of assets between two parties. On-chain, settlement can be **atomic** - meaning both assets are exchanged in a single transaction, or not at all. Blockchain systems prevent partial completion, ensuring trades are either fully executed or fully reverted.

**Custody** refers to where assets are held before and during the trade. In _Eve Frontier_, custody is managed by structures called **smart assemblies**. Currently, this is only possible within a **Smart Storage Unit (SSU)**.

To perform an atomic trade within an SSU, both the buyer and seller must first deposit their in-game assets into the SSU. This transfers ownership of the assets to the SSU itself, allowing its associated logic - referred to as a **system** - to execute the transaction securely and atomically.

### Discovery & Indexing

For players to interact with trade offers, they must be able to _find_ them. Discovery is how listings become visible to buyers, and indexing is how they are organized and retrieved efficiently. On-chain discovery is non-trivial - but there are a couple of approaches that can be taken:
- Smart contracts can **emit events** for off-chain indexers to consume, but true in-game visibility requires. This approach is more “gas efficient” (cheaper to execute on the blockchain due no consumption of state - the most expensive part of a blockchain transaction) at the expense of being more expensive to operate for the marketplace operator.
- Smart contracts can **write state** directly to the chain within the atomic transaction. This approach is less “gas efficient” (more expensive for users), but doesn’t require the market operator to run third party indexer.

## 2.3. Design Constraints in Games

In traditional games, trade actions are instant. The act of listing, browsing, or buying an item takes milliseconds. On-chain actions introduce latency: transaction signing, propagation, and confirmation introduce delays of up to several seconds. This latency must be masked or compensated for through clever UX patterns, such as signed intents executed in batch by game servers (so-called "meta-transactions").

Additionally, interactions can fail because transactions are executed in sequence - if another user's transaction is processed just before the current user receives an update, it can invalidate their action after it is originated.
# 3. Approach A: Order-Book Based DEX Model

## 3.1. Architecture Overview

The order-book model mirrors traditional financial exchanges, where buyers and sellers post limit orders specifying the asset they want to trade, the amount, and the price. In an on-chain context, this model can be implemented entirely in smart contracts, creating a decentralized exchange (DEX) where the marketplace logic is enforced by code rather than a central operator.

At its core, the architecture consists of:

- **Order Creation:** Traders submit limit orders to a smart contract by calling a function on an `Orderbook` contract. Each order must include the location of the assets (smart object id), the item id, the price, order type (buy/sell limit/market), the quantity to buy/sell, and the character id of the order owner.
- **Order Storage:** Orders are recorded in a structured on-chain data store (in Eve Frontier's current iteration - it would be a [MUD Table](https://mud.dev/introduction)). Orders should be sorted by price to keep gas fees minimized (more computation causes higher gas fees.
- **Matching Engine:** Order matching can happen fully on-chain via a matching algorithm baked into the contract. Orders are stored pre-sorted to reduce computational complexity.
- **Trade Settlement:** When a match occurs, the smart contract executes the trade atomically - transferring ownership of both assets between buyer and seller. Typically, the assets are held in escrow by the Smart Storage Unit until the transaction is complete.
- **Cancellation & Expiry:** Users can cancel their orders manually, and smart contracts can be configured to support order expiry based on a order creation timestamp field to reduce gas bloat and stale entries.

This model gives players full control over their trading intent - including setting precise prices and order parameters. However, the cost of storing and updating on-chain state, especially for a large order book, can become an issue if this exists on a more expensive blockchain. The blockchain that EveFrontier exists on shouldn't have this issue.

## 3.2. Workflow

1. User A deposits _CommodityX_ into the SSU manually.
2. User A creates a transaction to list a sell order for _CommodityX_ at _PriceY_ and submits to the chain
3. User B gets and update on their trading interface as soon as transaction submitted in #2 is included on-chain showing _CommodityX_ for sale.
4. User B creates a transaction (_TransactionB_) to execute a market order for _CommodityX_ with a maximum execution price of _PriceY_ * 1.01 (1% price slippage) and submits to the chain.
5. _TransactionB_ executes atomically on-chain - transferring _PriceY_ to UserA and _CommidityX_ to User B.

## 3.3. Pros & Cons

### Pros

- Familiar trading UX
- More precise control over pricing and order execution
### Cons

- On-chain gas costs are higher (even up to 100% higher than AMMs. This might be a non-issue due to Eve Frontier operating on a L2 blockchain with extremely low fees. At current transaction fee rates - an orderbook transaction execution would cost ~$0.00003015 worth of fees).
- Front-running risk (may be alleviated with).
- Bad order execution (can be somewhat alleviated with a configured maximum execution price).

# 4. Approach B: Automated Market Maker (AMM) Model

## 4.1. Architecture Overview

**Automated Market Makers (AMMs)** are a concept [introduced by Vitalik Buterin, the founder of Ethereum, in 2016](https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way/).

The core idea is simple:  
Users deposit assets into a shared pool (this is called "liquidity provision"), and other users can trade against that pool. Prices are determined algorithmically, without relying on traditional order books or limit orders.

Instead of matching buyers and sellers directly, AMMs use a **pricing formula**. The more you trade against the pool, the more the price moves - this is called **price slippage**. The assets in the pool follow a curve-based pricing model (e.g., constant product: _x * y = k_). So, while liquidity may appear "infinite," large trades will receive worse prices if the pool isn’t deep enough.

AMMs have become popular in blockchain ecosystems because:

- **They offer a smoother user experience** - especially for transaction execution. This is critical in blockchains, where front-running is a common issue due to latency of blockchain networks.
- **They are gas-efficient** - meaning they typically have lower transaction fees compared to traditional on-chain order book systems.

## 4.2. Workflow
### Traditional

1. User supplies asset & currency liquidity
2. Buyers swap assets via AMM formula (e.g. x·y=k)
3. Liquidity fees accrue to providers

### Constraints in Eve Frontier

While AMMs work well for divisible assets like tokens, _Eve Frontier_ presents unique challenges:
#### Issue 1: Lack of Divisibility

Most blockchain tokens (e.g., ETH) are divisible up to 18 decimal places (e.g., wei = 1e18 units of ETH). This makes fractional transactions smooth and intuitive:

> Buying 1 ETH might give you anywhere between 0.995 and 1.005 ETH 0 close enough, and no one minds.

But in _Eve Frontier_, many assets - like ships - are **not divisible**:

> You can’t meaningfully own 30% of a ship.

This undermines the very foundation of AMMs, which rely on assets being divisible to calculate smooth price curves. In order to make an AMM that functions for Eve Frontier, it's likely that the algorithm would need to be altered to function better as a price ladder - a sort of step function for pricing based off of liquidity of the market. Where the step-function's price increase per step would need to be revised to have a lower overall slope.

#### Issue 2: Ownership Ambiguity

To work around indivisibility, you might tokenize assets as if they were divisible:

> Represent a ship as `1e18` subunits of "ship".

But this raises new questions:

- Who owns the ship?  
	- The address with a majority of subunits?  
	- Or only the address holding all `1e18` units?
- What happens if two users each hold 50%?

This creates unclear or non-game-native ownership semantics.

#### Issue 3: Player Expectations

In _Eve Frontier_ (and Eve Online), players are used to **precise trades**:

> If I buy a ship, I expect to get **1 ship**, not a fraction.  
> If I buy 1,000 missiles, I expect **exactly** that number.

AMMs, by nature, deliver **curve-smoothed results** that are often approximate, especially in low-liquidity environments.

#### Issue 4: Step Function Volatility

Small inventory size = big price jumps.

Example:

- Only 5 ships are listed in a pool.
- You buy 2 ships.
- The second ship might cost **33% more** than the first - not because of external supply/demand, but due to how the AMM curve reacts to the shrinking pool.

This volatility is jarring and unintuitive in non-divisible item-based economies.
## 4.3. Pros & Cons

### Pros
- Always-on liquidity means better trade execution (higher transaction success rate)
- Reduced on-chain matching logic (which means it's cheaper to execute trades)
### Cons
- Price slippage
- Requires deep pools to be effective.
- Traders have lower precision in pricing.
# 5. When to Use Which

## 5.1. Order-Book Best-Fit Scenarios

In EveFrontier - due to the indivisibility of assets Orderbook-based DEXs likely fit the best scenario for a distributed market with low-liquidity. Their ability to provide stable expected pricing in low-liquidity environments means that they will likely be the main implementations for spread-out distributed marketplaces. They will also likely be the main implementation of ship-based markets due to the low liquidity of ship pricing.

## 5.2. AMM Best-Fit Scenarios

Due to the indivisibility of assets, AMM-based DEXes will likely only be appropriate in scenarios of high liquidity (the scenario of millions of units of a specific asset - where traders might care less about instability of purchase quantity and care more about success rate of trade execution). Even in this scenario, the UX issues will be difficult to overcome - and they may never be overcome for the purposes of using AMMs to trade low-liquidity assets like ships.

## 5.3. Hybrid Options

The eventual ideal approach might be one where a marketplace functions in a hybrid system -using an order book-based DEX by default for trade execution, but specific assets (i.e. ones that have high liquidity in a region and are in a trade hub) might be configured to use an AMM-exchange mechanism. 

This would require creating an abstraction around the storage unit inventory system where specific assets and subsets of their overall quantity in the inventory can be bucketed towards specific on-chain systems to better-handle asset bookkeeping.

# 6. Conclusion & Next Steps

In designing an on-chain marketplace for _Eve Frontier_, two dominant approaches emerge: the **Order Book DEX** and the **Automated Market Maker (AMM)** model. Each has unique strengths and trade-offs that align with different use cases in the game’s economy.

- **Order Book DEXs** offer precise, player-friendly pricing and align well with the indivisible, low-liquidity nature of most in-game goods (like ships). They're optimal for smaller markets and frontier systems where asset volumes are low but pricing fidelity is essential.
    
- **AMMs**, while gas-efficient and offering always-on liquidity, struggle with indivisible assets and low-liquidity environments. Their use may be limited to commodity-style goods in high-traffic trade hubs where price smoothing and approximate quantity resolution are acceptable.
    
- A **hybrid model** may ultimately be the most robust solution: order books by default, with AMMs selectively enabled for high-liquidity assets and locations.

At a strategic level, these on-chain designs unlock a transformative capability for _Eve Frontier_: **global demand transparency** and **decentralized trade coordination**. This not only empowers players as economic actors but also supports emergent gameplay through arbitrage, logistics, and industrial specialization - all rooted in a universally accessible and verifiable economic layer.

## Logical Next Steps

1. **Prototype a Minimal Viable Marketplace**

	- Build a working MVP of the order-book DEX in the current smart assembly system.
    - Measure gas costs, latency, and order update frequency under real gameplay conditions.

2. **Test UX Impact**

    - Conduct UX trials for both systems with real players.
    - Focus on expectation mismatches in AMM-based trade execution for items like ships.

3. **Publish Sample Code & Schema**

    - Share base smart contract logic (e.g., MUD tables, order struct formats) for community iteration.
    - Open a reference indexer design for off-chain order discovery. Or use the on-chain storage for order indexation (store formatted)

4. **Solicit Community Feedback**

    - Open up a discussion thread with other players/devs to identify friction points or new gameplay opportunities.

5. **Implement Hybrid Inventory Routing**

    - Extend SSU logic to support asset-specific routing to either AMM or order book systems based on liquidity or asset type.

6. **Design AMM Logic for Indivisible Assets**

    - Explore step-function pricing models.
    - Simulate pricing behavior across a small number of pools to validate player perception.
    - Solicit feedback


# 7. Concept Gallery

## Gas Fees 

Gas Fees are transaction fees paid by a transaction submitter - a representation of transaction complexity. They are broken into two different components:
- Gas Limit - The maximum amount of computational complexity (a measure of the reading/writing/executing that occurs within a transaction). _think of this like the amount of gasoline you need to make your car travel x miles_
- Gas Price - The price per unit gas consumed for the gas limit. _think of this like the cost of gasoline per liter/gallon_