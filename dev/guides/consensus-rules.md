# Beam Consensus Rules


_**WARNING:** Used for development and testing only!_


The following parameters allow to configuration of Consensus Rules in the Beam node and wallet.

Consensus Rules are for testing and development purposes only. Changing the consensus parameters changes the Rule Signature and breaks the node and wallet compatibility with the running Mainnet or Testnet servers.

### Rules

| **Parameter**              | **Description & Example**                                                                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| EmissionValue0             | Initial coinbase emission in a single block (in Groth, 10^-8 of Beam)                                                                                            |
| EmissionDrop0              | Height of the last block that still has the initial emission. Emission drops by half in the next block. Default equals one year (assuming one block per minute). |
| EmissionDrop1              | Number of blocks in halving cycle (defaults to four years, assuming one block per minute).                                                                       |
| MaturityCoinbase           | Number of blocks that should be mined (verified) before coinbase UTXO can be spent.                                                                              |
| MaturityStd                | Number of blocks that should be mined (verified) before normal (non-coinbase) UTXO can be spent.                                                                 |
| MaxBodySize                | Block body size (in bytes).                                                                                                                                      |
| DesiredRate\_s             | Target block rate (in seconds).                                                                                                                                  |
| DifficultyReviewWindow     | Number of blocks in the window for the mining difficulty adjustment.                                                                                             |
| TimestampAheadThreshold\_s | Block timestamp tolerance (in seconds).                                                                                                                          |
| WindowForMedian            | Number of blocks considered in calculating the timestamp median.                                                                                                 |
| AllowPublicUtxos           | Flag allowing regular (non-coinbase) UTXO to have non-confidential signature.                                                                                    |
| FakePoW                    | Flag to disable verification of PoW. Mining is simulated by timer.                                                                                               |

The sample below displays the corresponding `.cfg` fie section:

```
################################################################################
# Rules configuration:
################################################################################

# initial coinbase emission in a single block
# EmissionValue0=800000000

# height of the last block that still has the initial emission, the drop is starting from the next block
# EmissionDrop0=525600

# Each such a cycle there's a new drop
# EmissionDrop1=2102400

# num of blocks before coinbase UTXO can be spent
# MaturityCoinbase=240

# num of blocks before non-coinbase UTXO can be spent
# MaturityStd=0

# Max block body size [bytes]
# MaxBodySize=0x100000

# Desired rate of generated blocks [seconds]
# DesiredRate_s=60

# num of blocks in the window for the mining difficulty adjustment
# DifficultyReviewWindow=1440

# Block timestamp tolerance [seconds]
# TimestampAheadThreshold_s=7200

# How many blocks are considered in calculating the timestamp median
# WindowForMedian=25

# set to allow regular (non-coinbase) UTXO to have non-confidential signature
# AllowPublicUtxos=0

# Don't verify PoW. Mining is simulated by the timer
# FakePoW=0
```
