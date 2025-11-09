---
title: Recipe Book
excerpt: Where to find common data objects and logic in the World Chain Contracts
tags:
  - ecosystem
  - development
  - alias__sui-migration
date: July 9th, 2025
image: /dust.avif
---
# Data in tables
## Solidity
To read data from tables within Solidity - import the table from the relevant NPM package exporting the table (in non-custom cases, this will be auto-generated into a `@eveworld/world-v2/src/namespaces/evefrontier/codegen/tables/*` table).

Examples:
```solidity
import { Characters } from "@eveworld/world-v2/src/namespaces/evefrontier/codegen/tables/Characters.sol"

uint256 characterId = 1;
uint256 tribeId = Characters.getTribeId(characterId)
```
## Characters

### Get Character ID By Address
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e746965720000004368617261637465727342794163636f&query=SELECT+%22account%22,+%22smartObjectId%22+FROM+%22evefrontier__CharactersByAcco%22+LIMIT+100+OFFSET+0;&page=0&pageSize=100)
Fetch `characterId` by account `address` using the `CharactersByAcc` table.

### Get Character Name by ID
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e74696572000000456e746974795265636f72644d657461&query=SELECT%2520%2522smartObjectId%2522%252C%2520%2522name%2522%252C%2520%2522dappURL%2522%252C%2520%2522description%2522%2520FROM%2520%2522evefrontier__EntityRecordMeta%2522%2520LIMIT%25201000%2520OFFSET%25200%253B&page=0&pageSize=100)
Fetch character `name` by `characterId` using the `EntityRecordMeta` table.

### Get Tribe ID by Character ID
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e7469657200000043686172616374657273000000000000&query=SELECT%2520%2522smartObjectId%2522%252C%2520%2522exists%2522%252C%2520%2522tribeId%2522%252C%2520%2522createdAt%2522%2520FROM%2520%2522evefrontier__Characters%2522%2520LIMIT%2520100%2520OFFSET%25200%253B&page=0&pageSize=100&filter=)
Fetch what tribe  (`tribeId`) a character belongs to by `characterId` using the `Characters` table.

## Items

### Get Item `smartObjectId` (`itemid`) by `typeId`

You can use this utility to deterministically generate the on-chain `itemId` given the `typeId` (the ID used to denote items in the local DB in your game files):
```solidity
bytes32 constant hardcodedTenantId = 0xaef21cf96fe11f21f1942e49b070eeb260a15521b689c44fe6bc1ff8f021fee5;
ObjectIdLib.calculateObjectId(hardcodedTenantId, typeId);
```

## Ownership
### Smart Assembly
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e746965720000004f776e65727368697042794f626a6563&query=SELECT+%22smartObjectId%22,+%22account%22+FROM+%22evefrontier__OwnershipByObjec%22+LIMIT+100+OFFSET+0;&page=0&pageSize=100)
Check if what `account`  owns a smart assembly (by `smartObjectId`) using the `OwnershipByObjec` table.

## Metadata
### Smart Assembly Type
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e74696572000000536d617274417373656d626c79000000&query=SELECT%2520%2522smartObjectId%2522%252C%2520%2522assemblyType%2522%2520FROM%2520%2522evefrontier__SmartAssembly%2522%2520LIMIT%2520100%2520OFFSET%25200%253B&page=0&pageSize=100)
You can check what type a smart assembly is by it's `smartObjectId` using the `SmartAssembly` table. Current types are:
- `NWN` - Network Node.
- `manufacturing` - Manufacturing Deployable (Printers / refineries).
- `smart_hangar` - Hangars.
- `SSU` - Smart Storage Units.

