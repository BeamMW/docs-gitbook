```C++
struct KeyTag
{
	static const uint8_t Internal = 0;
	static const uint8_t InternalStealth = 8;
	static const uint8_t LockedAmount = 1;
	static const uint8_t Refs = 2;
	static const uint8_t OwnedAsset = 3;
	// ...
};
```
### Comment
The Node manages contract state in terms of its variables, which are key-value pairs that can be explicitly read and written.
Variables are divided into _types_, as denoted by the above tags.

* `Internal` - proprietary contract variable. 
* `InternalStealth` - same as above, except it's not included in the Merkle tree (more about this later)
* `LockedAmount` - auxiliary variables managed by the BVM, specify the amount of funds locked in the contract
   * Key: `AssetID` in big-endian format
   * Value: the amount, 128-bit unsigned integer (a.j.a. `AmountBig`) in big-endian format.
* `Refs` - auxiliary variables managed by the BVM to handle cross-contract references
* `OwnedAsset` - auxiliary variables managed by the BVM, to handle owned assets

### Visibility and permission

Contracts may access only their variables (i.e. variables saved for this contract). All the variable types are visible to the contract, all of them can be read by `LoadVar` method. However only proprietary variables (`Internal` and `InternalStealth`) may be modified (by `SaveVar`), others are read-only.

### Stealth variables

All variable types except `InternalStealth` are stored in a special Merkle tree, whose root is used in the system `Definition` calculation, which is a part of the Beam block header. By such it's possible to get a Merkle proof for contract variables.

Contracts may use `InternalStealth` to avoid some of their variables from being-accounted in the Merkle tree. This may be beneficial if the contract developers don't want to enable zero-knowledge proofs for those variables (by using zk-proofs malicious contract developers can use use commercially valuable data in other contracts).

### App shaders
App shaders (that run outside the blockchain) can read all the variables of all the contracts without limitations. [VarGetProof](VarGetProof) can be used to get a Merkle proof for any existing variable (except `InternalStealth` type).
