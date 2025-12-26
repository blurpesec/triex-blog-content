---
title: SUI | Client-side Recipe Book
excerpt: Client side "recipes" for how to build client-side apps interacting with the SUI blockchain
tags:
  - ecosystem
  - development
date: December 24th, 2025
image: /dust.avif
---
A practical, end-to-end reference for basic Sui user functionality using the official Sui npm package.

This document covers:

- Object ownership
- Coin ownership and balances
- Coin supply and metadata
- Transaction building and execution
- Wallet integration
- Dynamic fields and tables
- Real-time event subscriptions (WebSocket)
- Best practices and mental models

---

# Prerequisites

### Install the Sui SDK

```bash
npm install @mysten/sui
```

### Imports

```ts
import { SuiClient, getFullnodeUrl } from "@mysten/sui/client";
import { Transaction } from "@mysten/sui/transactions";
```

---

## Client Setup

### HTTP RPC Client (State Queries)

```ts
const client = new SuiClient({
  url: getFullnodeUrl("mainnet"), // "testnet" | "devnet"
});
```

### WebSocket Client (Events)

```ts
const wsClient = new SuiClient({
  url: "wss://fullnode.mainnet.sui.io:443",
});
```

---
## HTTP API
### Get Objects Owned by an Address

Purpose: Retrieve all objects (coins, NFTs, custom Move objects) owned by a wallet.

```ts
const address = "0xYOUR_ADDRESS";

const objects = await client.getOwnedObjects({
  owner: address,
});
```

#### With Object Details

```ts
const objects = await client.getOwnedObjects({
  owner: address,
  options: {
    showType: true,
    showContent: true,
    showOwner: true,
  },
});
```

Notes:

- Large wallets require pagination (`cursor`, `limit`)
- Objects are the authoritative state in Sui

---

### Get Coins Owned by an Address

Purpose: Retrieve coin objects owned by a wallet.

#### All Coins (All Types)

```ts
const coins = await client.getAllCoins({
  owner: address,
});
```

#### Coins of a Specific Type

```ts
const coins = await client.getCoins({
  owner: address,
  coinType: "0x2::sui::SUI",
});
```

Each coin object includes:

- `coinObjectId`
- `balance` (smallest unit)
- `version`
- `digest`

---

### Get Coin Balance (Quantity)

#### Recommended: Aggregated Balance API

```ts
const balance = await client.getBalance({
  owner: address,
  coinType: "0x2::sui::SUI",
});

console.log(balance.totalBalance);
```

#### Manual Aggregation (Advanced)

```ts
const coins = await client.getCoins({
  owner: address,
  coinType: "0x2::sui::SUI",
});

const total = coins.data.reduce(
  (sum, coin) => sum + BigInt(coin.balance),
  0n
);
```

---

### Get Total Supply of a Coin

Purpose: Retrieve global supply information.

```ts
const supply = await client.getTotalSupply({
  coinType: "0x2::sui::SUI",
});

console.log(supply.value);
```

---

### Get Coin Metadata

Purpose: Display-friendly information.

```ts
const metadata = await client.getCoinMetadata({
  coinType: "0x2::sui::SUI",
});

console.log(metadata.name);
console.log(metadata.symbol);
console.log(metadata.decimals);
```

---

### Reading Objects

#### Get Specific Object Details

```ts
const object = await client.getObject({
  id: "0xOBJECT_ID",
  options: {
    showContent: true,
    showType: true,
    showOwner: true,
  },
});
```

#### Type-Safe Content Parsing

```ts
const content = object.data?.content;
if (content?.dataType === "moveObject") {
  console.log(content.type);     // Full type string
  console.log(content.fields);   // Object fields
}
```

---

### Dynamic Fields & Tables

Purpose: Read nested data structures like maps and collections.

#### Get All Dynamic Fields of an Object

```ts
const fields = await client.getDynamicFields({
  parentId: "0xOBJECT_ID",
});

fields.data.forEach(field => {
  console.log(field.name);
  console.log(field.objectId);
});
```

#### Get Specific Dynamic Field Value

```ts
const field = await client.getDynamicFieldObject({
  parentId: "0xOBJECT_ID",
  name: {
    type: "0x1::string::String",
    value: "key_name",
  },
});

console.log(field.data?.content);
```

#### Common Pattern: Reading Table Entries

```ts
// For tables, dynamic fields store the actual data
const tableField = await client.getDynamicFieldObject({
  parentId: "0xTABLE_ID",
  name: {
    type: "address",
    value: "0xUSER_ADDRESS",
  },
});
```

