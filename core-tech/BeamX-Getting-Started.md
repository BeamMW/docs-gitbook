
# Welcome to BeamX

Hi and Welcome to **BeamX**, Confidential DeFi Experimental Platform based on Beam Confidential blockchain.

We invite you to build BeamX together with us and actively contribute both as a user and as developer. 
This document contains everything you need to get started with BeamX

## Before we start


**IMPORTANT**

**BeamX is very new and experimental project, it is currently in public testnet, so all the assets on it including Beam are for play only.**

You can expect a lot of things to be broken quite frequently

Here are two Telegram groups for user and developer support:
* https://t.me/BeamXSupport - for general issues related to BeamX
* https://t.me/BeamXDev     - for developers building on BeamX

We recommend that you join these groups and ask any questions 

## Getting the binaries

You can download BeamX compatible binaries for your platform at /downloads

If you would like to build BeamX yourself from source code please refer to the [Building Instructions Guide](https://github.com/BeamMW/beam/wiki/How-to-build)

We strongly recommend that you get the node and the CLI wallet in addition to the Desktop wallet even if you only intend to use BeamX. The reason for that is that many interesting contracts you might want to run will take time to provide graphical applications. Having your own node and running CLI wallet will allow you to use the newest and coolest features. It is not complicated, as we will see in a minute.

## Using BeamX CLI wallet

Once you download and unzip CLI binary, put it in a folder called 'beamx' anywhere on your machine. Once this is done, open your favorite Terminal or Command Prompt and change directory to the 'beamx' folder you have just created.

TIP: Since BeamX CLI wallet is an extended version of Beam CLI wallet most of the documentation [here](/docs/cli) is applicable to BeamX CLI Wallet as well.

### Initializing the wallet

Assuming your wallet binary is called something like beam-wallet-beamx.exe (on Windows) or beam-wallet-beamx (on Linux or Mac) run:

`beam-wallet-beamx.exe init` (for Windows)
`./beam-wallet-beamx init` (for Linux or Mac)

Running this command will:
1. Create a new wallet database in the same folder
2. Print out your seed phrase (which you should copy and keep somewhere, just in case)
3. Prompt you to setup wallet password, we recommend using something short like 123 since, remember, it's just play tokens

Once you are done, move to the next step

### Running your own node

We highly recommend that you run your own node to play with BeamX. To do so, download the node binary and put it in the same location as BeamX CLI Wallet, namely in the 'beamx' folder you have created.

Before you run your own node, you will need to get your owner key by running the following command:

`beam-wallet-beamx.exe export_owner_key` (for Windows)
`./beam-wallet-beamx export_owner_key` (for Linux or Mac)

Running this command will print out your owner key. You will need it in a second when running your node.

Run:

`beam-node-beamx --port=8501 --owner_key=<put here the owner key from the previous step without the brackets> --peer=us-node01.beamx.beam.mw:8100,us-node02.beamx.beam.mw:8100,us-node03.beamx.beam.mw:8100`

Wait for the node to synchronize to the current blockchain height which you can see here: http://beamx.explorer.beam.mw/

You are ready to start using your wallet. To make sure everything works you can start by running:

`beam-wallet-beamx.exe info` (for Windows)
`./beam-wallet-beamx info` (for Linux or Mac)

Which should show that your balance is currently 0

### Getting the Shaders

BeamX Smart Contracts are implemented with special programs called Beam Shaders. Shaders are small (or not so small) programs that can be written in several different programming languages but are eventually compiled to .wasm files (since Beam Virtual Machine runs on something called WebAssembly)

To get started you do not need to write or build Shaders yourself. Instead you can download the latest samples and examples from our repository here:

https://github.com/BeamMW/beam/tree/beamX/bvm/Shaders

As you can see, there are several folders there called 'faucet', 'vault' and 'roulette' and maybe others. In each of those folders are two files usually called 'contract.wasm' and 'app.wasm'.

The reason for that is that each application comes in two parts. One part is the contract that is deployed to the blockchain (contract.wasm) and the other one is used by the wallet to interact with the contract (app.wasm)

We suggest to create a folder called 'shaders' under the 'beamx' folder and put the subfolder with specific apps like 'faucet' and such there.

Now let's see how we can run some apps!

1. Get some coins from [Faucet contract](https://github.com/BeamMW/beam/wiki/Using-BeamX-Faucet-contract-with-CLI-Wallet)
2. Put some coins in a [Vault contract](https://github.com/BeamMW/beam/wiki/Using-BeamX-Vault-contract-with-CLI-Wallet)
3. Play Roulette with [Roulette contract](https://github.com/BeamMW/beam/wiki/Using-BeamX-Roulette-contract-with-CLI-Wallet)