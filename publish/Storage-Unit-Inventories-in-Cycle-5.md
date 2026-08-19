---
title: EVE Frontier Storage Unit Inventories in Cycle 5
excerpt: A builder's guide to EVE Frontier storage units - main, owned, and open inventories, shared storage for tribes, and the extensions that control them.
tags:
  - ecosystem
  - development
  - storage
date: March 14th, 2026
image: /smart-storage-unit.png
---
A **Storage Unit** is an on-chain, programmable storage structure anchored at a location in the EVE Frontier game world. It holds items across multiple distinct inventories, each with different access rules. This guide explains the inventory types, how players interact with them, and the kinds of gameplay they enable.

---

## The Three Inventory Types

Every Storage Unit can host three kinds of inventory, each keyed by a different ID as a dynamic field on the Storage Unit object.

### 1. Main Inventory (Extension-Controlled)

The main inventory is the Storage Unit owner's primary storage slot. It is keyed by the owner's `OwnerCap` ID and created automatically when the Storage Unit is anchored.

**Who controls it:** The registered **extension contract** - a third-party smart contract the owner has authorized on the Storage Unit.

| Action | Function | Who calls it |
|--------|----------|-------------|
| Deposit (extension) | `deposit_item<Auth>` | Extension contract |
| Withdraw (extension) | `withdraw_item<Auth>` | Extension contract |
| Deposit (owner) | `deposit_by_owner` | The owner (requires `OwnerCap<StorageUnit>` + matching sender address) |
| Withdraw (owner) | `withdraw_by_owner` | The owner (requires `OwnerCap<StorageUnit>` + matching sender address) |

The main inventory has **dual access**: the owner can interact with it directly via `deposit_by_owner` / `withdraw_by_owner`, *and* the registered extension can operate on it programmatically. The extension's logic can enforce custom rules (pricing, access lists, time locks, etc.) for other players interacting through it.
### 2. Owned Inventory (Player-Controlled)

Each player who interacts with a Storage Unit can have their own **owned inventory** - a personal slot keyed by that player's `OwnerCap` ID. Owned inventories are created on demand the first time items are deposited.

**Who controls it:** The **player** (character owner) directly, or an extension depositing on their behalf.

**How to interact:**

| Action | Function | Who calls it |
|--------|----------|-------------|
| Deposit (self) | `deposit_by_owner` | The player (requires `OwnerCap` + matching sender address) |
| Withdraw (self) | `withdraw_by_owner` | The player (requires `OwnerCap` + matching sender address) |
| Deposit (via extension) | `deposit_to_owned<Auth>` | Extension contract (recipient does **not** need to be the sender) |

The key difference: `deposit_to_owned` lets an extension push items into *another* player's owned inventory without that player being online or signing the transaction. The player can then withdraw their items at any time using `withdraw_by_owner`.

### 3. Open Inventory (Contract-Controlled)

The open inventory is a shared pool keyed by a deterministic ID derived from the Storage Unit's own object ID. It has no individual owner.

**Who controls it:** Only the registered **extension contract**. No player or owner can directly deposit or withdraw.

**How to interact:**

| Action | Function | Who calls it |
|--------|----------|-------------|
| Deposit | `deposit_to_open_inventory<Auth>` | Extension contract |
| Withdraw | `withdraw_from_open_inventory<Auth>` | Extension contract |

The open inventory is ideal for pooled or communal storage where the extension logic decides how items flow in and out.

---

## Bridging: Moving Items Between Game and Chain

Items can be bridged between the game server and the blockchain through the Storage Unit:

| Direction | Function | What happens |
|-----------|----------|-------------|
| Game → Chain | `game_item_to_chain_inventory` | The game server (via admin ACL) mints items into a player's on-chain owned inventory |
| Chain → Game | `chain_item_to_game_inventory` | A player burns on-chain items; the game server listens for the event and recreates them in-game |

Chain → Game requires a **proximity proof** - the player must be physically near the Storage Unit in the game world to withdraw items back to the game.

---

## Item Lifecycle

