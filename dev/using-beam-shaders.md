---
description: This tutorial describes using Beam Shaders on DAppnet
---

# Using Beam Shaders

{% hint style="success" %}
[Why DAppnet](https://dappnet.beam.mw/downloads/)?

Beam consists of three public networks:

1. **DAppnet:** a testing sandbox that produces a pseudo-mining block every 15 seconds.
2. **Testnet:** produces genuine mining blocks but is mostly a testing site before Mainnet deployment.
3. **Mainnet:** the living Beam network with real assets.
{% endhint %}

### Install DAppnet wallet

The latest version of the Beam DAppnet Wallet is on our [website](https://dappnet.beam.mw/downloads/).

When [installing DAppnet](https://beamx.gitbook.io/dappnet-user-guide/settings), choose the default settings to operate the local node (required) to run Beam Shader contracts.  Create a new wallet (save the seed phrase).

{% hint style="info" %}
Beam Shaders require the use of the integrated node found in the DAppnet Wallet and must remain open for this tutorial.
{% endhint %}

### Withdraw from the Faucet

All DAppnet transactions (including Beam Shader deployment) contain transaction fees that are paid in Beam coins, and require a positive wallet balance.

Launch [My DApp Store](https://beamx.gitbook.io/dappnet-user-guide/my-dapp-store) <img src=".gitbook/assets/Screen Shot 2023-05-10 at 10.27.02 PM.png" alt="" data-size="line"> and find the **Faucet Application** <img src=".gitbook/assets/Screen Shot 2023-05-11 at 5.38.24 PM.png" alt="" data-size="line">.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Launch **Faucet >>> Get your first Beam >>> Confirm,**  and Beam coins will deposit into your wallet after the next mining block.

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Configuring your CLI wallet

Beam shaders are deployed via CLI wallet which can be found on our [website](https://dappnet.beam.mw/downloads/). This tutorial uses Windows Command Prompt, but users can substitute [Mac](shader-sdk-mac.md) and Linux commands if desired.

Create a `shader` folder in your hard drive, and transfer CLI wallet archive. The `shader` folder should contain two files: two files: the `CLI wallet archive` and wallet `config` file.

{% hint style="danger" %}
For this tutorial, we will use our DAppnet's `wallet.db`(for testing purposes only. This practice is not recommended with wallets containing real assets).
{% endhint %}

### Copy `wallet.db` from DAppnet Wallet

Beam wallets store wallet metadata (transaction history, balance, etc) locally on your hard drive in the `wallet.db` file.

Locate your DAppnet `wallet.db` in the `%LOCALAPPDATA%\Beam Wallet folder`. Transfer the `wallet.db` in the same `shader` folder.

{% hint style="info" %}
Refer to our [Desktop Wallet guide](beam-wallets/desktop-wallet.md) for `wallet.db` file location for macOS and Linux operating systems.
{% endhint %}

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### Configure CLI wallet settings

One more crucial step before running Shader contracts involves configuring the CLI wallet settings to simplify and shorten commands.

To connect CLI wallet's to the built-in node, first edit the `config` file using text editor (such as [Visual Studio Code](https://code.visualstudio.com/)) and enter the following:

```
log_level=debug
pass=<your wallet password>
node_addr=127.0.0.1:10005
wallet_path=wallet.db
```

The `config` file should resemble the following:

<pre><code><strong>################################################################################
</strong># General options:
################################################################################

# log level [info|debug|verbose]
log_level=debug

# file log level [info|debug|verbose]
# file_log_level=debug

# old logs cleanup period (days)
# log_cleanup_days=5

################################################################################
# Wallet options:
################################################################################

# password for the wallet
pass=123

# phrase to generate secret key according to BIP-39.
# seed_phrase=

# address of node
node_addr=127.0.0.1:10005

# path to wallet file
wallet_path=wallet.db

# command to execute [new_addr|send|receive|listen|init|info|export_miner_key|export_owner_key|generate_phrase]
# command=listen


</code></pre>

The testing environment is ready to deploy Shaders, all that remains is selecting the Shader contract.

### Choosing a  Shader application

The [Dappnet Blockchain Explorer](https://dappnet.explorer.beam.mw/) (see **Contracts** tab) contains a list of Shader contract applications ready for testing.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

For the purposes of this tutorial, we will select the **Vault** Shader application (copy the Vault Contract ID -- we will use this later).

{% hint style="info" %}
At thid time, the list of available contracts is DAppnet. The majority of deployed contracts don't contain a contract description. These features will be adjusted with future iterations of DAppnet.
{% endhint %}

**Please download the Vault application Shader from our** [**Beam repository.** ](https://github.com/BeamMW/beam/tree/master/bvm/Shaders/vault)The Shader contract will not respond to CLI commands without the precompiled source code.

Download [`app.wasm`](https://github.com/BeamMW/beam/blob/master/bvm/Shaders/vault/app.wasm) and copy it to your `shader` folder. **Note**: if you are testing several applications, create a folder for each Shader application so each application contains a separate `app.wasm` file.

### Running Shader commands

Open as command line interface and change directory to the location of the CLI wallet executable.

First, print the contract API with the following command:

```
beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm
```

Note, we have provided only a single parameter (`--shader_app_file`) since it's assume all other parameters are set in the `config` file and a separate path to the application Shader is found in the `shader`  folder (with the Vault contract).

The output should resemble the following (debug logs are found in this example):

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption><p><code>--shader_app_file method</code></p></figcaption></figure>

Copy the Shader output (see below) into your text editor, (following JSON format).

```json
 {
     "roles":
     {
         "manager":
         {
             "create":
             {},
             "destroy":
             {
                 "cid": "ContractID"
             },
             "view":
             {},
             "view_logs":
             {
                 "cid": "ContractID"
             },
             "view_accounts":
             {
                 "cid": "ContractID"
             },
             "view_account":
             {
                 "cid": "ContractID",
                 "pubKey": "PubKey"
             }
         },
         "my_account":
         {
             "view":
             {
                 "cid": "ContractID"
             },
             "get_key":
             {
                 "cid": "ContractID"
             },
             "get_proof":
             {
                 "cid": "ContractID",
                 "aid": "AssetID"
             },
             "deposit":
             {
                 "cid": "ContractID",
                 "pkForeign": "PubKey",
                 "bCoSigner": "uint32_t",
                 "amount": "Amount",
                 "aid": "AssetID"
             },
             "withdraw":
             {
                 "cid": "ContractID",
                 "pkForeign": "PubKey",
                 "bCoSigner": "uint32_t",
                 "amount": "Amount",
                 "aid": "AssetID",
                 "amountCoSigner": "Amount"
             }
         }
     }
 }
```

The Vault contract API (as retrieved from the Vault Shader contract) separates the API actions into roles, (`manager` , `my_account`) following semantic structure.

The `view_accounts` method will list all Vault accounts present and were we can can plug the Valut Shader (found in the [DAppnet explorer](https://dappnet.explorer.beam.mw/)) contract ID into the `view_accounts` parameter.

```
d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14
```

Enter the following command:

```
beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm --shader_args=cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14,role=manager,action=view_accounts
```

The output should resemble the following:

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption><p>View_accounts method</p></figcaption></figure>

In the above example, there are currently no accounts within the contract. To create an account, utilize `deposit` method and deposit one Beam coin with the following command:

```
beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm --shader_args="cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14,role=my_account,action=deposit,amount=100000000"
```

**Note:** `amount` is set in Groth, which is $$1^10-8$$  of Beam.

The above action creates and sends the transaction to the network (so it will take a moment). When completed, the output should resemble the following:

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption><p><code>Deposit</code> Method</p></figcaption></figure>

The response generated (see below) clarfies the API response:

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption><p>Deposit successful</p></figcaption></figure>

To confirm the transaction was successful, re-enter the `accounts` command:

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption><p>New account creation</p></figcaption></figure>

The response generated confirms a new account is identified by your wallet public key and has a balance of one Beams. To get aquainted with the API function, try depositing and withdrawing Beam's into your account. Users can also attempt to retrieve Beam's from a seperate account (**spoiler: it won't work**).

To learn how to code your own Shader application contract and deploy it chain, please refer to our [Shader SDK tutorial](shader-sdk-win.md).
