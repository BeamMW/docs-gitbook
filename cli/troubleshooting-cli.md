---
description: The command line troubleshooting guide.
---

# Troubleshooting CLI

## Topics

* [I am getting`error code=26, file is not a database`error message when starting the command line wallet](troubleshooting-cli.md#i-am-gettingerror-code-26-file-is-not-a-databaseerror-message-when-starting-the-command-line-wallet).
* [I am getting the`Failed. No inputs`exception when starting the command line wallet](troubleshooting-cli.md#i-am-getting-thefailed-no-inputsexception-when-starting-the-command-line-wallet).

### I am getting`error code=26, file is not a database`error message when starting the command line wallet

Refer to your computer terminal:

```
I 2018-12-23.17:32:34.619 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.17:32:34.620 starting a wallet...
Enter password: ***
D 2018-12-23.17:32:36.664 sqlite error code=26, file is not a database
E 2018-12-23.17:32:36.665 Wallet data unreadable, restore wallet.db from latest backup or delete it and reinitialize the wallet
```

**Diagnosis**: You have submitted an incorrect password. The wallet can not decrypt the database file, which is why it reports that data is unreadable. **If you are certain you are using the correct wallet password, delete the wallet database and perform a wallet restoration from your seed phrase**.

### I am getting the`Failed. No inputs`exception when starting the command line wallet

Refer to your computer terminal:

```
I 2018-12-23.17:45:12.529 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.17:45:12.530 starting a wallet...
Enter password: *
I 2018-12-23.17:45:13.226 wallet sucessfully opened...
I 2018-12-23.17:45:13.228 WalletID 14a38140d8e66be9b8f1e8d770161fd33e35f7000053147b5a0f6a83178926b956 subscribes to BBS channel 20
I 2018-12-23.17:45:13.271 [9edc454f2752461eb682f21c4efbd33e] Sending 10 beams  (fee: 0 groth )
E 2018-12-23.17:45:13.272 You only have 0 groth
E 2018-12-23.17:45:13.273 [9edc454f2752461eb682f21c4efbd33e] exception msg:
E 2018-12-23.17:45:13.273 [9edc454f2752461eb682f21c4efbd33e] Failed. No inputs
I 2018-12-23.17:45:13.293 [9edc454f2752461eb682f21c4efbd33e] Transaction failed. Rollback...
```

**Diagnosis**: This error most commonly occurs when sending a transaction with insufficient funds. You can not send an amount of Beam that is greater than the total balance in your wallet.
