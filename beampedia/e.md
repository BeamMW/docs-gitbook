# E

## Elliptic Curve Cryptography

An elliptic curve is the set of points that satisfy an equation in the form of y^2 = x^3 + ax + b. It has interesting properties, such as symmetry over the x-axis. The second property is that any two points on the curve would make a line which crosses a third point on that curve. If the two points are exactly vertical, the third point is considered to be at infinity. If the two points are the same, the derivative of the curve is taken at that point (the slope of curve at that point). Entering two points and receiving the third one is defined to be the addition of two points. The third point is later reflected over the x-axis.

The idea behind encryption with an elliptic curve is of taking a plain message and defining it as a point in the curve (since a message can be turned into a long number). This point is added to itself over and over again a k number of times. The message turned into a point, and the point turned into another point in the curve multiple times, like a pool ball bouncing around. After some other manipulations depending on the system, we have our cypher-text, i.e., our encrypted message. Without knowing k (the private key), it is prohibitively hard to decrypt the message. i.e., without knowing how many times a pool ball bounced, it is impossible to know where it started.

In practice, the curve be made of discrete points, and not a continuous line, since only integer numbers are used. Furthermore, if we limit ourselves to a subset of numbers by picking a prime number as a maximum, and modulate through it so as numbers start over from 0 if they surpass the maximum, we get a scattered (but symmetrical) array of dots.

An elliptic curve system can be configured by picking:

- A prime number as a maximum,
- A curve equation,
- A public point on the curve.

The private number is the number of times the public point will be added to itself. Computing the private key from the public key is called the elliptic curve discrete logarithm function, and is a trapdoor function.

Elliptic curve cryptography is a modern system and very efficient in time and space. It is slated to replace the ubiquitous RSA system based on prime factorization. Unlike factoring, there doesn’t appear to be shortcuts, and mathematicians still haven’t found an algorithm to solve this problem better than a naive approach. As such, its security is exponentially stronger than RSA.

It has been implemented in by the U.S. government to encrypt internal communications, the Tor project uses it, and it is the new default method for authentication for secure web browsing over SSL/TLS. Among cryptocurrencies, it is used by Bitcoin of course, as well as Ethereum, EOS and Zcash. And as the wiki page says, the list is incomplete, and we can help by expanding it.

Beam uses the libsecp256k1 optimized C library implemented in the Bitcoin Core client, for operation on a curve of type secp256k1, see also this answer. Secp256k1 refers to the parameters of the elliptic curve, as defined in Standards for Efficient Cryptography (SEC). The resulting curve is that of y^2 = x^3 + 7, defined over the field Z (integer numbers) modulo 2^256-2^32-977. Be prepared to use huge integer arithmetic, since this number is several lines long.

## Emission

The act of issuing new coins, minting. In Beam, coins are emitted smoothly, as the reward changes with each new block. This allows a predictable steady growth of money supply determined by the block reward.

When a block is discovered, the discoverer may award themselves a certain number of coins, which is agreed-upon by everyone in the network. This is in order to incentivize mining activity, and for the miners to have a stake in the system to remain honest.

Beam has set in its protocol a periodic halving of reward, with a total amount of approximately 250 million coins. As the number of new coins that are allowed to create in each block dwindles, mining fees will make up a much more important percentage of a miner’s income.

## Encryption

Methods of deliberately distorting a message so that the legitimate receiver can recover it easily while being nearly impossible for an eavesdropper to do the same.

In symmetric encryption, a personal encryption key cipher known only to the sender and its intended receiver is used to control the encryption of the data. Manual encryption has been used since Roman times (such as Caesar cipher, used by Julius Caesar in his private correspondence). Mechanical systems improved on this, and were used during the world wars. With digital hardware, the most commonly used algorithm is Data Encryption Standard (DES), now phased out for Advanced Encryption Standard (AES).

These schemes allow for cryptographic communications among people who have made prior preparation for cryptographic security. However, it is unviable for large telecommunications systems, where users would have to wait for a key to be sent over secure, physical means.

Asymmetric encryption, or public-key cryptography, requires a pair of keys: one for encryption and one for decryption. Public-key cryptography is a modern breakthrough designed by Whitfield Diffie and Martin Hellman, two academics who disrupted the government monopoly on encryption. It uses trapdoor one-way functions so that encryption may be done by anyone with access to the "public key" but decryption may be done only by the holder of the "private key." A public key may be freely published in directories, while the corresponding private key is closely guarded.

The RSA algorithm (developed by Rivest, Shamir, and Adleman) is the most widely used form of public key encryption. Pretty Good Privacy (PGP) was written by Phillip Zimmerman as an open implementation of RSA which empowered people to take their privacy into their own hands. As EFF co-founder John Perry Barlow put it: "You can have my encryption algorithm... when you pry my cold dead fingers from my private key."

For performance reasons, a popular scheme is to use RSA to transmit session keys and then a high-speed cipher like DES for the actual message text. Public-key cryptography enabled private communication and financial transactions over the internet.

Elliptic Curve Cryptography (ECC) is a modern system and very efficient in computational time and space. It is slated to replace RSA. Based on the trap-door properties of points on elliptic curves. Unlike prime factoring, there doesn’t appear to be any mathematical shortcuts. As such, its security is exponentially stronger than RSA.

## Equihash

Equihash is a mining algorithm proposed by Alex Biryukov and Dmitry Khovratovich as part of the University of Luxembourg research group CryptoLUX. It proposed a fast, memory-asymmetric and optimization/amortization-free PoW, based on the generalized birthday problem. In plain English, the problem states that: “Given a year with d days, what is the n amount of randomly chosen people to have a birthday coincidence of at least 50%?” Which can be generalized into k-types of people, finding common birthdays between them. The generalization states that: “Given k lists of n-bit values, find some way to choose one element from each list so that the resulting k values XOR to zero.”

Zcash Foundation implemented Equihash PoW, specified with parameters of n=200, k=9, which dictate how much memory space/bandwidth will be needed to find a solution. In June 2018, Bitmain released the ASIC Antminer Z9 mini, which managed to mine Equihash much more efficiently than commodity CPUs and GPUs by interfacing with SRAM chips at relatively low cost. Zcash founder Zooko Wilcox wrote in a forum post that he never committed to ASIC-resistance since it would be impossible in the long-term.

Equihash, however, allows for an initial widespread distribution of the coins and mining. Beam will set the parameters giving CPU and GPU miners significant head start over ASICs. Thus we will achieve a balance between the tradeoffs of a widespread, democratic coin distribution and the inevitable but synergic attack-resistance from ASIC miners.