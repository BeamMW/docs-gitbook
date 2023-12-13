# Beam Eager Electron 5.0 Upgrade Guide for pools and exchanges

Beam Eager Electron 5.0 includes breaking Hard Fork that will be activated on height 777,777 (around 28th of June 2020). It is important to complete all upgrade before that date.

## Mining algorithm update

Eager Electron 5.0 includes change in the mining algorithm. Please make sure your miners are updated to support Beam Hash III algorithm and auto switch at the hard fork block height. 

**IMPORTANT**

Before starting the upgrade procedure it is highly recommended to stop the wallets and back up the wallet.db files.

## Upgrading the node and the wallet

1. Download the new binaries from [Beam Website](/downloads) or build from [source](https://github.com/BeamMW/beam/tree/mainnet)

2. When you first run the new wallet and node binaries on the old databases, the database structure will be upgraded to a new version. This can take a lot of time and it is normal.

3. In order to guarantee network connectivity, we recommend to use the following parameters in your node configuration:

    peers_persistent=1

    peer=ap-nodes.mainnet.beam.mw:8100, eu-nodes.mainnet.beam.mw:8100, us-nodes.mainnet.beam.mw:8100, ap-hk-nodes.mainnet.beam.mw:8100

4. Eager Electron 5.0 introduces support for Confidential Assets. This means that there could be additional coins and tokens traded on the Beam network. By default the wallet support for these assets is **DISABLED**. The pools should keep this option disabled. If you are an exchange and wish to enable support for Confidential Assets in the 5.0 please contact Beam Team directly for guidance.





