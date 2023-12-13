# Using Beam smart contracts with CLI Wallet

## Getting Started

Beam compatible binaries are available for [all platforms](/downloads/mainnet-mac) on our main website.

Advanced users that wish to build Beam directly from our source code can see this [guide](https://github.com/BeamMW/beam/wiki/How-to-build) for a more detailed run down. 

We strongly recommend for all users, casual and hardcore, that you download the node, CLI wallet in addition to the Desktop wallet. The application for smart contracts is in its infancy, with many unique and helpful smart contracts lacking a graphical interface, meaning they are only available through the Command-Line.

Building out your node and operating the CLI wallet will keep you at the forefront of all the latest and coolest features that Beam has to offer. It is a simple process that only takes a minute.

## Getting the Shaders

Beam Smart Contracts deploy via an application called **Beam Shaders**. Beam Shaders are small (or not so small) programs compatible with several different programming languages before being compiled as a `.wasm` file since Beam Virtual Machine (BVM) runs on WebAssembly.

No coding experience is necessary; several Beam Shader applications are ready and available to [download](https://github.com/BeamMW/beam/tree/mainnet/bvm/Shaders).

Select from the list of Beam Shader apps and get into some 'shady business!'

![](<.gitbook/assets/Screen Shot 2021-10-26 at 12.43.27 PM.png>)

As seen from the above example, there are several app folders, i.e., '`Faucet`,' '`Dao-core`,' and inside each folder will be a list of files that will resemble the following:

![](<.gitbook/assets/Screen Shot 2021-10-26 at 12.39.50 PM.png>)

The reason for that is that each application comes in two parts. The first is the contract deployed to the blockchain (`contract.wasm`) and the second allows the wallet to interact with the contract (`app.wasm`). We suggest creating a `shaders` folder inside the `beam` folder with the subfolder with specific apps like `faucet`.

Now let's see how we can run some apps!

* [`Faucet`](using-beamx-faucet-contract-with-cli-wallet.md#faucet-contract)
* [`BeamX DAO`](using-beamx-faucet-contract-with-cli-wallet.md#beamx-dao-contract)

## `Faucet` contract

The Beam `Faucet` contract allows users to withdraw or deposit small amounts of Beam coins.

### All `Faucet` contract commands

To view all the available contract commands, enter this command:

```
./beam-wallet shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501

I 2021-10-19.16:16:18.853 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-19.16:16:18.854 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-19.16:16:18.854 starting a wallet...
I 2021-10-19.16:16:19.093 wallet successfully opened...
Executing shader...
Shader output: {"roles": {"manager": 
{"create": {"backlogPeriod": "Height","withdrawLimit": "Amount"},
"destroy": {"cid": "ContractID"},"view": {},"view_params": {"cid": "ContractID"},
"view_funds": {"cid": "ContractID"},"view_accounts": {"cid": "ContractID"},
"view_account": {"cid": "ContractID","pubKey": "PubKey"}},
"my_account": {"view": {"cid": "ContractID"},
"deposit": {"cid": "ContractID","amount": "Amount","aid": "AssetID"},
"withdraw": {"cid": "ContractID","amount": "Amount","aid": "AssetID"}}}}
```

### View contract

Be sure you have the latest version of [`Faucet`](https://github.com/BeamMW/beam/tree/mainnet/bvm/Shaders/faucet) inside your `shaders` folder as instructed. The `app.wasm` file uses `Faucet` and `contract.wasm` interacts with the blockchain.

We will use the Beam Faucet contract to retrieve the contract`CID`from the blockchain for this example.

The Command Line instructions for`Faucet`are compatible with macOS (Terminal Window), Linux (Shell), and Windows (Command Prompt) but for the example below, we will use Terminal Window commands.

#### Retrieving the Faucet contract


We assume that your node is running on a local machine at `port 8501`. If your parameters differ from any of the instructions, please adjust them accordingly.


1.  Launch a Terminal Window and change the directory to your Beam CLI wallet by entering the following command:

    ```
    cd /wallet_folder
    ```
2.  Assuming you followed the proper subfolder hierarchy for Faucet, enter the following command:

    ```
    ./beam-wallet shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=view"
    ```



    ```
    ./beam-wallet shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=view"

    I 2021-10-19.16:16:18.853 Beam Wallet 6.1.12023 (mainnet)
    I 2021-10-19.16:16:18.854 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
    I 2021-10-19.16:16:18.854 starting a wallet...
    I 2021-10-19.16:16:19.093 wallet successfully opened...
    Executing shader...
    I 2021-10-19.16:34:07.670 Tip has not been changed
    I 2021-10-19.16:34:07.674 Current state is 1463705-3c732eee8cdb116f
    Shader output: {"contracts": 
    [{"cid": "3fdd4171972875e0ac8f0131b3da047e8323cc9c2c8d53327be427c455d2a716",
    "Height": 1280003}]}
    ```

This command will result in a list of Contract `IDs` from shaders of faucet types currently deployed on the network. It is possible for more than one shader because the Application Shader (`app.wasm`) can interact with several different Contract Shaders (`contract.wasm`) of the same type.

### Withdraw Beams from contract

To withdraw 0.05 Beams from the`Faucet`contract, enter the following command:

```
./beam-wallet shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=my_account,action=withdraw,cid=3fdd4171972875e0ac8f0131b3da047e8323cc9c2c8d53327be427c455d2a716,amount=6100000" -n 127.0.0.1:8501
```

You can replace the`CID`parameter with a different contract `ID` that you received in the previous [steps](using-beamx-faucet-contract-with-cli-wallet.md#view-contract).


**A transaction fee of 0.011 Beams automatically subtracts from the`Faucet`contract withdrawal**. In total, the action draws 0.061 Beams + the transaction fee, leaving the user with 0.05 Beams.


This operation can result in several different outcomes:

* Each contract`Faucet`limits the amount of Beam coins you can withdraw within a given timeframe. These 'constructor parameters' of the shader specify that you should receive the Beams you have requested when you are under that limit.
* If you exceed this limit, you will receive nothing or a`Magical Error 17`, depending on the contract implementation.
* `Magical Error 17`can also appear for a different glitch within the contract.

But how do I know what are the contract parameters?

### Contract parameters

To view all available contract parameters, enter the following:

```
./beam-wallet shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=manager,action=view_params,cid=3fdd4171972875e0ac8f0131b3da047e8323cc9c2c8d53327be427c455d2a716" -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=manager,action=view_params,cid=3fdd4171972875e0ac8f0131b3da047e8323cc9c2c8d53327be427c455d2a716" -n 127.0.0.1:8501
I 2021-10-19.17:06:35.384 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-19.17:06:35.385 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-19.17:06:35.386 starting a wallet...
I 2021-10-19.17:06:35.744 wallet successfully opened...
Executing shader...
Shader output: {"params": {"backlogPeriod": 1440,"withdrawLimit": 6100000}}
```

The shader output should resemble the following:

```
"": {"params": {"backlogPeriod": 1440,"withdrawLimit": 6100000}}
```

The above output means you can extract up to 0.061 Beam every 1440 blocks from this `Facuet`.

### Deposit Beams to`Faucet`

To deposit the 0.05 Beam to the `Faucet` , enter the following:

```
./beam-wallet shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=my_account,action=deposit,cid=3fdd4171972875e0ac8f0131b3da047e8323cc9c2c8d53327be427c455d2a716,amount=5000000" -n 127.0.0.1:8501
```

You can replace the`CID`parameter with any contract `ID` received from the [view contract](using-beamx-faucet-contract-with-cli-wallet.md#view-contract) step.

### Create your Faucet

Since the Faucet is not an 'ownable' contract, creating a different faucet contract with the same parameters is impossible. The solution is to create your own parameters.

Be sure you have to `contract.wasm` file (located in `app.wasm` folder) file to properly deploy the new contract to the blockchain.

With that out of the way, enter the following command:

```
./beam-wallet shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=create,backlogPeriod=13,withdrawLimit=520000000" --shader_contract_file shaders/faucet/contract.wasm
```

Assuming a `Faucet` contract has not run within the last 13 blocks, the contract deployment will be successful, and a new contract will issue the return value.

The final step is to deposit funds into the`Faucet`contract and share the contract `ID` with another user!

## BeamX DAO contract

The founding mission behind Beam was to make the Beam ecosystem as decentralized as possible. The introduction of the BeamX DAO is another step towards realizing that founding mission. The BeamX DAO empowers BeamX stakeholders with total control over the governance protocol.

Stakeholders have voting power on every aspect of the ecosystem through the BeamX token.

BeamX stakeholders are encouraged to vote with the personal voting selection, identity, and financial holdings will all remain private.

BeamX has a 100M total supply with a 4-year emission schedule. Earn BeamX tokens by staking your Beam coins.

### View all commands for BeamX`DAO`contract

To view all DAO contract commands, enter the following:

```
./beam-wallet shader --shader_app_file Shaders\dao-core\app.wasm -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file Shaders\dao-core\app.wasm -n 127.0.0.1:8501
I 2021-10-21.12:12:37.014 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.12:12:37.015 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.12:12:37.015 starting a wallet...
I 2021-10-21.12:12:37.229 wallet successfully opened...
Executing shader...
Shader output: {"roles": {"manager": {"view": {},"explicit_upgrade": {"cid": "ContractID"},
"view_params": {"cid": "ContractID"},"my_xid": {},
"my_admin_key": {},"prealloc_totals": {"cid": "ContractID"},
"prealloc_view": {"cid": "ContractID"},
"prealloc_withdraw": {"cid": "ContractID","amount": "Amount"},
"farm_view": {"cid": "ContractID"},
"farm_get_yield": {"cid": "ContractID","amount": "Amount","hPeriod": "Height"},
"farm_totals": {"cid": "ContractID"},
"farm_update": {"cid": "ContractID","amountBeamX": "Amount","amountBeam": "Amount","bLockOrUnlock": "uint32_t"}}}}
```

### View contract

#### Retrieve the`DAO`contract


Be sure the latest version of the`dao-core`contract folder is within the`shaders_folder` hierarchy as previously instructed. Both `app.wasm`(`Faucet`) and`contract.wasm`to deploy your `DAO`contract.


1.  Launch a Terminal Window and change the directory to your Beam CLI wallet by entering the following command:

    ```
    cd /wallet_folder
    ```
2.  Enter the following command:

    ```
    ./beam-wallet shader --shader_app_file shaders/dao-core/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=view"
    ```

    ```
    ./beam-wallet shader --shader_app_file shaders/dao-core/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=view"
    I 2021-10-19.16:16:18.853 Beam Wallet 6.1.12023 (mainnet)
    I 2021-10-19.16:16:18.854 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
    I 2021-10-19.16:16:18.854 starting a wallet...
    I 2021-10-19.16:16:19.093 wallet successfully opened...
    Executing shader...
    Shader output: {"versions": [{"Number": 0,"Height": 1464845,"cid": "3db292122e26df7b216ae89ae7504c9068b87273f149059915c6f4abbf72ee2d"}],
    "contracts": [{"cid": "3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b","Height": 1464852,"min_upgrade_delay": 10000,
    "min_approvers": 1,
    "admins": [{"id": 0,"pk": "cbf7c381febb71d61457cc51d52f5b1ce53b45eee92734d2bdb293d82a25355000"}],
    "current_version": 0}]}
    ```

    The result will display the contract registered to the Beam blockchain.

    The`CID`contract output should resemble the following:

    ```
    "contracts": [{"cid": "3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b"
    ```

### Staking Beam


**The minimum amount of Beam coins users must stake in order to receive BeamX rewards is 16 Beams!**


To receive BeamX tokens, users must first stake their Beam coins by issuing a contract and claiming their BeamX tokens as rewards. To stake their Beam coins, enter the following command:

```
./beam-wallet  shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeam=1600000000,bLockOrUnlock=1 -n 127.0.0.1:8501
```

For this example, we will use the minimum 16 Beam coins required to fulfill the staking requirements of this contract.

```
./beam-wallet  shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeam=1600000000,bLockOrUnlock=1 -n 127.0.0.1:8501
I 2021-10-21.19:42:42.354 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.19:42:42.354 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.19:42:42.355 starting a wallet...
I 2021-10-21.19:42:42.607 wallet successfully opened...
Executing shader...
I 2021-10-21.19:42:42.667 Tip has not been changed
I 2021-10-21.19:42:42.670 Current state is 1466765-f25f1929fb7fff03
Shader output: {}
Creating new contract invocation tx on behalf of the shader
        Comment: Lock/Unlock and get farmed beamX tokens        Send 16 BEAM
        Total fee: 1100000 GROTH
I 2021-10-21.19:42:51.192 [784c1ac2d03d4832aa5136d2f3762eb2][1] Get proof for kernel: d973ed7374e571f8
I 2021-10-21.19:42:51.206 [784c1ac2d03d4832aa5136d2f3762eb2] Transaction completed        
```

### View total BeamX farmed

To view the total amount of BeamX earned or 'farmed,' enter the`farm_view` command:

```
./beam-wallet  shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_view,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
```

```
./beam-wallet  shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_view,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
I 2021-10-21.19:48:56.753 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.19:48:56.754 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.19:48:56.755 starting a wallet...
I 2021-10-21.19:48:57.005 wallet successfully opened...
Executing shader...
I 2021-10-21.19:48:57.069 Sync up to 1466769-c8c91bf474915cfe
I 2021-10-21.19:48:57.069 Synchronizing with node: 0% (0/1)
I 2021-10-21.19:48:57.081 Synchronizing with node: 100% (1/1)
I 2021-10-21.19:48:57.081 Current state is 1466769-c8c91bf474915cfe
Shader output: {"farming": {"duation": 268,"emission": 203957381876,"h": 1466769,"h0": 252840},
"user": {"beams_locked": 1600000000,"beamX_old": 569592,"beamX_recent": 65919,"beamX": 635511}}
```

The `dao-core` contract parameters are as follows:

| Parameter      | Definition                                                                |
| -------------- | ------------------------------------------------------------------------- |
| `beams_locked` | The total number of Beams staked.                                         |
| `beamX_old`    | The number of BeamX tokens credited to your account.                      |
| `beamX_recent` | BeamX rewards forecast.                                                   |
| `beamX`        | The total amount of BeamX tokens available for withdrawing from staking.  |

### Withdraw BeamX

To view the amount of BeamX available in your wallet with the [f`arm_view`](using-beamx-faucet-contract-with-cli-wallet.md#farm-view) command.

To withdraw BeamX, enter the following command:

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeamX=635511 -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeamX=635511 -n 127.0.0.1:8501
I 2021-10-21.21:34:07.371 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.21:34:07.371 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.21:34:07.371 starting a wallet...
I 2021-10-21.21:34:07.620 wallet successfully opened...
Executing shader...
I 2021-10-21.21:34:07.682 Tip has not been changed
I 2021-10-21.21:34:07.685 Current state is 1466870-f3aa6e59a16e3c3c
Shader output: {}
Creating new contract invocation tx on behalf of the shader
        Comment: Lock/Unlock and get farmed beamX tokens        Recv 635511 ASSET-7-GROTH
        Total fee: 1100000 GROTH
I 2021-10-21.21:35:06.460 [1a2ca260c4a74a7ca4261c812ba68ed2][1] Get proof for kernel: 7c87ecc149b7c77a
I 2021-10-21.21:35:06.475 [1a2ca260c4a74a7ca4261c812ba68ed2] Transaction completed
```

### Stake and withdraw with one transaction

To stake additional Beam coins while withdrawing BeamX tokens, enter the following command:

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeamX=635511,amountBeam=10000000,bLockOrUnlock=1 -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeamX=635511,amountBeam=10000000,bLockOrUnlock=1 -n 127.0.0.1:8501
I 2021-10-21.21:39:17.636 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.21:39:17.637 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.21:39:17.637 starting a wallet...
I 2021-10-21.21:39:17.886 wallet successfully opened...
Executing shader...
I 2021-10-21.21:39:19.075 Sync up to 1466878-4ee8a74a8f1e1175
I 2021-10-21.21:39:19.075 Synchronizing with node: 0% (0/1)
I 2021-10-21.21:39:19.076 Synchronizing with node: 100% (1/1)
I 2021-10-21.21:39:19.077 Current state is 1466878-4ee8a74a8f1e1175
Shader output: {}
Creating new contract invocation tx on behalf of the shader
        Comment: Lock/Unlock and get farmed beamX tokens        Send 10000000 GROTH
        Recv 635511 ASSET-7-GROTH
        Total fee: 1100000 GROTH
```


To withdraw Beam and BeamX with one transaction, specify `"bLockOrUnlock= "in shader_args where : 0 - withdraw Beam , 1 - send Beam`

`--shader_args role=manager,action=farm_update,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b,amountBeamX=635511,amountBeam=10000000,`<mark style="color:blue;">`bLockOrUnlock`</mark>`=`<mark style="color:red;">`1`</mark>


### `Dao-core` contract parameters

To view the `dao-core` contract parameters, enter the following command:

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=view_params,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=view_params,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
I 2021-10-21.22:15:32.925 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.22:15:32.926 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.22:15:32.927 starting a wallet...
I 2021-10-21.22:15:33.176 wallet successfully opened...
Executing shader...
I 2021-10-21.22:15:33.238 Tip has not been changed
I 2021-10-21.22:15:33.240 Current state is 1466907-15c917e6c0cdaa1b
Shader output: {"params": {"aid": 7,"locked_beamX": 9999983410129195,"locked_beams": 1126598099627933}}
```

The output should resemble the following:

```
Shader output: "{"params": {"aid": 7,"locked_beamX": 9999983410129195,"locked_beams": 1126598099627933}}
Where : 
aid = 7 is asset number
locked_beamX = BeamX that are still available 
locked_beams = Total amount in the contract
```

To view more details such as the number of users staking Beam coins, enter the following command:

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=view_farm,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
```

```
./beam-wallet shader --shader_app_file Shaders/dao-core-masternet/app.wasm --shader_args role=manager,action=view_farm,cid=3f3d32e38cb27ac7b5b67343f81cf2f8bc53217eb995cc6c5d78ddc5e7b0642b -n 127.0.0.1:8501
I 2021-10-21.22:55:47.434 Beam Wallet 6.1.12023 (mainnet)
I 2021-10-21.22:55:47.435 Rules signature: 0-ed91a717313c6eb0, 321321-6d622e615cfd29d0, 777777-1ce8f721bf0c9fa7, 1280000-3eaab6ab65b65f94
I 2021-10-21.22:55:47.435 starting a wallet...
I 2021-10-21.22:55:48.634 wallet successfully opened...
Executing shader...
I 2021-10-21.22:55:48.695 Tip has not been changed
I 2021-10-21.22:55:48.697 Current state is 1466935-6d5753dbb8b989d0
Shader output: {"duation": 435,
"total": 100000000000000,
"total_users": 1056,
"avail": 331050228045,
"received": 12621458260,
"beam_locked": 1152179812748426}
```

Command paramters from the above output:

| Parameter     | Definition                                           |
| ------------- | ---------------------------------------------------- |
| `total_users` | The total numer of Beams currently staked.           |
| `avail`       | The total remaining BeamX coins not yet distributed. |
| `received`    | The total amount of BeamX coins distributed.         |
| `beam_locked` | The total Beam coins currently being staked.         |
