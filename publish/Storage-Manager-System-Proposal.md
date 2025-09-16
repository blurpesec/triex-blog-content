---
title: Storage Manager System
excerpt: A proposal for handling multi-system storage units by subdividing primary inventory
tags:
  - ecosystem
  - eve-frontier
  - development
  - proposal
  - storage-abstraction
  - allocation-system
date: June 20th, 2025
author: Hecate
updated_at: July 9th, 2025
created_at: June 20th, 2025
---
# Problem Statement

When assets are deposited into a storage unit they immediately go to ephemeral storage if they’re a non-owner and to primary inventory if the depositor is the owner. If non-owners transfer to the primary inventory (maybe as the result of using a "Marketplace" system or a "Tribe Storage" system) - their goods get aggregated together with the both the owner’s and each other's goods. It is possible to solve this for a single storage unit system by keeping track of deposited / withdrawn goods with an additional table - but this only works as long as that is the only system using it.

# Proposal

Create a proxy wrapper around the basic storage unit functions that handles transferring to/from the primary inventory. Each withdraw-deposit will be extended to include a `bucketId` property. This will still have the issue of the owner depositing directly to primary inventory and not being allocatable - but perhaps that itself can be an "allocation" with it's own abstraction.

## Mechanisms
### Tables

In order to provide a wrapper around where assets are "allocated" to buckets - we effectively need a duplicate table that tracks primary inventory (akin to the existing `InventoryItem` table, but with unique `bucketId` by "deposit target"). This table structure can look like this:

```solidity
BucketedInventoryItem: {
  schema: {
	bucketId: "uint256",
	itemObjectId: "uint256",
	exists: "bool",
	quantity: "uint256",
	index: "uint256",
	version: "uint256",
  },
  key: ["smartObjectId", "bucketId"],
},
```

Additionally - we will want some metadata about the bucket to help with display:
```soldity
BucketMetadat: {
  schema: {
	smartObjectId: "uint256",
	bucketId: "uint256",
	name: "string",
  },
  key: ["smartObjectId", "bucketId"],
},
```

### String Utils

To control sizing in tables - use utils for capping strings at 31 bytes.
(_Note: this is still WIP - DO NOT copy-paste to use_)

```solidity

contract StringCapValidator {
    uint8 constant MAX_BYTES = 31;

    function validateString(string memory input) public pure returns (bool) {
        bytes memory strBytes = bytes(input);
        require(strBytes.length <= MAX_BYTES, "String exceeds 31-byte limit");
        return true;
    }

    function storeValidatedString(string memory input) public pure returns (bytes32) {
        validateString(input);
        bytes32 result;
        assembly {
            result := mload(add(input, 32)) // Load first 32 bytes of the string
        }
        return result;
    }

	function packString(string memory s) public pure returns (uint256) {
	  bytes memory b = bytes(s);
	  validateString(s)
	  uint256 result;
	  for (uint i = 0; i < b.length; i++) {
	    // shift the byte into its position and OR it in
	    result |= uint256(uint8(b[i])) << (8 * (31 - i));
	  }
	  // optionally, you could store the length in the low byte if you need it
	  // result |= b.length;
	  return result;
	}
	
	/// @notice Unpacks a uint256 into an ASCII string, stopping at the first zero byte.
    /// @param value The uint256 containing up to 31 ASCII bytes in its high-order bytes.
    /// @return str The decoded string.
    function unpackString(uint256 value) internal pure returns (string memory str) {
        // Maximum 31 characters
        bytes memory buffer = new bytes(31);
        uint256 idx = 0;
        // We assume the string is left-aligned in the high bytes:
        // i = 0 → byte 31 (the least significant byte) ... i = 30 → byte 1 (the most significant)
        for (uint256 i = 0; i < 31; i++) {
            // shift right by 8*(30 - i) to bring the target byte to the lowest position
            uint8 char = uint8(value >> (8 * (30 - i)));
            if (char == 0) {
                // stop on zero (padding)
                break;
            }
            buffer[idx++] = bytes1(char);
        }
        // shrink to actual length
        assembly {
            mstore(buffer, idx)
        }
        return string(buffer);
    }

}
```

### Systems

