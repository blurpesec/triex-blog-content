---
title: Terminology
excerpt: Reference-able terminology guide for Eve Frontier / Blockchain
tags:
  - ecosystem
  - eve-frontier
  - development
date: June 17th, 2025
author: Hecate
---
# Terminology

## EVM

The **Ethereum Virtual Machine (EVM)** is the runtime environment where all Ethereum smart contracts run.

Think of it like a lightweight, sandboxed computer that runs the same way on every Ethereum node. When you deploy or interact with a smart contract, your code is executed **inside the EVM** in a cryptographically-verifable way.

#### What does the EVM actually do?

- **Executes Smart Contracts:** It runs the bytecode generated from Solidity (or other high-level smart contract programming languages). This bytecode is the compiled version of your contract.

- **Maintains State:** It keeps track of account balances, contract storage, and changes over time.

- **Ensures Determinism:** Every node runs the same EVM logic and arrives at the same result. This is how EVM-based blockchains achieves consensus.

- **Restricts Execution:** To avoid abuse or infinite loops, every operation in the EVM costs "gas" - a fee paid in ETH. If a contract runs out of gas, execution stops.

## MUD Framework

The MUD framework (built by [Lattice.XYZ](https://lattice.xyz/)) is a tech stack for deploying complex applications on top of the EVM, particularly ones that require entity relationships (think - records in relational databases).

[Read the MUD Docs for a more in-depth explanation](https://mud.dev/introduction)

### MUD World

The MUD world is the overarching name that describes the set of systems and tables that are supported natively by a game world. In the case of Eve Frontier, the MUD world would be the EVE Frontier's native systems defined by the [world-chain-contracts](https://github.com/projectawakening/world-chain-contracts).

### Tables

In the MUD framework created by Lattice, _Tables_ are data stores following formalized data schema. They come in two types:
#### On-chain

On-chain tables are the default table implementation. They persist all data written and updated in a MUD table to the network that the MUD world exists on.

#### Off-chain

Off-chain tables are a different table implementation that doesn't persist the data on-chain, but instead takes advantage of the EVM network's event logs tooling to persist records for off-chain indexing (including in SQL-queryable formats). The benefits of this are to reduce gas fees for data not explicitly needed by the state for contract functionality. Since emitted logs are not stored in blockchain state on nodes - the cost of servicing logs is cheaper for node operators, so they have a lower transaction fee impact for users than publishing all data on-chain in state.

These Off-chain tables are by-default handled by the [MUD Indexer](https://mud.dev/indexer) to make it easier to query.

#### Use-cases

**An example of where to use each**:
In an item exchange - when a user makes an interaction to buy something - the actual state needs access to pieces of data like "item id", "quantity", "price", and "location" to conduct the trade. These would all exist in **On-chain** tables.

Types of data that would exist in **Off-chain** tables would be "order receipts" or "list of items for sale at location _X_" or "list of active trading locations". The off-chain data would still be useful for improving the user experience of a trading interface - but isn't explicitly needed to conduct trades.

### Systems

_MUD Systems_ are the logic that are able to interact with _Tables_

## Smart Assemblies

A smart assembly is a deployed structure within Eve Frontier. This covers things like SSUs, Smart Gates, Smart Turrets, etc.
### Identities
#### `account`
In EveFrontier - an **"account"** refers to a **public-private key pair**, usually generated from your **secret recovery phrase**. This account is what **"owns" your character**.

In the context of **decentralized public key infrastructure** (such as blockchains), **"ownership"** can be thought of like having the **keys to a lockbox** - only the holder of the private key can access or control the associated assets or identities.
#### `smartObjectId`
The on-chain identity of a deployed (or previously-deployed) smart assembly or other type of on-chain objects (like player-characters).



