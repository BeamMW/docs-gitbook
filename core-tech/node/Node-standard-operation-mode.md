According to the protocol Node advertises its tip (header) to all the connected peers, which may _potentially_ be interested in it. Potentially interested means the following:
* The peer tip has lower Chainwork
* The peer tip has the same Chainwork, but the tip is different
Immediately after connecting the tip is always sent, because the peer tip is not known yet (by default assumed empty). Then, after each state transition, the tip is sent only if either of the above criterias is satisfied.

In addition to the advertising, Nodes synchronize actively: request and send headers and blocks. The data request logic is designed such that all the nodes aim to reach the tip of the heaviest branch, but in practice nodes would also request any data to achieve a higher state than the current (so that they're not easily paralyzed by revealing a non-existing tip).

There are 2 types of requests: headers and block.

## Headers




sends its tip (header) to every peer (be it client or other Node) when it connects. 

Then, if the peer tip is also received, the Node