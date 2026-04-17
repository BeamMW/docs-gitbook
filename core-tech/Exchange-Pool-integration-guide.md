
# Exchange/Pool integration guide 

## IMPORTANT: The recommended number of confirmations for Beam transfers is 80 blocks. 

## Getting binaries
First of all, define the network you'd like to "play" with:
- `mainnet` - it's the latest released production version, working with real money and you can get binaries from the [official website](https://www.beam.mw/downloads), see [Github Releases](https://github.com/BeamMW/beam/releases) or build yourself from the sources of ([mainnet branch](https://github.com/BeamMW/beam/tree/mainnet)).
- `testnet` - to check the features will be released soon to production and  you can get binaries from the [official website](https://beam.mw/downloads/testnet) 
 , see [Github Releases](https://github.com/BeamMW/beam/releases) or build from the [testnet branch](https://github.com/BeamMW/beam/tree/testnet).
- `master` - to see the latest changes in development build the [master branch](https://github.com/BeamMW/beam/tree/master).

Here are [detailed instructions on how to build a project](https://github.com/BeamMW/beam/wiki/How-to-build) for *Windows*, *Linux*, *Mac* platforms .
> Add `-DBEAM_NO_QT_UI_WALLET=On` command line parameter to the Cmake if you need only CLI version of the wallet without UI and QT5 library dependencies.

You will need to build 4 binaries: `beam-node`, `explorer-node`, `beam-wallet` and `wallet-api`.
 
## Initializing Wallet

To create a new wallet run the following command (`wallet.db` will be created in the working folder): 
 
`./beam-wallet init`  

Output example for `init` operation:

```
I 2019-02-19.15:28:55.500 Beam Wallet 1.2.4392 (master)
I 2019-02-19.15:28:55.501 Rules signature: 7c360d0c2ee92d9e
I 2019-02-19.15:28:55.501 starting a wallet...
Enter password: ***
Confirm password: ***
I 2019-02-19.15:29:01.716 Generating seed phrase...
======
Generated seed phrase:

        army;wing;very;trim;crumble;meat;spawn;click;donate;loyal;trap;gauge;

        IMPORTANT

        Your seed phrase is the access key to all the cryptocurrencies in your wallet.
        Print or write down the phrase to keep it in a safe or in a locked vault.
        Without the phrase you will not be able to recover your money.
======
I 2019-02-19.15:29:01.950 wallet successfully created...
I 2019-02-19.15:29:01.971 New address generated:

1f27eea1057633e7df23aa51f835a05fa759ca199535fe99952bf9aecad17186d9d

I 2019-02-19.15:29:01.971 comment = default
```
You can also restore wallet with your `seed_phrase` if you've already had it.

`./beam-wallet restore --seed_phrase=<semicolon separated list of 12 seed phrase words>;`

Once wallet is initialised, you have to export your *Owner Key* (it will be needed in the future to start own node).
<details>
<summary>
More info about `owner key`
</summary>
The purpose of the `owner key` is to allow all nodes mining for you to be aware of all mining rewards mined by other nodes so that you would only need to connect to one node to collect all the rewards into your wallet. While in most other cryptocurrencies this is done by simply mining to a single address you control, in Mimblewimble it is not as simple since there are no addresses and the mining rewards should be coded with unique blinding factors which are deterministically derived from the `master key`, and then tagged by the single `owner key`.
</details>

Output example for `export_owner_key` operation:

```
$ ./beam-wallet export_owner_key
I 2019-02-19.15:32:16.217 Beam Wallet 1.2.4392 (master)
I 2019-02-19.15:32:16.218 Rules signature: 7c360d0c2ee92d9e
I 2019-02-19.15:32:16.219 starting a wallet...
Enter password: ***
Owner Viewer key: +SevBZ++xL1wEM+yyGbMI+ZElHahudX8mh6Hu/atJrdtzAOD2zpeb2LPIqQcnvry3JUQFBa9gTAHT98RMQMdcggr+LX0oqdGsVIx3KRkTxyvRdKBnw8lz9uAmMx0P2TNlk30E+M5MCnX7Ngp
```
_Owner Key_ should be kept secret. _Owner Key_ does not allow to spend coins, however it will allow to see all coins mined for you by all miners that use this _Owner Кey_.

Here you can read about all the `beam-wallet` commands: https://beamx.gitbook.io/cli-guide/

## Starting a Node
Run the node with your _Owner Key_ and make sure it has completed the synchronization with the network:  

`./beam-node.exe --peer=eu-node01.mainnet.beam.mw:8100 --key_owner=XPWoJ/ViEO1whRcYC1z/nylDH1C2lqLxpMYU0/AbqP67XI4sgYDkUIvJfUpMFjwdPYNz2A7PCHWo7c/kHHTZ2EDUNv2BJvQHb1KHZjLNZPFgV2wceHfzvCYIUF3cR9ADfVSBquTxEldipNgp`  

>Please, make sure you pass proper peer address for the current network you have chosen before.

<details>
<summary>
List of peers
</summary>

```
MASTER peers:
eu-node01.masternet.beam.mw:8100
eu-node02.masternet.beam.mw:8100
eu-node03.masternet.beam.mw:8100
eu-node04.masternet.beam.mw:8100

TESTNET peers:
ap-node01.testnet.beam.mw:8100
ap-node02.testnet.beam.mw:8100
ap-node03.testnet.beam.mw:8100
eu-node01.testnet.beam.mw:8100
eu-node02.testnet.beam.mw:8100
eu-node03.testnet.beam.mw:8100
us-node01.testnet.beam.mw:8100
us-node02.testnet.beam.mw:8100
us-node03.testnet.beam.mw:8100

MAINNET peers:
eu-node01.mainnet.beam.mw:8100
eu-node02.mainnet.beam.mw:8100
eu-node03.mainnet.beam.mw:8100
eu-node04.mainnet.beam.mw:8100
us-node01.mainnet.beam.mw:8100
us-node02.mainnet.beam.mw:8100
us-node03.mainnet.beam.mw:8100
us-node04.mainnet.beam.mw:8100
ap-node01.mainnet.beam.mw:8100
ap-node02.mainnet.beam.mw:8100
ap-node03.mainnet.beam.mw:8100
ap-node04.mainnet.beam.mw:8100
```
</details>

### Mining support
Beam Node also provides built-in support for *Stratum API* allowing to connect multiple external mining clients to a single node, [read more...](https://beamx.gitbook.io/beam-mining).

## Wallet API

With Wallet API you can 
* check current [wallet status and (balance)](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#wallet_status)
* get all your [UTXO](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#get_utxo)/[transactions](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#tx_list) list
* [create](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#create_address)/[verify](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#validate_address) address 
* [send funds](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#tx_send) and [cancel](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#tx_cancel) transactions
* make a [split of UTXO](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1#tx_split) - create a specific set of outputs with given set of values

There are two ways to send JSON RPC commands to the API, 
* using TCP socket 
* via HTTP calls. 

With TCP you can hold only one connection, send JSON commands separated with `\n` symbol and receive callbacks from the API, in the one hand it's more flexible but a bit difficult to implement in the other hand. 
With HTTP you can do POST requests using `curl` command.  

So, to start API with HTTP support use the command: 
 
`./wallet-api --node_addr=x.x.x.x:port --use_http=1`  
where `node_addr` is your node address and port.

See detailed [wallet API documentation](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API).

## Node Explorer API

With this API you can get information about current blockchain state and blocks using simple GET requests.

To run explorer use the command: 
 
`./explorer-node.exe --peer eu-node01.mainnet.beam.mw:8100 --api_port=8080`

>Please, make sure you pass proper peer address for the current network you have chosen before.
  
See detailed [explorer API documentation](https://github.com/BeamMW/beam/wiki/Beam-Node-Explorer-API)