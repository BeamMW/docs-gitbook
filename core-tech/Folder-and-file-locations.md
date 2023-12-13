# Desktop Wallet app

## General points to mention:
* The default location of the Desktop Wallet app can be modified during the installation process (Windows only).
* The default Database location for the Desktop Wallet app can be changed setting the `appdata` parameter to `beam-wallet.cfg` (Windows only).
* Memory dump files are generated on Windows only. A dedicated memory dump file is created per each crash case.

## Windows

| File | Location |
|-----|-----|
| Main Executable | `\Program Files\Beam\Beam Wallet.exe`| 
| Configuration | `\Program Files\Beam\beam-wallet.cfg` |
| Logs | `\Users\\{your User name}\AppData\Local\Beam Wallet\logs` | 
| Database | `\Users\{your User name}\AppData\Local\Beam Wallet\wallet.db (node.db)` | 
| Dumps | `\Users\{your User name}\AppData\Local\Beam Wallet\Beam Wallet.exe0.dmp` |

## Mac

| File | Location |
|-----|-----|
| Main Executable | `/Applications/Beam Wallet.app` |
| Configuration | N/A |
| Logs | `/Users/{your User name}/Library/Application Support/Beam Wallet/logs` |
| Database | `/Users/{your User name}/Library/Application Support/Beam Wallet/wallet.db (node.db)` |

## Linux

| File | Location |
|-----|-----|
| Main Executable | `/usr/bin/BeamWallet` |
| Configuration | `/usr/bin/beam-wallet.cfg` |
| Logs | `/home/{your User name}/.local/share/Beam Wallet/logs` |
| Database |`/home/{your User name}/.local/share/Beam Wallet/wallet.db (node.db)` |

# Node or CLI wallet

## All Platforms (small differences apply, see below)
| File | Location |
|-----|-----|
| Main Executable folder | User can unpack the archive in any folder to his convenience, *all the files/folders mentioned below are located within this folder* |
| Main Executable file | `beam-node` or `beam-wallet` |
| Configuration file | `beam-node.cfg` or `beam-wallet.cfg` |
| Logs folder | `logs` | 
| Database | `node.db` or `wallet.db` | 
| Dumps (on Windows only) | `beam-node.exe0.dmp` |