Existing mechanisms to write follow the same functionality as listed in the [World V2 Functionality Outline](https://metalyth.org/posts/World-v2-Functionality-Outline#inventory).
- `transferToInventory` - Transfers from one smart object's primary inventory to another smart object's primary inventory.
- `transferFromEphemeral` - Transfers from ephemeral inventory of one player to smart object's primary inventory.
- `transferToEphemeral` - Transfers from smart object's primary inventory to player's ephemeral inventory.
- `crossTransferToEphemeral` - Transfers from ephemeral inventory of one player to ephemeral inventory of another player.

The system will create a wrapper around each function that contains the bucket info it's interacting with.
- `transferFromEphemeral` =>  `deposit` 
- `transferToEphemeral` =>  `withdraw` 
- `transferToInventory` =>  `externalTransfer`  _(Note: Not included in MVP)_

As well as creating a new cross-bucket transfer function
`internalTransfer`

#### Functions
##### `deposit`
A current implementation using `transferFromEphemeral` looks like this:

```solidity
func transferItemsFromEphemeralInventory(
	uint256 smartObjectId,
	uint256 quantityToTransfer,
	uint256 itemId
) public {
	address senderOfItems = _msgSender();
	
	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	ephemeralInteractSystem.transferFromEphemeral(
		smartObjectId, 
		senderOfItems, 
		transferItems
	);
}
```

We'll be extending it to cover the `bucketId` use-case

```solidity
func `deposit`(
	uint256 smartObjectId, 
	uint256 quantityToTransfer, 
	uint256 itemId,
	uint256 bucketId
) public {
	address senderOfItems = _msgSender();

	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	aasInteractSystem.deposit(
		smartObjectId, 
		bucketId, 
		senderOfItems, 
		transferItems
	);
}
```

##### `withdraw`
A current implementation using `transferToEphemeral` looks like this:

```solidity
func transferItemsToEphemeralInventory(
	uint256 smartObjectId, 
	uint256 quantityToTransfer,
	uint256 itemId
) public {
	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	
	address recipientOfItems = _msgSender()
	
	// define transfer(s)
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	ephemeralInteractSystem.transferToEphemeral(
		smartObjectId,
		recipientOfItems,
		transferItems
	);
}
```

We'll be extending it to cover the `bucketId` use-case.

```solidity
func withdraw(
	uint256 smartObjectId, 
	uint256 quantityToTransfer, 
	uint256 itemId, 
	uint256 bucketId
) public {
	address recipientOfItems = _msgSender();

	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	aasInteractSystem.withdraw(
		smartObjectId,
		bucketId,
		recipientOfItems,
		transferItems
	);
}
```

##### `externalTransfer`
_Note: this currently isn't viable since there are some problems with item teleportation in the world-chain-contracts._
A current implementation using `transferToInventory` looks like this:

```solidity
func transferItemsBetweenPrimaryInventories(
	uint256 sourceSmartObjectId,
	uint256 recipientSmartObjectId, 
	uint256 quantityToTransfer, 
	uint256 itemId
) public {
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	
	// define transfer(s)
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	inventoryInteractSystem.transferToInventory(
		sourceSmartObjectId, 
		recipientSmartObjectId,
		transferItems
	 );
}

```

We'll be extending it to cover the `bucketId` use-case.

```solidity
func externalTransfer(
	uint256 sourceSmartObjectId,
	uint256 sourceBucketId,
	uint256 recipientSmartObjectId,
	uint256 recipientBucketId,
	uint256 quantityToTransfer,
	uint256 itemId
) public {

	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	aasInteractSystem.externalTransfer(
		sourceSmartObjectId,
		recipientSmartObjectId,
		sourceBucketId,
		recipientBucketId, // expects recipient bucket to exist
		transferItems
	);
}
```

##### `internalTransfer`
This one will be created from scratch. It doesn't actually "transfer" any items - just changes which `bucketId` "owns" the `itemId` of `quantityToTransfer`.

```solidity
func intenralTransfer(
	uint256 smartObjectId,
	uint256 sourceBucketId,
	uint256 recipientBucketId,
	uint256 quantityToTransfer,
	uint256 itemId
) public {

	// instantiate transfers array
	InventoryItemParams[] memory transferItems = new InventoryItemParams[](1);
	transferItems[0] = InventoryItemParams({
		smartObjectId: uint256(itemId),
		quantity: uint256(quantityToTransfer)
	});
	
	aasInteractSystem.internalTransfer(
		smartObjectId,
		sourceBucketId,
		recipientBucketId,
		transferItems
	);
}
```

### Access Control
 We'll need to come up with some delegation of the access control system that is extensible enough to allow people to set up their own sub-access control for assets within buckets they control.
#### Permissions
 We have some options on approaches to permissions. 
##### Delegated Access Control

In order to provide an extensible access control system, the Storage Manager System implements a replacement of the access control by way of proxying the access control exposed for the primary inventory in the Eve Frontier `world-chain-contracts`. The way this works is similar to the way that smart turrets and smart gates work, where there is a default functionality implemented for specific methods (`canJump`, `inProximity`, `aggression`) - and if the owner doesn't specify a system to delegate these calls to - it won't use a delegated method, but rather just fall-back

#### Locking down owner's access to primary inventory.

CCP hasn't provided any guidance yet on how to make a storage owned by a system - denying owner the ability to withdraw from the SSU's primary inventory.
#### Sane Defaults
If the user doesn't pre-configure access-control for the bucket they're depositing into for the first time, it should auto-configure a self-owned bucket to a shared "Tribe Storage". If the user is attempting to deposit into a bucket that already exists, there should be a `deposit` permission that they are required to have before they can deposit. If they attempt to deposit to it - it should fail. We can use this `deposit` param to dictate what users see when they navigate to the dapp.
## Use-cases

- File-system-like object organization (would maybe require a sort of "tree" structure for bucket ownership, perhaps, where access controls can be dictated by higher-level bucket's access controls).
- Access-controlled corp hangars.
- Marketplaces.
- Direct transfers between players.

Considerations

- We may want to expose some utility methods that allow for easy calculation of the smart object owner's "owned amount" of an asset - with the intent being we can use that to help provide UIs for object owners (or even better - just stop depositing owner items by-default into the shared primary inventory).
- We need a way to create "trustless" systems.