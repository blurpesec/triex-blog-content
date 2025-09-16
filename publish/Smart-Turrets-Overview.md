---
title: "Smart Turrets: Overview"
excerpt: An explanation of how smart turrets function
tags:
  - ecosystem
  - eve-frontier
  - smart-turrets
date: September 15th, 2025
author: Hecate
---


# Smart Turrets: Building Programmable Fire Control Systems

Smart turrets represent one of the core programmable systems within Eve Frontier. These automated weapons platforms don't just mindlessly shoot at everything that moves - they can be programmed with sophisticated logic to make complex targeting decisions based on real-time data.

## How Smart Turrets Work by Default

At their core, smart turrets operate on a simple principle: as long as the turret is `online` and has both a weapon and ammunition, it will automatically engage any target within range (approximately 60km) that doesn't belong to the same tribe as the turret's `owner`.

But the real magic happens when you dig into the programmable behavior system.

## The Two Pillars of Fire Control

Smart turrets use two fundamental functions to determine their targeting behavior, and understanding these is key to unlocking their full potential:

### 1. Proximity-Based Targeting (`inProximity`)

The [`inProximity`](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/SmartTurretSystem.sol#L71) function controls whether the turret should fire at a player who enters its 60km engagement range.

When a [player](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/types.sol#L15C22-L16C1) comes within range, the function receives detailed information including:

- Player ID
- Ship type they're piloting
- Current HP percentages for shield, armor, and hull
- The existing target queue

You can use any combination of this data to decide whether to add the new player to the target queue or even reprioritize existing targets.

### 2. Aggression-Based Targeting (`aggression`)

The [`aggression`](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/SmartTurretSystem.sol#L118C12-L118C22) function handles a different scenario entirely. It determines whether the turret should engage an `aggressor` (a [player](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/types.sol#L15C22-L16C1) who attacks another) based on the relationship between the aggressor and their `victim` (also a [player](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/types.sol#L15C22-L16C1)).

This function returns an updated target priority list, allowing for complex rules around when to intervene in player conflicts.

## Deploying Custom Fire Control Logic

If you want to implement custom targeting behavior, you'll need to complete two steps:

### Step 1: Deploy Your Contract

Create and deploy a Solidity contract that implements the `inProximity` and `aggression` functions with your custom logic.

### Step 2: Register Your System

Configure the turret to use your custom contract by calling [`configureTurret(uint256 smartObjectId, ResourceId systemId)`](https://github.com/projectawakening/world-chain-contracts/blob/develop/mud-contracts/world-v2/src/namespaces/evefrontier/systems/smart-turret/SmartTurretSystem.sol#L48C6-L52). This ensures that `SmartTurretConfig.get(params.smartObjectId)` returns your custom system ID instead of the default behavior.

## The Possibilities Are Endless

This programmable system opens up incredible possibilities for sophisticated targeting strategies:

**Basic Strategies:**

- Fire on everyone. All the time.
- Fire on everyone except tribe members.
- Fire on everyone except syndicate members (groups of tribes).
- Maintain allow/deny lists for specific players.
- Target specific tribes or coalitions.
- Create "safe passage" rules for certain players.

**Advanced Aggression Rules:**

- Only engage players who have attacked others.
- Protect tribe/syndicate members by targeting their attackers.
- Join tribe members in their conflicts by targeting their victims.
- Implement complex rules like "fire on anyone not in my alliance, except those whose names start with 'A'".
- Create ship-type specific targeting (like "only target Tades-class ships").

## On-Chain Data as Targeting Criteria

Since smart turrets can access any on-chain data, the targeting possibilities extend far beyond simple friend-or-foe identification. Future implementations could include:

- **Payment-based safe passage**: "Has this player paid the toll in the last 5 minutes?"
- **Organizational membership**: "Is this player a shareholder in the Android Players Trading Consortium?"
- **Insurance status**: "Has this player purchased ship insurance for the ship they're currently flying?"
- **Reputation systems**: Target based on kill/death ratios, faction standings, or community reputation scores.
- **Economic activity**: Protect or target players based on their trading history or market activity.