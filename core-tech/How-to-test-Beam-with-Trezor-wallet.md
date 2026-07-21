## Preparing
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