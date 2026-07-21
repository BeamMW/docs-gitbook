> **Historical Research** — This document describes a strategy for verifying Ethash PoW inside Beam BVM contracts to enable trustless Ethereum bridges. It is preserved as a record of research into cross-chain bridge design.

To implement trustless bridges between Beam and other blockchains there's a need to verify the headers of the appropriate blockchains in the contract. For a PoW-based blockchain there's a need to verify the appropriate header PoW.

Ethereum mined according to Ethash PoW algorithm. Turns out not only mining, but even the verification of Ethash is a complex task for a contract.

**UPDATE: (2021-05-09)**

Turns out a very similar idea has already been elaborated by [Kyber](https://blog.kyber.network/waterloo-a-decentralized-practical-bridge-between-eos-and-ethereum-c25b1698f010).

And threre's a [Rainbow Ethereum-NEAR Rainbow Bridge](https://near.org/technology/) deployed using this technology.

While the idea is very similar, there are significant differences in the implementation. Our version is somewhat more efficient (in both proof size and verification times) since we use a more sophisticated variation of the Merkle proofs.

# Ethash

Ethash is designed to be memory-hard. Its hi-level design:
* Mining parameters (a.k.a. _Epoch_) are changed each 30K blocks (roughly 5.2 days)
* For the current _Epoch_ a _Cache_ is generated, which as a pseudo-random opaque data set.
  * Its size is gradually increased with each _Epoch_, as of today its size is roughly **70MB**.
  * The cache generation is deliberately complex. Each element depends on the previous (so that it can't be parallelized), and CPU-intensive hash function is used.
  * As for today it takes **several seconds** to calculate on a typical desktop machine.
* There is a much larger data set that can be calculated from the cache (a.k.a. _DAG_).
  * As for today the cache size is roughly **4GB**
  * Each element calculation is deliberately complex. Involves CPU-intensive hash functions, and pseudo-random access of cache.
  * Elements however are independent

The PoW puzzle is to find a _nonce_, from which a 64-element-long pseudo-random _Path_ is derived, whose last element together with the _nonce_ reaches the difficulty target. It is assumed that effective miner should have the _DAG_ in memory, and probe different pseudo-random paths until the needed difficulty target is reached. To verify the PoW only the cache is needed. The chosen path elements can effectively be calculated from the cache (since they are independent).

So that, unlike most of the other blockchains, Ethash verification demands significant memory. Once the cache is prepared, the PoW verification is reasonably fast, but still may be considerable if many headers should be verified at once.

For native Ethereum clients this is not a problem. Normally they interpret blocks/headers sequentially (perhaps with slight reorgs), so that the cache can be generated once and kept in memory for ~5 days. The complexity of verification of a header (tens of milliseconds) is also acceptable.

# Why Ethash verification is hard in contract

First of, the cache is needed. As we mentioned it has size roughly **70MB** (as for today), and computation time of several seconds, even if done natively. This is totally unrealistic for contracts, which are deliberately restricted in both available memory and running time.

One strategy to solve this problem (that we tried) is to provide contracts a special bvm function. It would be like this:

```C++
hash256 get_EthashDagEndpoint(int epochNumber, hash512 seed);
```

The bvm implementation would generate and keep the cache for the specified Epoch in memory, and interpret the needed path for the contract. The rest of the header verification (seed derivation, difficulty test, etc.) can be done inside the contract.

This strategy however has the following drawbacks:
* Adding very specialized functions to the bvm is a drawback. We prefer to keep the functions very generic, and avoid specialized functions unless there's no other option.
* Unlike native Ethereum clients, it's not possible to assume that only the current Epoch would be needed. Contracts might need to process headers from the past, as well as headers from different sidechains with different heights.
* Calculating the cache for each requested Epoch is heavy. Pre-calculating and keeping many of them in the local storage is unrealistic too for most of the clients, because of the considerable cache size of each Epoch.
* Since the contract may want to process many Ethereum headers at once, the performance may be complex even with prepared cache.

# Our strategy

First let's study the pseudo-code of the supposed `get_EthashDagEndpoint`:

```C++
hash256 get_EthashDagEndpoint(int epochNumber, hash512 seed)
{
    Init MyState from seed;
    loop 64 times:
    {
        derive Index from MyState;
        get hash1024 Element from DAG of the specified epoch at position Index;
        mutate MyState by Element
    }

    derive Hash256 MixHash from MyState;
    return  MixHash;
}
```
Note that this code can basically be implemented in the contract. The `MyState` mutation and consequent `Index` derivation are pretty simple (use fast hash function). The problem of course is getting the element from the _DAG_ at the requested `Index`.

The idea is to provide those elements for the verification. Means in order to verify the header, in addition to the header itself, the prover would include those 64 1024-bit elements of the path for the contract. Then the contract would repeat the same path and compute the same `MixHash`.

The question is how can the contract verify that the provided elements of the _DAG_ are indeed correct. For this the Prover would include the Merkle multi-element proof (more about this later).

So that the contract gets:
* The header.
* 64 elements from the _DAG_.
* Merkle proof that proves that the above elements are indeed part of the _DAG_, and (<u>importantly!</u>) they are at the asserted positions.

So, in order to deploy and use contracts written in this way the following is needed:

### Setup.
For each epoch:
* calculate the _DAG_, build a Merkle tree, and save it locally.
  * to reduce the space the lower levels of the tree may be omitted. Each omitted level saves x2 space, and increases x2 the complexity of the consequent proof generations.
* Calculate and save the Merkle tree root.
	
Practically the total number of Epochs can be limited to 1024. This will cover all the headers from the very beginning up to year 2030.
	
### Deploy the contract.
For each Epoch the contract has (hard-coded) the following:
* The size of the _DAG_.
* Merkle root.
		
Alternatively instead of storing this data for each Epoch, we can build yet another Merkle tree. By such the contract would only need a single Merkle root. Then, during the verification, one more Merkle proof would be necessary to verify this specific Epoch params.
		
### Using
To invoke the contract method that is supposed verify the Ethereum header the following is performed by the caller:

* Simulate the PoW verification locally. Obtain the 64 Elements that are needed for the path as well as deduce their indexes.
* Build the Merkle multi-element proof for those elements, given their indexes.
* Concatenate the elements and the Merkle proof, to form a single unified proof.

## Merkle multi-element proof

The goal is to prove that the specified subset of elements is a part of the original set. A naive way to accomplish this would be providing a separate Merkle proof for each element, which is essentially a set of hashes, by interpreting which the verifier should calculate and verify the Merkle root for each element individually. This is suboptimal however, since hashes are likely to repeat (i.e. show up in proof for different elements). Instead of doing this the prover would provide the minimal set of hashes, by using which the verifier would calculate the Merkle root only once.

In our specific case the verifier knows both the original data set size, and the asserted indexes (positions) of the elements in its subset. We designed our protocol such that it proves not only the existence of those elements in the original set, but also their positions. The simplified verifier pseudo-code would look like this:

```C++
Hash get_Root(Set myElements, int rangeBegin, int rangeSize)
{
    if myElements is empty
        // this is where the prover is supposed to provide the complementary sibling hash
        return next Hash from the attached proof;
		
    if rangeSize == 1
        // if the myElements contains more than 1 element (i.e. original set of elements contains collisions, unlikely but possible) - make sure they are all equal.
        return Evaluate hash from myElements[0];
	
    // split the range
    rangeSize /= 2;
	
    int pivot = rangeBegin + rangeSize;
	
    Split myElements into 2 subsets myElementsLo and myElementsHi, according to pivot;
	
    Hash hashLo = get_Root(myElementsLo, rangeBegin, rangeSize);
    Hash hashHi = get_Root(myElementsHi, pivot, rangeSize);
	
    return Hash(hashLo | hashHi);
}
```

The prover interprets this according to the same logic. It just doesn't evaluate anything, and provides the appropriate Merkle tree hashes when needed.

# Pros and cons

Pros:
* No need for specialilzed support functions. Hence no additional threat of abuse and etc.
* Header verification is reasonably fast.
	
Cons:
* To invoke the contract the prover needs to generate and process the _DAG_.
* The proof size is considerable.
  * The needed elements take 8K (64 elements, 1024 bits == 128 bytes each)
  * The Merkle proof size varies, but seems to be around 1100-1200 hashes for all the elements.
  * We decided to use 160-bit hashes, to reduce the proof size, yet have decent security.
  * The overall proof size is about **30KB**

Overall this solution seems very practical. The proof size, albeit considerable, is not prohibitively large. With our current block size limitation of 1MB it's possible to verify up to tens of Ethereum headers at once.
The verification time is reasonably fast too (actually faster than normal verification even with cache prepared). Dominated by the hashing, for which there are native functions provided by the bvm.
