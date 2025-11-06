---
description: Trezor Hardware Wallet support
---

# Trezor

We currently do not support smart contract transactions for the Hardware Wallets.


### Trezor

According to Trezor they don't consider accepting new coins (not clones of existing ones) to their official release.
Currently we investigate the possibility to add our app to an unofficial release.

# Instructions for the host machine

We support Linux, Windows and Mac, i.e. all the platforms on which the Beam Wallet is already supported. No additional tools/prerequisites are required. Just plug the HW wallet, it should be detected by our desktop wallet automatically.

### Note for Linux users
On Linux, however, the Beam desktop wallet should have appropriate permissions to connect to the HW wallet.

Desktop wallet can be run with elevated privileges (`sudo`) but this is not recommended. Instead it's possible to enable device access for all the users in the system.
To allow this the following file should be downloaded, and copied `/etc/udev/rules.d` into directory. After the device is re-plugged the new permissions should be in effect.


### Installation


### Install **Git** and **Python3** (with pip3)
- Debain/Ubuntu: `sudo apt-get -y install git python3 python3-pip`  
- Windows/MacOS: download from https://www.python.org/downloads

### Install Trezor Bridge
Go to https://wallet.trezor.io/#/bridge, download and install.

### Install Protobuf Compiler
Go to https://github.com/protocolbuffers/protobuf/releases and download the latest `protoc` for your OS.  
Extract it and add `<path to protoc folder>/bin` path to the system `PATH`: `export PATH=$PATH:/<path to protoc folder>/bin`.


### Build Firmware Loader
Run the following commands (don't use `sudo` prefix on Windows):
```
git clone https://github.com/BeamMW/python-trezor.git
cd python-trezor
git checkout beam
git submodule update --init --recursive --force

sudo pip3 install protobuf click requests mnemonic construct ecdsa pyblake2 typing_extensions
python3 setup.py prebuild
```

## Install the firmware
- Wipe your Trezor device using `python3 ./trezorctl wipe-device` command.
- Go to https://github.com/BeamMW/trezor_beam_minimal_wallet/releases and download the latest `firmware.bin`.
- Restart the device to enter bootloader mode (video how to do it https://www.youtube.com/watch?v=xVBiSFTx0qQ).
- Call `python3 ./trezorctl firmware-update -f <path to firmware folder>/firmware.bin` to install firmware.
### Windows
- Install `trezorctl` https://wiki.trezor.io/Installing_trezorctl_on_Windows .
- Restart the device to enter bootloader mode (video how to do it https://www.youtube.com/watch?v=xVBiSFTx0qQ).
- Call `trezorctl firmware-update -f <path to firmware folder>/firmware.bin` to install firmware.

## Test Beam with Trezor
- Go to https://builds.beam.mw/#trezor_build and download/install the latest build.
- Connect your device, go to https://trezor.io/start, create a new wallet or recover with your seed phrase.
- Run installed **Beam Wallet** and push `create new Trezor wallet` button.
- Agree with generating **Owner Key** on Trezor device and wait, it usually takes about 15 sec.
![image](https://user-images.githubusercontent.com/1101448/65770926-c5d87f80-e13f-11e9-9095-a9fbac692917.png)
- Remember generated password from the `Key Password` page and enter it in the Beam Wallet.
![image](https://user-images.githubusercontent.com/1101448/65770789-7e51f380-e13f-11e9-899e-33dc09d96787.png)
- Hurray, the wallet is initialized, now try to send/receive beams to test how it works...


# Usage

## Implications of device auto-lock

HW wallets usually auto-lock if idle for some time. This makes sense of course (otherwise, if left unlocked and unattended, it could be used by any stranger or intruder to steal your funds).

However in Beam the HW wallet is used not only to send funds, but also to receive. Hence the desktop wallet won't be able to receive funds while the device is locked. Our desktop wallet (both CLI and UI) gives an appropriate message if it can't access the HW wallet, and if/when the user unlocks it - the transaction will proceed. But if the user is not around then there will be no one to unlock the device, and all transactions will end stuck.

At the moment we recommend the following options:

Receiver: if you don't expect to receive eventual transactions, then we recommend keeping the auto-lock feature. If, however, you prefer to be able to receive transactions constantly, then consider disabling the auto-lock feature.

Sender: If you expect the receiver to be online, then send transactions as usual. Otherwise it's always possible to use offline transactions types (public offline, or max privacy), since those are non-interactive transactions, and the receiver is not involved.