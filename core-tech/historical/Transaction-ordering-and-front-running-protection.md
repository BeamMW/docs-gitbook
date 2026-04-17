> **Historical Proposal** — This document is an early design proposal for transaction ordering and front-running protection. The mechanism that was actually implemented is described in [Transactions-Hi-Frequency](../transactions/Transactions-Hi-Frequency.md). This document is preserved for historical reference only.

The following is a design of a trading system, which aims to achieve 2 goals:
1. Front-running protection.
1. Provide faster confirmation time.

It consists of 2 parts that should be implemented:
* Layer-1 (contract)
* Layer-2 (node/miner, transaction creation)

## Layer-1

The idea is to enforce strict transaction ordering. The transaction that relies on specific states of the contracts invoked, should only be included at the intended position. Should not be possible to include if other transaction that may have relevant effects is included in front of it.

This is achieved by a special synchronization contract, named `Semaphore`, designed to be invoked by other contracts. Its state consists of a single hash value, which effectively denotes the current state of the trading system. When a contract that operates in such a synchronized manner is invoked - it in turn invokes the `Semaphore` contract, which does the following:
1. Verifies that its current state is indeed what it should be.
1. Mutates its state w.r.t. the caller contract, and the details (parameters) of its invocation.

That is, a pseudo code of the `Semaphore` looks like this:
```C++
OnSynchronize(Hash initialValue, int iMethod, Blob params)
{
    Hash currentValue = LoadState();
    if (currentValue != initialValue)
        Halt(); // order violation!

    HashCalculator calc;
    calc
        << initialValue
        << get_CallerContractID()
        << iMethod
        << params;

    calc >> currentValue;
    SaveState(currentValue);
}
```

Then, any other contract that enforces ordering:

```C++
OnSomeMethod(Hash semaphoreState, Blob params)
{
      // ... ensure correct access order
      CallFar(semaphoreContractID, semaphoreState, methodID, params);
      // ... continue as usual
}
```

By such strict transaction order is enforced, and any transaction, if accepted, yields exactly the results anticipated by the user.

## Layer 2

The above solves the front-running problem, but if implemented alone means that only a single `Semaphore`-dependent transaction can be mined in a block.
The next step is improving the management of the transaction pool. That is pending transactions, that are supposed to be mined at some point, but not mined yet. It consists of the following:
* Node should report all the pending transactions in the pool (already implemented).
* Node should support the blockchain state querying (contract variables and etc.), which includes the side effects of the selected transactions.
   * This is needed to create a transaction that is supposed to be included after a specific pending transaction.
* When transaction is broadcast, it may specify after which other transaction it should be included.
   * Then its validation should be done on top of that other transaction respectively.
* For mining, more sophisticated transaction selection algorithm should be used
   * At the moment, transaction set is considered _flat_, the node selects them according to their fee/resources ratio.
   * A more correct algorithm should organize transactions in a set of _trees_ of dependent transactions.

Once implemented, this achieves the following:
* Many transactions may be included in the block, yet their order is guaranteed.
* Public fee market. In case of order conflict, i.e. different users send transactions for the same order, they are encouraged to pay more fees to be selected.
* Faster effective confirmation times. Although the average block time remains the same, the transactions are already visible in the pool. The more transactions (and their fees) are placed on top of a user transaction - the more likely it will be included in that order. Users may even send additional dependent transactions on top of their previous ones, before they are finally confirmed in a block.




