> **Historical Proposal** — This document is an early design proposal for the I/O and P2P layer, written before the current implementation. It is preserved for historical reference only. The actual P2P protocol is described in the source under `core/proto.h` and `core/peer_manager.h`.

## Current vision for organizing a network (including p2p) module

### Several levels within the module itself
Just right for testing. The code is clear, you don’t have to look for a piece of logic in other directories. From bottom to top:

1. IO level: operates with connections, raw bytes, reconnections, timeouts and asynchrony (there is polling of the most general nature)

2. Peer level:
* protocol and serialization,
* peers collection,
* broadcastcasts,
* persistent storage associated with peers (let it be separate from the blockchain storage),
*ban/unban
* Dandelion (yes, he is here, not in level 3)

3. Integration layer:
* filter and cache (this is a set of hashes that will allow you not to send a TX or a block to the node that has it, and also not to send to other modules what they already know)
* bridge with the main system, i.e. generation and translation of requests/responses between it (API modules) and level 2
* inter-thread communication

### Highlights more details

1. One thread for all of the above (network logic, as well as levels 2 and 3)
* There is logic here that does not require calculations, you can have a lot of active connections, the limit can only be the network or memory, but not the cpu
* Exception: You may need a separate thread that zips bulky responses. Let them be formed asynchronously as needed
* Interaction with other threads either through queues (if these are requests/responses) or directly (make it explicit to make it clear that this is a constant piece of memory). Place mutexes on small pieces of updated data (for example { total_difficulty, total_height }), which can be quickly read
* In queues, only something easily copied or immutable data, for example. { Type type; shared_ptr<const Tx>, etc.}

2. Networking and protocol
* We are limited to TCP and IPv4. Then we’ll have time to add anything if we need it, otherwise let there be less code at first
* I really like libuv as a library for networking and asynchrony. The rationale is below.
* You can take the protocol from grin as a basis, removing/adding something will not be a problem
* Dandelion: you need to decide on one of the 2 schemes specified in the grin documentation

3. Peers
* Take the logic from grin, for starters
* For storage (not blockchain, only for peers, it should be separate) take sqlite as the most time-tested thing. It is flexible (indexes, etc.). By the way, in-memory sqlite has also proven itself well as a tool for working with tabular data and complex indexes. When/if performance limitations are visible or it becomes clear that indexing is not needed, you can select key-value storage faster

4. Caches and filters
* Same as grin to begin with. It would be possible to try some kind of bloom filter to avoid the limitations that we see in this part from grin

### The case for libuv
It's here: [http://docs.libuv.org/en/v1.x/](http://docs.libuv.org/en/v1.x/), [https://github.com/libuv /libuv](https://github.com/libuv/libuv)

* High-quality and compact item. It contains all node.js and its asynchrony and [much more](https://github.com/libuv/libuv/wiki/Projects-that-use-libuv).
* You can fix the version and embed the sources in the build system
* I have a very positive experience with her, and more than one. In general, I can take on this entire part and guarantee a good result, and a quick one, because... I know from the inside what is there
* It will be a few% slower than a self-written solution on epoll, but it’s mature and more than one Linux
* Asynchrony and timers are also added there; a decent API can be output to std::function, for example
* You can also add http and TLS when needed