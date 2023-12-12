# P

## Pedersen Commitment

The basic tool that Confidential Transactions is based on. A commitment scheme lets you keep a piece of data secret but commit to it so that you cannot change it later. Two properties need to be satisfied: binding and hiding. Binding makes sure that commitments cannot be changed later, while hiding ensures that adversaries are unable to find the original value.

A simple commitment scheme can be constructed using a cryptographic hash, by hashing together the data and a blinding factor, and revealing the hash:

*commitment = SHA256( blinding_factor || data )*

This hash is the commitment, and the operation cannot be reversed to reveal the original data. The data can be later revealed together with the blinding factor and can be verified by hashing them with the same hashing algorithm.

The blinding factor is present because, without one, the data could be potentially guessed, –if the data is small and simple– by comparing the guess to the commitment.

A Pedersen commitment works like the above but with an additional property: commitments can be added. The sum of a set of commitments is the same as a commitment to the sum of the data, with a blinding key set as the sum of the blinding keys.

*C(BF1, data1) + C(BF2, data2) == C(BF1 + BF2, data1 + data2)*

## Privacy

Privacy is the prevention of unauthorized extraction of information from communications over an insecure channel, it is the best-known problem that field of cryptography attempts to solve.

There are many reasons why people need privacy: personal, psychological, financial, social, etc. People encrypt for the same reason they close and lock their doors. Encryption can protect criminals, but so do locked doors.

Although there is no explicit "right of privacy" enumerated in the U.S. Constitution, the assumption that an individual is to be secure from absent a valid warrant is central, and dates back to the Carta Magna. That includes random searches and invasive tactics to catch criminals.

There has never been a ruling or law that persons have to speak in a language understandable by wiretappers. Similarly, there hasn’t been a rule banning encrypted communication.

Privacy is essentially tied to fungibility in cryptocurrencies. As cryptocurrencies with a public ledger such as Bitcoin often lose fungibility, as some coins are treated as more acceptable than others. A user can unknowingly accept coins involved in questionable transactions, which can taint the entire amount of his/her wallet and result in a loss of reputation.

Beam achieves privacy by default, unlike other cryptosystems which require an opt-in. Zcash or Monero, for instance, requires users to choose a private transaction, an option which significantly slows down the transaction, and consequently is only used by 4% of the network, in the case of Zcash.