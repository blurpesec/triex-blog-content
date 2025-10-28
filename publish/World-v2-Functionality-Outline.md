---
title: World v2 Functionality Outline
excerpt: An overview of how Eve Frontier World v2 works
tags:
  - ecosystem
  - eve-frontier
  - development
date: June 16th, 2025
author: Hecate
updated_at: July 9th, 2025
created_at: June 24th, 2025
---
## Smart Object Metadata

Ownership in World v2 is controlled by the [`OwnershipSystem`](https://github.com/projectawakening/world-chain-contracts/blob/14128cda741a9f1711087a04461a15c94a35058b/mud-contracts/world-v2/src/namespaces/evefrontier/systems/ownership/OwnershipSystem.sol) and [`AccessSystem`](https://github.com/projectawakening/world-chain-contracts/blob/14128cda741a9f1711087a04461a15c94a35058b/mud-contracts/world-v2/src/namespaces/evefrontier/systems/access-system/AccessSystem.sol)
## Functionality

### Inventory

In EVE Frontier, various smart assemblies have inventories. Smart assembly inventories are broken up into two types: "_primary_" and "_ephemeral_".

#### Primary Inventory

The "primary" inventory is the share of inventory that a smart assembly has natively assigned to it that is accessible only by the `owner` of the smart assembly and the specific Systems that the owner has delegated access for that smart assembly to.

##### `transferToInventory`

Transfers from one smart object's primary inventory to another smart object's primary inventory.

```solidity
// instantiate transfers array
InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);

uint256 senderSmartObjectId = 1;
uint256 recipientSmartObjectId = 2;

// define transfer(s)
transferItems[0] = InventoryItemParams({
	smartObjectId: uint256(itemId),
	quantity: uint256(quantityToTransfer)
});

// transfer - function in EphemeralInteractSystem
transferToInventory(senderSmartObjectId, recipientSmartObjectId, transferItems);
```


##### Access Control
_...coming soon_

#### Ephemeral Inventory

The "ephemeral" inventory is the share of inventory within a smart assembly that belongs to a player. By default, the *ephemeral inventory* is not accessible to the owner of a smart assembly - but the players can choose to make interactions with the assembly's systems that transfer items out of their ephemeral inventory (perhaps in exchange for currency or other items).

##### `transferFromEphemeral`

Transfers from ephemeral inventory of one player to smart object's primary inventory.

```solidity
function transferItemsFromEphemeral(uint256 smartObjectId, uint256 quantityToTransfer, uint256 itemId) public {
	address senderOfItems = _msgSender();
	
	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	ephemeralInteractSystem.transferFromEphemeral(smartObjectId, senderOfItems, transferItems);
}
```

##### `transferToEphemeral`

Transfers from smart object's primary inventory to player's ephemeral inventory.

```solidity
function transferToEphemeral(uint256 smartObjectId, uint256 quantityToTransfer, uint256 itemId) public {
	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	
	address recipientOfItems = _msgSender();
	
	// define transfer(s)
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	ephemeralInteractSystem.transferToEphemeral(smartObjectId, recipientOfItems, transferItems);
}
```
##### `crossTransferToEphemeral`

Transfers from ephemeral inventory of one player to ephemeral inventory of another player.

```solidity
// instantiate transfers array
InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);

address senderOfItems = _msgSender();
address recipientOfItems = address(0x0000000000000000000000000000000000000000);

// define transfer(s)
transferItems[0] = InventoryItemParams({
	smartObjectId: uint256(itemId),
	quantity: uint256(quantityToTransfer)
});

// transfer - function in EphemeralInteractSystem
crossTransferToEphemeral(smartObjectId, senderOfItems, recipientOfItems, transferItems);
```

##### Access Control
_...coming soon_

## Terminology

### EVM

The **Ethereum Virtual Machine (EVM)** is the runtime environment where all Ethereum smart contracts run.

Think of it like a lightweight, sandboxed computer that runs the same way on every Ethereum node. When you deploy or interact with a smart contract, your code is executed **inside the EVM** in a cryptographically-verifable way.

#### What does the EVM actually do?

- **Executes Smart Contracts:** It runs the bytecode generated from Solidity (or other high-level smart contract programming languages). This bytecode is the compiled version of your contract.

- **Maintains State:** It keeps track of account balances, contract storage, and changes over time.

- **Ensures Determinism:** Every node runs the same EVM logic and arrives at the same result. This is how EVM-based blockchains achieves consensus.

- **Restricts Execution:** To avoid abuse or infinite loops, every operation in the EVM costs "gas" - a fee paid in ETH. If a contract runs out of gas, execution stops.

### MUD Framework

The MUD framework (built by [Lattice.XYZ](https://lattice.xyz/)) is a tech stack for deploying complex applications on top of the EVM, particularly ones that require entity relationships (think - records in relational databases).

[Read the MUD Docs for a more in-depth explanation](https://mud.dev/introduction)

#### MUD World

The MUD world is the overarching name that describes the set of systems and tables that are supported natively by a game world. In the case of Eve Frontier, the MUD world would be the EVE Frontier's native systems defined by the [world-chain-contracts](https://github.com/projectawakening/world-chain-contracts).

#### Tables

In the MUD framework created by Lattice, _Tables_ are data stores following formalized data schema. They come in two types:
##### On-chain

On-chain tables are the default table implementation. They persist all data written and updated in a MUD table to the network that the MUD world exists on.

##### Off-chain

Off-chain tables are a different table implementation that doesn't persist the data on-chain, but instead takes advantage of the EVM network's event logs tooling to persist records for off-chain indexing (including in SQL-queryable formats). The benefits of this are to reduce gas fees for data not explicitly needed by the state for contract functionality. Since emitted logs are not stored in blockchain state on nodes - the cost of servicing logs is cheaper for node operators, so they have a lower transaction fee impact for users than publishing all data on-chain in state.

These Off-chain tables are by-default handled by the [MUD Indexer](https://mud.dev/indexer) to make it easier to query.

##### Use-cases

**An example of where to use each**:
In an item exchange - when a user makes an interaction to buy something - the actual state needs access to pieces of data like "item id", "quantity", "price", and "location" to conduct the trade. These would all exist in **On-chain** tables.

Types of data that would exist in **Off-chain** tables would be "order receipts" or "list of items for sale at location _X_" or "list of active trading locations". The off-chain data would still be useful for improving the user experience of a trading interface - but isn't explicitly needed to conduct trades.

#### Systems

_MUD Systems_ are the logic that are able to interact with _Tables_

### Smart Assemblies

A smart assembly is a deployed structure within Eve Frontier. This covers things like SSUs, Smart Gates, Smart Turrets, etc.
#### Identities
##### `account`
In EveFrontier - an **"account"** refers to a **public-private key pair**, usually generated from your **secret recovery phrase**. This account is what **"owns" your character**.

In the context of **decentralized public key infrastructure** (such as blockchains), **"ownership"** can be thought of like having the **keys to a lockbox** - only the holder of the private key can access or control the associated assets or identities.
##### `smartObjectId`
The on-chain identity of a deployed (or previously-deployed) smart assembly or other type of on-chain objects (like player-characters).