---

### Pagination Patterns

Purpose: Handle large result sets efficiently.

```ts
let cursor = null;
let allObjects = [];

do {
  const response = await client.getOwnedObjects({
    owner: address,
    cursor,
    limit: 50,
    options: {
      showType: true,
      showContent: true,
    },
  });
  
  allObjects.push(...response.data);
  cursor = response.nextCursor;
} while (response.hasNextPage);
```

---

### Transaction Building & Execution

Purpose: Call Move functions and modify on-chain state.

#### Basic Transaction Structure

```ts
const tx = new Transaction();

tx.moveCall({
  target: `${PACKAGE_ID}::module_name::function_name`,
  arguments: [
    tx.object("0xOBJECT_ID"),
    tx.pure.u64(1000),
    tx.pure.address(address),
  ],
});
```

#### Common Transaction Patterns

##### Splitting Coins

```ts
const tx = new Transaction();

// Split gas coin
const [coin] = tx.splitCoins(tx.gas, [1000]);

// Split specific coin
const [splitCoin] = tx.splitCoins(
  tx.object("0xCOIN_ID"),
  [500]
);
```

##### Transferring Objects

```ts
const tx = new Transaction();

tx.transferObjects(
  [tx.object("0xOBJECT_ID")],
  address
);
```

##### Chaining Move Calls

```ts
const tx = new Transaction();

// First call returns a result
const result = tx.moveCall({
  target: `${PACKAGE_ID}::module::create_item`,
  arguments: [tx.pure.string("Item Name")],
});

// Use result in second call
tx.moveCall({
  target: `${PACKAGE_ID}::module::process_item`,
  arguments: [result],
});
```

##### Merging Coins

```ts
const tx = new Transaction();

tx.mergeCoins(
  tx.object("0xDESTINATION_COIN"),
  [tx.object("0xSOURCE_COIN_1"), tx.object("0xSOURCE_COIN_2")]
);
```

---

### Wallet Integration

Purpose: Connect to user wallets and sign transactions.

#### Using Sui Wallet Standard

```ts
import { getWallets } from "@mysten/dapp-kit";

// Get available wallets
const wallets = getWallets();
const wallet = wallets[0];

// Connect to wallet
await wallet.connect();

// Get current account
const accounts = await wallet.getAccounts();
const currentAccount = accounts[0];
```

#### Sign and Execute Transaction

```ts
const result = await wallet.signAndExecuteTransaction({
  transaction: tx,
  options: {
    showEffects: true,
    showObjectChanges: true,
    showEvents: true,
  },
});

console.log("Transaction digest:", result.digest);
```

#### Sign Transaction Only (Advanced)

```ts
const signedTx = await wallet.signTransaction({
  transaction: tx,
});

// Execute separately
const result = await client.executeTransactionBlock({
  transactionBlock: signedTx.bytes,
  signature: signedTx.signature,
});
```

---

### Transaction Results & Parsing

#### Check Transaction Status

```ts
if (result.effects?.status?.status === "success") {
  console.log("Transaction succeeded");
} else {
  console.error("Transaction failed:", result.effects?.status?.error);
}
```

#### Parse Object Changes

```ts
result.objectChanges?.forEach(change => {
  switch (change.type) {
    case "created":
      console.log("Created object:", change.objectId);
      break;
    case "mutated":
      console.log("Modified object:", change.objectId);
      break;
    case "deleted":
      console.log("Deleted object:", change.objectId);
      break;
    case "transferred":
      console.log("Transferred to:", change.recipient);
      break;
  }
});
```

#### Parse Events

```ts
result.events?.forEach(event => {
  console.log("Event type:", event.type);
  console.log("Event data:", event.parsedJson);
});
```

#### Get Gas Used

```ts
const gasUsed = result.effects?.gasUsed;
console.log("Computation cost:", gasUsed?.computationCost);
console.log("Storage cost:", gasUsed?.storageCost);
console.log("Storage rebate:", gasUsed?.storageRebate);
```

---

### Error Handling

#### Transaction Execution Errors

```ts
try {
  const result = await wallet.signAndExecuteTransaction({
    transaction: tx,
  });
  
  if (result.effects?.status?.status !== "success") {
    const error = result.effects.status.error;
    console.error("Transaction failed:", error);
    
    // Handle specific error types
    if (error?.includes("InsufficientGas")) {
      console.error("Not enough gas");
    } else if (error?.includes("ObjectNotFound")) {
      console.error("Object doesn't exist or was deleted");
    }
  }
} catch (error) {
  console.error("Execution error:", error);
  
  // Handle wallet rejection
  if (error.message?.includes("rejected")) {
    console.error("User rejected transaction");
  }
}
```

