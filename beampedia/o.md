# O

## Offline Transactions

In Mimblewimble there are no addresses, and transactions need to be built interactively by the participating parties. This requires that both parties set direct socket connection for every transaction, which can be impractical, and doesn’t allow for transactions while the receiving party is offline.

A Secure Bulletin Board (SBBS) system solves this problem, enabling offline transactions, which makes the user experience the same as other cryptocurrencies such as Bitcoin. With SBBS, client wallets can exchange messages by store-and-forward nodes.

With SBBS, messages are private when addressed to an individual, such messages are encrypted with a public key before they are sent, without having to trust relaying nodes. Other messages can be set to public and be unencrypted, or sent to specific channels.

## Opt-in Auditability

Auditability allow business to manage in a confidential way while being fully compliant.

The Auditability feature is an extension that we added to the original Mimblewimble protocol, allowing businesses or private individuals to report their financial history to their auditors or any other party of their choosing in a secure and provable way. If the user chooses to use Auditability, his/her wallet generates a key pair for audit.

With Auditability, the user can include additional information (e.g. info about the sender/receiver, invoice details etc) thus making it easier for the auditor to classify each transaction.
The auditor is not, able to create any transaction, nor spend any funds. The auditor can retrieve all transactions form the wallet and the blockchain, make sure that the transactions comply with the presented documents. Auditability is strictly an opt-in feature. If the user hasn’t chosen to be auditable, there is no way anyone can retrieve any information about his transactions from the blockchain.