Items exist in two forms:

- `ItemEntry` (at rest) - a lightweight data record stored inside an Inventory. No object overhead; think of it like a balance entry.
- `Item` (in transit) - a full Sui object created when withdrawn. Carries a `parent_id` linking it back to the Storage Unit it came from, plus location data. Destroyed when deposited into another inventory.

This means items can only return to the Storage Unit they were withdrawn from (enforced by `parent_id` checks). An item withdrawn from Storage Unit A cannot be deposited into Storage Unit B.

Each inventory tracks **volume capacity**: every item type has a volume, and the total `volume × quantity` across all item types cannot exceed the inventory's `max_capacity`.

---

## Extensions: Programmable Behavior

The owner of a Storage Unit can register an **extension** - a custom smart contract that gains deposit/withdraw authority over the main and open inventories. Extensions use the [typed witness pattern](https://move-book.com/programmability/witness-pattern.html): only the module that defines the witness type `Auth` can create an instance of it, so authorization cannot be forged.

### Extension Management

| Action | Function | Requirement |
|--------|----------|-------------|
| Register/change extension | `authorize_extension<Auth>` | Owner holds `OwnerCap` |
| Freeze extension (permanent) | `freeze_extension_config` | Owner holds `OwnerCap`; extension must be set |

**Freezing** is irreversible. Once frozen, the owner can never change the extension. This is a trust mechanism: other players can verify that the rules governing a Storage Unit will not change.

---

## Gameplay Possibilities

These mechanics combine to support a variety of player-driven scenarios:

### Player Marketplace / Vending Machine
An extension acts as an automated shop - the pattern behind [Trinary Exchange's item marketplace](/features/marketplace). Players deposit items into the main inventory; the extension enforces pricing logic and uses `deposit_to_owned` to deliver purchased items directly into the buyer's owned inventory. The buyer can withdraw at their leisure using `withdraw_by_owner`.

### Tribe Hangar / Shared Storage
A tribe deploys a Storage Unit with an extension that checks tribe membership. The open inventory acts as a [communal pool](/features/shared-storage) - tribe members deposit surplus gear, and authorized members withdraw what they need. The extension enforces any quotas or rank-based restrictions.

### Secure Escrow / Trading Post
Two players agree on a trade. Each deposits items into their own owned inventories at the same Storage Unit. The extension verifies both sides have deposited, then atomically moves items between owned inventories using `deposit_to_owned`, completing the swap without either party needing to trust the other.

### Automated Rewards / Loot Distribution
A game event or quest system runs as an extension. When a player completes an objective, the extension withdraws a reward from the open inventory and deposits it into the player's owned inventory via `deposit_to_owned` - even if the player is offline.

### Rental / Time-Locked Storage
An extension could allow other players to use the main inventory for a fee, enforcing time-based withdrawal windows. Items deposited during a rental period can only be withdrawn before the lease expires; after that, items return to the owner.

### Trustless Public Infrastructure
The owner deploys a Storage Unit, registers an audited extension, and **freezes** the configuration. Other players can inspect the frozen extension on-chain and trust that the rules will never change - no rug-pull possible. This enables community-run infrastructure like public warehouses or [decentralized logistics hubs](/marketplace/discovery).

---

## Quick Reference

| Inventory | Keyed By | Direct Player Access | Extension Access | Created |
|-----------|----------|---------------------|-----------------|---------|
| Main | Owner's `OwnerCap` ID | Owner: Deposit + Withdraw | Deposit + Withdraw | At anchor |
| Owned | Player's `OwnerCap` ID | Deposit + Withdraw | Deposit only | On first use |
| Open | Derived from Storage Unit ID | No | Deposit + Withdraw | At anchor (or on first extension deposit) |

**Constraints that apply to all inventories:**
- The Storage Unit must be **online** (connected to a powered Network Node)
- Items must belong to the same **tenant** as the Storage Unit
- Transit items must have been withdrawn from **this** Storage Unit (`parent_id` check)
- Total stored volume cannot exceed `max_capacity`