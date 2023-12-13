# Node Settings


#### Commands entered via Command-line do not override the settings of your `beam-node.cfg` file.

The `beam-node.cfg` file loads all the settings not dictated by the Command-line. To revert to the original settings of your`beam-node.cfg`file, manually restart your Beam node.


Users can adjust and configure node settings via the Command Line or by editing the`beam-node.cfg` file (located in the `node_folder`).

### Node settings (parameters)

| `--port (or -p)`     | `Port` to start the server on.                    |
| -------------------- | ------------------------------------------------- |
| `--log_level`        | Log level `[info\|debug\|verbose].`               |
| `--file_log_level`   | File log level `[info\|debug\|verbose].`          |
| `--log_cleanup_days` | Old `logfiles` cleanup period (measured in days). |

### Node options (parameters)

Node options can be plugged directly into the`beam-node.cfg`file or enter them via the command line.

| `--storage`              | Path to the node database file (defaults to `node.db`).                                                                                                                                                                                  |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--miner_key`            | The secret key that attributes the mining rewards mined by the node to your wallet created using CLI wallet `export_miner_key` command with `--subkey=<miner id>` [parameter](https://beamx.gitbook.io/cli-guide/#exporting-miner-key).  |
| `--owner_key`            | A secret key that allows the node to monitor mining rewards accumulated by all the mining nodes associated with this key. Created using CLI wallet `export_owner_key`[command](https://beamx.gitbook.io/cli-guide/#exporting-owner-key). |
| `--pass`                 | The wallet password (both the`miner_key`and `owner_key`are password protected).                                                                                                                                                          |
| --`stratum_port`         | `Port`on which stratum server will scan for incoming connections. `Port=0`if the stratum server is disabled.                                                                                                                             |
| `--stratum_secrets_path` | `Path`to the folder containing stratum certificates.                                                                                                                                                                                     |
| `--peer`                 | Nodes that are available for connection.                                                                                                                                                                                                 |


#### CPU Mining for Beam is not recommended.

Beam uses an Equihash mining algorithm with (150,5) parameters and a customized data path that is inefficiently mined on CPUs.


### Node options (continued)

| `-h` (or `--help`)           | Prints a list of all available options and commands.                                                     |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- |
| `-v` (or `--version`)        | Prints latest project version.                                                                           |
| `--git_commit_hash`          | Prints git commit hash value.                                                                            |
| `--fast_sync`                | Fast sync on/off (override horizons)                                                                     |
| `--print_txo`                | Print TXO movements (create/spend) recognized by the owner key.                                          |
| `--peers_persistent`         | Keep a persistent connection to the specified peers, regardless of ratings                               |
| `--mining_threads`           | a number of mining threads(there is no mining if 0). It works if FakePoW is enabled                      |
| `--pow_solve_time`           | pow solve time. It works if FakePoW is enabled                                                           |
| `--verification_threads`     | number of threads for cryptographic verifications (0 = single thread, -1 = auto)                         |
| `--history_dir`              | Path to the folder where compressed (cut-through) history files are stored. Defaults to the same folder. |
| `--temp_dir`                 | Path to temp folder for compressed history files. Must be on the same volume as history\_dir             |
| `--stratum_use_tls`          | Enable TLS on stratum server                                                                             |
| `--nonceprefix_digits`       | number of hex digits for nonce prefix for stratum client (0..6)                                          |
| `--generate_recovery`        | Recovery file to generate immediately after the start                                                    |
| `--recovery_auto_path`       | Path and file prefix for recovery auto-generation                                                        |
| `--recovery_auto_period`     | Period (in blocks) for recovery auto-generation                                                          |

Make running your local node in fast sync mode easier with the following command:

```
./beam-node --port=10000 --peer=eu-nodes.mainnet.beam.mw:8100,us-nodes.mainnet.beam.mw:8100,ap-nodes.mainnet.beam.mw:8100,ap-hk-nodes.mainnet.beam.mw:8100 --fast_sync=on
```

![](<.gitbook/assets/image (2).png>)
