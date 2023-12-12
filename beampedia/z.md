# Z

## Zero-Knowledge Proof (ZK)

Proofs in which no knowledge of the actual proof is conveyed. Alice proves to Bob that she is indeed in possession of some piece of knowledge without revealing any of that knowledge. The concept has been around since around 1985 with "Minimum Disclosure Proofs of Knowledge." Protocols for allowing a “prover” to convince a “verifier” that he knows some verifiable secret without allowing the verifier to learn anything about the secret.

This is useful for computer networks since eavesdroppers cannot steal the knowledge given. Useful for proving possession of some property, or credential, such as age or voting status, without revealing personal information. A practical example is user passwords, which are hashed before transmitted through the network, and thus proves possession of it without disclosing any part of it.

A zero-knowledge proof must satisfy three properties:

- Completeness: if the statement is true (e.g. I have the password), the honest verifier will be convinced of this fact by an honest prover.
- Soundness: if the statement is false (e.g. I don’t have the password), no cheating prover can convince the honest verifier that it is true, except with some small probability.
- Zero-knowledge: if the statement is true, no verifier learns anything other than the fact that the statement is true.