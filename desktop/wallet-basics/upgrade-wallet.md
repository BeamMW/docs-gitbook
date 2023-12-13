---
description: Upgrading from previous version
---

# Upgrade Wallet

Because your transaction history and other wallet data are saved locally in your wallet database file e.g., `wallet.db` of your hard drive, migrating your old `wallet.db` file to the new Beam wallet version is crucial for maintaining consistent bookkeeping.

## To migrate your wallet database

If your wallet automatically discovers one or more previous versions installed, it will suggest migrating to the new version. Migration will copy the existing `wallet.db` file, keeping the previous data intact.

Select **start auto migration** for your Beam wallet to migrate your `wallet.db` file automatically. To manually migrate your `wallet.db` file, follow the steps below:

1. **Launch** Beam desktop wallet<img src="../.gitbook/assets/Screen Shot 2022-07-31 at 8.29.33 PM.png" alt="" data-size="line">.
2. Click **migrate wallet database file manually.**
3. Select the `wallet.db` file you wish to migrate.
4. Enter your wallet password.
5. Wait for your wallet to synchronize with the blockchain (this may take a few minutes).

## How it looks in your wallet

Once a file is selected, click next to enter your wallet database password.

if you do not wish to migrate an existing wallet, you can create a new wallet or choose the location of the `wallet.db` file manually by clicking "Restore wallet or create a new one**."**



![](../.gitbook/assets/Screenshot\_104.png)



### Select database file

If you select start auto migration, you will be asked to choose which `wallet.db` file you want to migrate.

![](../.gitbook/assets/Screenshot\_213.png)

### Enter password

Enter database password (same as wallet password).

![](../.gitbook/assets/Screenshot\_214.png)

### Wait for synch

After your password is entered, the migration will finish, and your wallet will resume synchronization with the blockchain.

![](../.gitbook/assets/Screenshot\_215.png)
