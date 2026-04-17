> **Historical** — This document contains the node and wallet command-line instructions for Beam Testnet 1 (early 2019). The testnet is long retired. It is preserved for historical reference.

**Welcome to Beam Testnet 1**

Node binaries can be downloaded from the Beam website (http://beam-mw.com/downloads) or from Github (https://github.com/beam-mw/beam-builds/tree/master/testnet/release)

You can run the node with command line options, or modify the attached config file. Once you start the node it will create a node.db file in the same folder. This file will store internal state of the node.

**Reporting issues**

In case you encounter any problems, please report them via еmail to testnet@beam-mw.com or open a github ticket at https://github.com/beam-mw/beam

Attach archive with logs, configuration file and command line parameters to allow effective investigation of the issues.

**Running a node**

Node parameters can be either passed through command line or written in the beam-node.cfg file.

Example:

*./beam-node --peer 138.68.130.189:8101 --wallet_seed 111 --mining_threads 1 --file_log_level debug*

The --peer parameter specifies a comma separated list of peers that the node will initially connect to. After the connection is established, the node will get an updated list of peers from the nodes, along with peer ratings and from that moment it will manage the connections on its own.

--wallet_seed - is a secret key for the wallet that will connect to this node to collect mining rewards (if the node is mining)

--mining_treads - specifies the number of CPU cores utilized for mining. If set to 0, node acts as validating node only.

--file_log_level - allows you to see that is going on behind the scenes.

Upon first launch, the node will download current blockchain history in batch mode as one large macroblock. After the initial sync is complete, the node will continue to sync blocks and individual transactions from the current blockchain Tip and onwards. This can be seen in the log entry:

*My Tip: 24704-f2ab414ba6430d85*

Here is the full list of options supported by the node

General options:
  -h [ --help ]                     	list of all options
  -p [ --port ] arg (=10000)        	port to start the server on
  --wallet_seed arg                 	secret key generation seed
  --log_level arg                   	log level [info|debug|verbose]
  --file_log_level arg              	file log level [info|debug|verbose]
  -v [ --version ]               	return project version
  --git_commit_hash                 	return commit hash


Node options:
  --storage arg (=node.db)          	node storage path
  --history_dir arg (=./)           	directory for compressed history
  --temp_dir arg (=/tmp/)           	temp directory for compressed history,
                                        must be on the same volume
  --mining_threads arg (=0)         	number of mining threads(there is no
                                        mining if 0)
  --verification_threads arg (=-1)  	number of threads for cryptographic
                                        verifications (0 = single thread, -1 =
                                        auto)
  --miner_id arg (=0)               	seed for miner nonce generation
  --peer arg                        	nodes to connect to
  --import arg (=0)                 	Specify the blockchain height to
                                        import. The compressed history is
                                	    assumed to be downloaded the the
                                        specified directory

Rules configuration:
  --CoinbaseEmission arg (=80000000)	coinbase emission in a single block
  --MaturityCoinbase arg (=60)      	num of blocks before coinbase UTXO can
                                        be spent
  --MaturityStd arg (=0)            	num of blocks before non-coinbase UTXO
                                        can be spent
  --MaxBodySize arg (=1048576)      	Max block body size [bytes]
  --DesiredRate_s arg (=60)         	Desired rate of generated blocks
                                        [seconds]
  --DifficultyReviewCycle arg (=1440)   num of blocks after which the mining
                                        difficulty can be adjusted
  --MaxDifficultyChange arg (=2)    	Max difficulty change after each cycle
                                        (each step is roughly x2 complexity)
  --TimestampAheadThreshold_s arg (=7200)
                                 	   Block timestamp tolerance [seconds]
  --WindowForMedian arg (=25)       	How many blocks are considered in
                                        calculating the timestamp median
  --AllowPublicUtxos arg (=0)       	set to allow regular (non-coinbase)
                                        UTXO to have non-confidential signature
  --FakePoW arg (=0)                	Don't verify PoW. Mining is simulated
                                        by the timer. For tests only


**Running a command line wallet**

After extracting the wallet binary to a folder, first we need to initialize the wallet by running:

*./beam-wallet --command init*

You will be prompted to provide wallet password and then seed. If you are using a miner node, the seed should be the same as was set in the --wallet_seed flag of the node


**Printing wallet information**

To get the information about the status of the wallet, run:

*./beam-wallet --command info -n 127.0.0.1:10000*


**Receiving Beams**

To receive beams we need to start a wallet in a listening mode by running:

*./beam-wallet --command listen -n 127.0.0.1:10000;*

After entering the password, wallet will print out the line similar to:

*WalletID 4a0e54b24d5fdf06891a8eaa57b4b3ac16731e932a64da8ec768083495d624f1 subscribes to BBS channel 9*

This show the SBBS address the wallet is listening on. This address can be copied and sent to Sender.

**Sending Beams**

*./beam-wallet --command=send -n 127.0.0.1:10000 -r 77de6bd3de40bc58ab7e4fb68d5e0596fd1e72f3c4fb3eb3d106082d89264909 -a 11.3 -f 0.2*

To send Beams, use 'send' command with the following parameters:
-r <SBBS address of the receiver node>
-a <amount of beams to send>
-f <transaction fee>


**Full list of wallet options**

General options:
  -h [ --help ]    	                 list of all options
  -p [ --port ] arg (=10000)        	port to start the server on
  --wallet_seed arg                 	secret key generation seed
  --log_level arg                   	log level [info|debug|verbose]
  --file_log_level arg              	file log level [info|debug|verbose]
  -v [ --version ]                  	return project version
  --git_commit_hash                 	return commit hash

Wallet options:
  --pass arg                        	password for the wallet
  -a [ --amount ] arg               	amount to send (in Beams, 1 Beam =
                                        1000000 chattle)
  -f [ --fee ] arg (=0)             	fee (in Beams, 1 Beam = 1000000
                                        chattle)
  -r [ --receiver_addr ] arg        	address of receiver
  -n [ --node_addr ] arg            	address of node
  --treasury_path arg (=treasury.mw)	Block to create/append treasury to
  --wallet_path arg (=wallet.db)    	path to wallet file
  --bbs_keystore_path arg (=bbs_keys.db)
                                        path to file with bbs keys
  --tx_history                      	print transacrions' history in info
                                        command
  --tr_Count arg (=30)              	treasury UTXO count
  --tr_HeightStep arg (=1440)       	treasury UTXO height lock step
  --tr_BeamsPerUtxo arg (=10)       	treasury value of each UTXO (in Beams)
  --command arg                     	command to execute [send|receive|listen
                                        |init|info|treasury]

Rules configuration:
  --CoinbaseEmission arg (=80000000)	coinbase emission in a single block
  --MaturityCoinbase arg (=60)      	num of blocks before coinbase UTXO can
  	                                  be spent
  --MaturityStd arg (=0)            	num of blocks before non-coinbase UTXO
                                        can be spent
  --MaxBodySize arg (=1048576)      	Max block body size [bytes]
  --DesiredRate_s arg (=60)         	Desired rate of generated blocks
                                        [seconds]
  --DifficultyReviewCycle arg (=1440)   num of blocks after which the mining
                                        difficulty can be adjusted
  --MaxDifficultyChange arg (=2)    	Max difficulty change after each cycle
                                        (each step is roughly x2 complexity)
  --TimestampAheadThreshold_s arg (=7200)
                                        Block timestamp tolerance [seconds]
  --WindowForMedian arg (=25)       	How many blocks are considered in
                                        calculating the timestamp median
  --AllowPublicUtxos arg (=0)       	set to allow regular (non-coinbase)
             	                       UTXO to have non-confidential signature
  --FakePoW arg (=0)                	Don't verify PoW. Mining is simulated
                                        by the timer. For tests only

**List of testnet 1 nodes**

142.93.89.204:8101
188.166.148.169:8101
206.189.141.171:8101
138.68.130.189:8101
178.128.225.252:8101
128.199.142.41:8101
139.59.191.116:8101
206.189.3.9:8101
206.189.15.198:8101
204.48.26.118:8101
174.138.58.140:8101
142.93.241.66:8101
188.166.122.215:8101
142.93.17.121:8101
104.248.77.220:8101
104.248.27.246:8101
188.166.60.223:8101
128.199.144.164:8101
104.248.182.148:8101
104.248.182.152:8101
159.203.72.8:8101
178.128.233.252:8101
104.248.43.86:8101
104.248.43.99:8101
178.62.19.156:8101
104.248.75.183:8101
206.81.11.82:8101
206.189.138.82:8101
178.128.225.8:8101
142.93.246.182:8101
104.248.31.169:8101
128.199.144.48:8101
178.128.229.48:8101
128.199.144.196:8101
159.65.40.42:8101
178.128.229.50:8101
138.197.193.229:8101
128.199.144.206:8101
178.128.229.65:8101
159.89.234.65:8101
104.248.43.120:8101
104.248.186.25:8101
128.199.145.212:8101
188.166.15.205:8101
138.68.163.99:8101

