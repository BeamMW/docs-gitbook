# W

## Wallet

A wallet is a container used to manage the public and private keys in a public key cryptosystem. Beam, as all other cryptocurrencies use public keys to sign transactions, which prove that it is the user who agreed to pay an amount to an address. The signature, along with the corresponding signed data, is used to update the ledger, and write the updated amounts to the corresponding accounts, including fees.

The wallet itself does hold any coins or even have a balance. It is the software implementation which displays this information, retrieving the information from an updated node.

Wallets can be nondeterministic, where each key is generated from a random number, or deterministic, where the keys are derived from a single master key (the seed). The most popular type is the HD wallet (hierarchical deterministic wallet). These seeds are often encoded as the mnemonic code works, for better user experience and security.

Beam Wallet files will be created in following folders:

- Mac: `/Users/{your_user_name}/Library/Application Support/Beam Wallet/`
- Windows: `\Users\{your_user_name}\AppData\Local\Beam Wallet`
- Linux: `/home/{your_user_name}/.local/share/Beam Wallet`