### Smart Assembly Location
[MUD Explorer](https://explorer.mud.dev/pyrope/worlds/0xcdb380e0cd3949caf70c45c67079f2e27a77fc47/explore?tableId=0x746265766566726f6e746965720000004c6f636174696f6e0000000000000000&query=SELECT+%22smartObjectId%22,+%22solarSystemId%22,+%22x%22,+%22y%22,+%22z%22+FROM+%22evefrontier__Location%22+LIMIT+100+OFFSET+100;&page=1&pageSize=100)
You can check the location of a smart assembly by it's `smartObjectId` using the `Location` table.

# Utilities
## Tables

### Generate TableId
To generate a table id - you first need to know what the name of the table is as well as what type it is (off-chain vs on-chain). From there, you can use [`ResourceIdLib.encode`](https://github.com/latticexyz/mud/blob/e76d72504e7ee51c76fa380bdc4a56f4815b7b59/packages/store/src/ResourceId.sol) to encode the table id.

A table's `typeId` defines how they function on-chain (off-chain tables writes/updates only occur using log events, so they don't persist state on-chain like on-chain tables). TypeIds can be found in [the MUD repo](https://github.com/latticexyz/mud/blob/e76d72504e7ee51c76fa380bdc4a56f4815b7b59/packages/store/src/storeResourceTypes.sol#L12).

#### Solidity

```solidity
import { ResourceId, ResourceIdLib } from "@latticexyz/store/src/ResourceId.sol";
import { RESOURCE_TABLE, RESOURCE_OFFCHAIN_TABLE } from "@latticexyz/store/src/storeResourceTypes.sol";

// Using the `RESOURCE_TABLE` type makes it an onchain table
ResourceId tableId = ResourceIdLib.encode({
  typeId: RESOURCE_TABLE, // "tb"
  name: "Balance"
});

// Using the `RESOURCE_OFFCHAIN_TABLE` type makes it an offchain table
ResourceId offchainTableId = ResourceIdLib.encode({
  typeId: RESOURCE_OFFCHAIN_TABLE, // "ot"
  name: "Balance" // In actuality, this will end up being both the table namespace AND the table name. The first 14 bytes are the table namespace and the last 16 are the table name.
});
```

#### Javascript
You can also generate the table id with Javascript like so:
```javascript
function encodeResourceId(type, namespace, name) {
	const typeIdBytes = new TextEncoder().encode(type); // Uint8Array
	const nameBytes = new TextEncoder().encode(name); // Ensure exactly 30 bytes
	const paddedName = new Uint8Array(30);
	const namespaceBytes = new TextEncoder().encode(namespace);
	const nameBytesOffset=14
	paddedName.set(nameBytes, nameBytesOffset);
	paddedName.set(namespaceBytes);
	if (!(typeIdBytes instanceof Uint8Array) || typeIdBytes.length !== 2) {
		throw new Error("typeId must be a 2-byte Uint8Array");
	}
	if (!(nameBytes instanceof Uint8Array) || paddedName.length !== 30) {
		throw new Error("name must be a 30-byte Uint8Array");
	}
	
	const fullBytes = new Uint8Array(32);
	fullBytes.set(typeIdBytes, 0);       // first 2 bytes: type
	fullBytes.set(paddedName, 2);         // next 30 bytes: name
	const tableId = Array.from(fullBytes).map(b => b.toString(16).padStart(2, '0')).join('')
	return tableId;
}

const resourceId = encodeResourceId('tb', 'algo_net','AutoDelegation');
console.log(`0x${resourceId}`)

// Expected output: 
// 0x7462616c676f5f6e65740000000000004175746f44656c65676174696f6e0000
```

## Reverse TableId to Table Name & Namespace
To reverse a TableId to it's component table name / namespace use the following solidity functions:
### Solidity

```solidity
ResourceId tableId = 0x6f74776f726c6400000000000000000046756e6374696f6e5369676e61747572;

// From the bytes30 output - the first 14 bytes are the table's namespace and the next 16 are the table name (this is often represented in MUD's interfaces as `${tableNamespace}__${tableName}`).

bytes30 tableResource = tableId.getResourceName(tableId);

function splitResourceName(bytes30 input) external pure returns (bytes14 tableNamespace, bytes16 tableName) {
	assembly {
		tableNamespace := shr(0x60, input) // Keep the top 14 bytes
		tableName := shl(0x70, input) // Shift top 14 out, keep bottom 16 bytes
    }
}

bytes14 tableNamespace, bytes16 tableName = splitResourceName(tableResource);
```

### Javascript
```javascript
function decodeResourceId(tableIdHex) {
  // strip optional 0x
  const hex = tableIdHex.startsWith('0x') ? tableIdHex.slice(2) : tableIdHex;
  if (hex.length !== 64) {
    throw new Error("Expected 32‑byte hex (64 hex chars) after stripping '0x'");
  }

  // build a Uint8Array
  const fullBytes = new Uint8Array(32);
  for (let i = 0; i < 32; i++) {
    fullBytes[i] = parseInt(hex.substr(i * 2, 2), 16);
  }

  // split out
  const typeBytes = fullBytes.slice(0, 2);
  const paddedName = fullBytes.slice(2);

  const decoder = new TextDecoder();

  // decode the 2‑byte type
  const type = decoder.decode(typeBytes);

  // namespace lives at offset 0 of paddedName, up to the first zero byte (or up to offset 14)
  const namespaceZone = paddedName.slice(0, 14);
  const nsEnd = namespaceZone.indexOf(0);
  const namespace = decoder.decode(
    nsEnd === -1 ? namespaceZone : namespaceZone.slice(0, nsEnd)
  );

  // name lives at offset 14 of paddedName, up to the first zero byte
  const nameZone = paddedName.slice(14);
  const nameEnd = nameZone.indexOf(0);
  const name = decoder.decode(
    nameEnd === -1 ? nameZone : nameZone.slice(0, nameEnd)
  );

  return { type, namespace, name };
}

// Example:

// output from `const resourceId = encodeResourceId('tb', 'algo_net','AutoDelegation');`
const resourceId = '0x7462616c676f5f6e65740000000000004175746f44656c65676174696f6e0000';
const { type, namespace, name } = decodeResourceId(resourceId);
console.log({ type, namespace, name })
// Expected output: 
// { type: 'tb', namespace: 'algo_net', name: 'AutoDelegation' }
```