#### API Query Errors

```ts
try {
  const object = await client.getObject({
    id: "0xOBJECT_ID",
  });
  
  if (object.error) {
    console.error("Object query failed:", object.error);
  }
} catch (error) {
  console.error("API error:", error);
}
```

---

## Real-Time Events (WebSocket)

Sui supports event subscriptions over WebSocket for reactive systems:

- Games
- DEXs
- Wallet notifications
- Indexers

---

### Subscribe to Events from a Package

Use case: Listen to everything emitted by a smart contract package.

```ts
const unsubscribe = await wsClient.subscribeEvent({
  filter: {
    Package: "0xPACKAGE_ID",
  },
  onMessage: (event) => {
    console.log("Package event:", event);
  },
});
```

---

### Subscribe to Events from a Module

```ts
const unsubscribe = await wsClient.subscribeEvent({
  filter: {
    MoveModule: {
      package: "0xPACKAGE_ID",
      module: "module_name",
    },
  },
  onMessage: (event) => {
    console.log("Module event:", event);
  },
});
```

---

### Subscribe to a Specific Move Event Type

Best for production systems.

```ts
const unsubscribe = await wsClient.subscribeEvent({
  filter: {
    MoveEventType: "0xPACKAGE_ID::market::OrderFilled",
  },
  onMessage: (event) => {
    console.log("Order filled:", event.parsedJson);
  },
});
```

---

### Subscribe to Events by Sender

```ts
const unsubscribe = await wsClient.subscribeEvent({
  filter: {
    Sender: "0xADDRESS",
  },
  onMessage: (event) => {
    console.log("Sender activity:", event);
  },
});
```

---

### Unsubscribe

```ts
await unsubscribe();
```

---

### Replay Historical Events

WebSocket delivery is not guaranteed.  
Use event queries to recover missed data.

```ts
const events = await client.queryEvents({
  query: {
    MoveEventType: "0xPACKAGE_ID::market::OrderFilled",
  },
  limit: 50,
});
```

---

### Best Practices

- Prefer MoveEventType filters over broad subscriptions
- Treat events as signals, not state
- Always re-query objects for correctness
- Implement reconnect and resubscribe logic
- Track `(txDigest, eventSeq)` for recovery

---

## Complete Example: DEX Swap
_Note: this uses a fake/non-existent DEX swap transaction to simulate how things might work for an example dapp_

```ts
import { SuiClient, getFullnodeUrl } from "@mysten/sui/client";
import { Transaction } from "@mysten/sui/transactions";

const client = new SuiClient({ url: getFullnodeUrl("mainnet") });

async function executeSwap(wallet, userAddress) {
  try {
    // 1. Get user's coin balance
    const coins = await client.getCoins({
      owner: userAddress,
      coinType: "0x2::sui::SUI",
    });
    
    if (coins.data.length === 0) {
      throw new Error("No coins available");
    }
    
    // 2. Build transaction
    const tx = new Transaction();
    
    // Split coin for exact amount
    const [swapCoin] = tx.splitCoins(
      tx.object(coins.data[0].coinObjectId),
      [1000000000] // 1 SUI
    );
    
    // Call swap function
    const [outputCoin] = tx.moveCall({
      target: `${DEX_PACKAGE}::pool::swap`,
      arguments: [
        tx.object(POOL_ID),
        swapCoin,
        tx.pure.u64(900000000), // min output
      ],
      typeArguments: [
        "0x2::sui::SUI",
        "0xUSDC_TYPE",
      ],
    });
    
    // Transfer output to user
    tx.transferObjects([outputCoin], userAddress);
    
    // 3. Execute transaction
    const result = await wallet.signAndExecuteTransaction({
      transaction: tx,
      options: {
        showEffects: true,
        showObjectChanges: true,
        showEvents: true,
      },
    });
    
    // 4. Parse results
    if (result.effects?.status?.status === "success") {
      const swapEvent = result.events?.find(
        e => e.type.includes("::pool::SwapEvent")
      );
      
      console.log("Swap successful!");
      console.log("Amount out:", swapEvent?.parsedJson?.amount_out);
      
      return result;
    } else {
      throw new Error(result.effects?.status?.error || "Swap failed");
    }
    
  } catch (error) {
    console.error("Swap error:", error);
    throw error;
  }
}
```