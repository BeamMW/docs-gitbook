# Understanding Beam Logs

## Understanding Beam logs

Beam logs are files that record the events that occur within the Beam software and the Beam network. These files contain vital information that is very helpful to Beam support if you encounter any technical difficulties.

### Beam Log Locations


_Find CLI wallet and Node logs in the same folder as the binaries._


| Operating System | Desktop Wallet Log Location                                        |
| ---------------- | ------------------------------------------------------------------ |
| Mac              | /Users/{your\_user\_name}/Library/Application Support/Beam Wallet/ |
| Windows          | \Users\\{your\_user\_name}\AppData\Local\Beam Wallet               |
|  Linux           | /home/{your\_user\_name}/.local/share/Beam Wallet                  |

For a complete list of file locations see `:ref: Files and Locations`

### Node Logs

Each log starts with the Rules Signature. Rules Signature is the hash of the Consensus Rules and must be compatible to connect with your desired network. Consensus Rules for Testnet and Mainnet logs are different. View the relevant Rules Signature for each network here `:ref: rules_signature`

Beam logs have a simple structure. The first field is the severity level, followed by a timestamp and log message:

_Log Severity +——————–+—————–+ | I | Info | +——————–+—————–+ | W | Warning | +——————–+—————–+ | E | Error | +——————–+—————–+ | D | Debug | +——————–+—————–+_

The sample below shows the start of a new node:

```
I 2018-12-31.16:48:58.838 Rules signature: 7e16d65b64ef2fbb
I 2018-12-31.16:48:58.986 starting a node on 10000 port...
I 2018-12-31.16:48:58.996 Node ID=5c8f92a1cfaee337
I 2018-12-31.16:48:58.996 Initial Tip: 0-0000000000000000
I 2018-12-31.16:48:58.996 Requesting block 0-0000000000000000
I 2018-12-31.16:48:58.997 PI 0000000000000000--0.0.0.0 New
I 2018-12-31.16:48:58.997 PI 0000000000000000--0.0.0.0 Address changed to 23.239.24.209:8201
I 2018-12-31.16:48:58.999 stratum server listens to 0.0.0.0:10002
```

In this case, the node connects to the first peer, _23.239.24.209:8201_, and downloads the initial Tip at _height 0_. It then requests the matching block. In this specific example, the node also starts the Stratum server.
