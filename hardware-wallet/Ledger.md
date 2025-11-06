---
description: Ledger Hardware Wallet support
---

# Ledger

We currently do not support smart contract transactions for the Hardware Wallets.

## Ledger

Ledger allows for 3rd-party developers to develop embedded apps that support custom coins. The app should be submitted for official review, and once they find it compliant to their safety standards - it will become available for installation via their official software (Ledger Live).

Our app is submitted, and is waiting for the review. However, given the complexity of Beam, it looks unlikely to see Beam accepted anytime soon (as of 2023-01).

Meanwhile our app can be installed as unofficial release. This is called sideloading. Some (but not all!) Ledger devices support this.

#### Ledger Nano S
This device is outdated (discontinued), but we support it. However its very weak performance may impact user experience. It may take minutes (!!) to sign a transaction. Please be patient with it.

#### Ledger Nano S Plus
This is the recommended device. Relatively inexpensive, yet powerful enough. Fully supports our functionality.

#### Ledger Nano X
Unfortunately this device does not support sideloading. Will have to wait until Ledger officially accepts our app.

#### Ledger Stax
No development environment available yet for this device.


# Instructions for the host machine

We support Linux, Windows and Mac, i.e. all the platforms on which the Beam Wallet is already supported. No additional tools/prerequisites are required. Just plug the HW wallet, it should be detected by our desktop wallet automatically.


# Installation instructions

Once the Beam app will be officially accepted by Ledger, it'll be possible to install it on the Ledger secure device using the official software (Ledger Live).
Until then our unofficial release should be installed.

The best and easiest way to do this is using our CLI wallet. It contains the embedded apps for all the supported devices, and allows to install it on any supported device. Just run the following command line:

`beam-wallet hid_install`

It will automatically find the connected supported device, and install the appropriate app on it. This also ensure the embedded app version is fully compatible with the desktop wallet version.

Nevertheless there's an option to download and sideload the app manually, as described in the next section.

## Alternative installation method

Overall the process of side-loading is documented [here](https://docs.radixdlt.com/main/user-applications/ledger-app-sideload.html). In essence the user should download python, and use the specified command line (included in our releases) to load the Beam application on the device.

### Note for Linux users
On Linux, however, the Beam desktop wallet should have appropriate permissions to connect to the HW wallet.

Desktop wallet can be run with elevated privileges (`sudo`) but this is not recommended. Instead it's possible to enable device access for all the users in the system.
To allow this the following file should be downloaded, and copied `/etc/udev/rules.d` into directory. After the device is re-plugged the new permissions should be in effect.
* [ledger-hid.zip](https://github.com/BeamMW/app-beam/files/10475866/ledger-hid.zip)
(unzip it before copying into target directory)

#### Stable releases

| Release Date| Commit  | Api Ver | Nano S | Nano S Plus | Remarks |
|---|---|---|---|---|---|
| 2023-01-23| 58704c5cf328c47e8bde59a165c3c1d93d6c7244  | 3 | [download](https://github.com/BeamMW/app-beam/files/10475776/58704c5-nanos.zip) | [download](https://github.com/BeamMW/app-beam/files/10475779/58704c5-nanosp.zip) | |
| 2023-01-29| bc7585a346d15061b42d7706f39b68b9b395c573  | 3 | [download](https://github.com/BeamMW/app-beam/files/10529521/bc7585a-nanos.zip) | [download](https://github.com/BeamMW/app-beam/files/10529522/bc7585a-nanosp.zip) | |
| 2023-02-03| d00f0fecc7df294986104af3dd57cf38725b44e0  | 3 | [download](https://github.com/BeamMW/app-beam/files/10576697/d00f0fe-nanos.zip) | [download](https://github.com/BeamMW/app-beam/files/10576696/d00f0fe-nanosplus.zip) | Fixed incorrect endpoint for shielded txs |
| |   |   |   |   | |

# Usage

## Implications of device auto-lock

HW wallets usually auto-lock if idle for some time. This makes sense of course (otherwise, if left unlocked and unattended, it could be used by any stranger or intruder to steal your funds).

However in Beam the HW wallet is used not only to send funds, but also to receive. Hence the desktop wallet won't be able to receive funds while the device is locked. Our desktop wallet (both CLI and UI) gives an appropriate message if it can't access the HW wallet, and if/when the user unlocks it - the transaction will proceed. But if the user is not around then there will be no one to unlock the device, and all transactions will end stuck.

At the moment we recommend the following options:

Receiver: if you don't expect to receive eventual transactions, then we recommend keeping the auto-lock feature. If, however, you prefer to be able to receive transactions constantly, then consider disabling the auto-lock feature.

Sender: If you expect the receiver to be online, then send transactions as usual. Otherwise it's always possible to use offline transactions types (public offline, or max privacy), since those are non-interactive transactions, and the receiver is not involved.

