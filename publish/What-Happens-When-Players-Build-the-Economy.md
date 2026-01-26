---
title: What Happens When Players Build the Economy
excerpt: Why did CCP Games choose to build Eve Frontier using blockchain? What does it add to the game? How does it benefit players?
tags:
  - primers
date: January 24th, 2026
author: Hecate
updated_at: January 24th, 2026
created_at: January 24th, 2026
image: /hub-landing.png
---
_For a detailed look into the context of what blockchains are used for - see the Part 1 post [**"Why Blockchain?"**](https://trinary.exchange/posts/why-blockchain)_

Part 1 explained why decentralization matters: blockchains let people who don't trust each other, or any central authority, coordinate using shared rules enforced by transparent code. This post explores what that enables: smart contract blockchains, permission-less modularity, and what happens when you apply these principles to a game world.

## What are Smart Contract Blockchains?

Smart contract blockchains extend Bitcoin's core innovation, decentralized state and execution, to general-purpose programs. While Bitcoin's blockchain is purpose-built for transferring value, smart contract blockchains can run any logic that can be written in code: lending agreements, marketplaces, insurance policies, organizational structures and access controls. These programs (called "smart contracts") execute automatically in a decentralized network and can be programmed to have no owner or administrators

### Building Without Permission

The real power of this technology isn't just running programs in a decentralized way - it's that these programs become building blocks anyone can use. Think of smart contracts like public infrastructure that anyone can connect to and build upon.

Here's how it works: when someone creates a smart contract - say, an online marketplace - anyone else can write code that plugs into it without needing approval. You could build a price-tracking tool that watches that marketplace and texts you when something goes on sale.

Or if someone builds a system that tracks a player or tribe's reputation over time based on inputs from other players, you could create a background check service that looks people up in that system to see if their actions align with the goals of your organization.

### Smart Contracts vs Traditional Platforms

Consider how this differs from traditional games. In World of Warcraft or Eve Online, if you want to build tools that interact with the game, you're limited to whatever developer tools the developer provides - and they can shut it down at any time. With smart contracts, there's no gatekeeper. The marketplace IS the developer's tool. Anyone can read its data, write programs against it, or layer new services on top.

## Why Would CCP Games Use Blockchain for Eve Frontier?

Eve Online has always been known for its player-driven economy and emergent gameplay. Players build corporations, wage wars, manipulate markets, and create their own stories within a complex virtual world. But it all runs on CCP's servers with CCP's code. Want to create new financial instruments for your corp? You're limited to what CCP built. Want tools that analyze market trends? You need their developer tools, which they control.

Eve Frontier changes this by building core game systems - assets, location, identity, and organizations - as smart contracts. The infrastructure becomes something anyone can interact with and extend.

### What This Actually Enables

Consider ship lending. In Eve Online, lending a valuable ship requires personal trust - will they return it? The developer could build a lending system or a mission system (as they released in 2025), but you'd be constrained by whatever features they do or don't choose to include.

In Eve Frontier, someone writes a lending contract: you deposit collateral, borrow the ship, and the code handles everything automatically. Return it on time, get your deposit back. Don't return it, lender gets compensated. No personal trust required.

But here's where it gets interesting. Someone else can now write a contract that references the lending contract - maybe an investment fund that pools player capital to buy expensive ships, then lends them out for profit. The fund contract automatically splits returns among investors based on their stake. Another player writes a reputation tracker that monitors lending history, offering better rates to reliable borrowers. None of these developers needed permission from the original lending contract author.

Or maybe you want to lend to someone who doesn't have enough collateral but is willing to pay a premium interest rate for the privilege. A player organization operates a reputation system tracking contract performance - on-time returns, successful deals, disputes. Before accepting the under-collateralized loan, you query their reputation contract. If they've got a solid track record, you take the risk for the higher return. If not, you pass. The reputation system operator didn't need to coordinate with the lending contract author - they just built a complementary service that makes the original system more useful.

Or take bounty hunting. Someone writes a bounty contract where players can put up ISK for another player's destruction. When someone claims the bounty, the contract verifies the kill (by checking blockchain records of ship destruction) and automatically pays out. A third party writes a bounty board that aggregates all active bounties and shows them on a map. A fourth writes an insurance contract specifically for high-bounty players.

Or territorial control. A player alliance writes a smart contract that collects a tax on all trading in their territory - the tax automatically goes into a shared treasury that funds defense operations. Members vote on tax rates based on how much they've contributed to defense. A rival alliance writes a competing contract offering lower taxes, trying to poach miners and industrialists. The original alliance responds by adding profit-sharing to their contract. None of this required CCP to build "alliance governance mechanics" - players built the exact systems they wanted.

The pattern repeats everywhere. Player-run exchanges where the fee structure is transparent and governed by the top traders. Intelligence networks where scouts get automatically paid based on how valuable their information proves to be. Investment funds backing exploration ventures in exchange for a percentage of discoveries. Each system exists independently, but they all reference each other, stack on each other, and create possibilities no single developer would have designed.

### Game-only Elements

This doesn't mean everything runs on-chain or that CCP has no role. Performance-critical gameplay elements like combat and physics will likely always run on traditional servers. But by putting core economic systems on a blockchain, CCP creates an open ecosystem where the community can extend and enhance the game in ways that would be impossible in a traditional architecture.

## How Does This Benefit Players?

### Economic Warfare Gets Strategic Depth

A richer economy creates more reasons to interact - more reasons to trade, spy, betray, ally, and go to war.

When investment funds back expeditions, those funds become targets. Drain the fund, strand the expedition, watch the fallout. When territorial tax contracts govern mining operations, disrupting that contract means disrupting an alliance's entire funding model. When bounty contracts make assassination profitable, the bounties themselves become strategic weapons - put enough ISK on someone's head and watch their allies reconsider.

Consider what happens when someone exploits a major investment fund - finds an edge case in the smart contract and drains it. In a traditional game, that's a bug report. In Eve Frontier, that's a heist. Every player who had capital in that fund is now looking for payback. The exploit becomes a motive. The fallout becomes a war. The rebuilding becomes a story about which investors stepped up to make others whole and which ones cut their losses.

### More Roles Worth Playing

Not everyone wants to fly combat ships. Some players excel at managing investment portfolios - analyzing which expeditions to back, which funds to join. Others might run bounty boards, curating which contracts are legitimate and which are scams. A third group operates exchanges, competing on fees and features. A fourth specializes in finding exploits in poorly-written contracts - the digital equivalent of a safecracker. The game makes space for economic specialists, each playing a role that matters to the ecosystem.

### Organizations That Match How You Want to Play

Player organizations can take new forms. Eve Online's corporation system is powerful, but everyone works within the same structure CCP designed. With smart contracts, players design their own systems.

A mining cooperative where profits split automatically based on contribution. A mercenary company where contract payments sit in escrow until job completion. A territorial alliance where tax revenue funds shared defense, with members voting on rates based on contribution to military operations. An investment syndicate where fund managers compete for investor capital based on transparent track records.

Or the opposite: a secretive cartel where leadership controls the books and everyone else operates on faith. A raiding alliance where loot distribution is opaque - maybe fair, maybe not, but nobody outside leadership can verify. The structure becomes a choice, not a constraint.

### Emergent Narratives

This matters because structure creates drama. When finances are transparent, disputes become "are these rules fair?" When finances are opaque, disputes become "did leadership steal from us?" - and you can never prove it either way. When governance is encoded, coups require exploiting the code itself. When territorial taxes are algorithmic, competition moves to tax rates and services rather than just military force.

Eve has always been a game where player actions create narratives - heists, betrayals, wars with real stakes. A more complex economy multiplies the possibilities. When an investment fund collapses because someone found a contract exploit, alliances fracture over who's responsible. When a territorial tax contract gets hijacked by a rival who buys up voting rights, the response isn't just military - it's economic countermeasures, competing contracts, trade wars. When a bounty board operator sells targeting data to the highest bidder, the reputation system that everyone relied on suddenly becomes suspect.

These aren't scripted events. They're emergent gameplay made possible by systems players built, tested, exploited, and rebuilt. For a game that's always been about player agency and emergent complexity, blockchain infrastructure aligns with Eve's philosophy - but Eve Frontier takes it further by making the infrastructure itself open and extensible. What players and developers create on this foundation remains to be seen - but for the first time, that's genuinely up to them.