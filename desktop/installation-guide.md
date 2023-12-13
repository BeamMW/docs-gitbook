# Installing Beam Wallet

This document describes the process of installing Beam wallet on:

* [MacOS](installation-guide.md#macos)
* [Linux](installation-guide.md#linux)
* [Windows](installation-guide.md#windows)

As the wallet will be continuously updated, the actual version numbers and SHA values might be different from the screenshot at the moment of reading.

## MacOS

#### 1. User License Agreement

Please read the Disclaimer and End User License Agreement carefully before selecting "agree."

![](.gitbook/assets/image\_2021-05-28\_13-04-34.png)

#### 2. Install Folder

Drag the Beam Wallet icon into the Applications folder to complete the installation.

![](.gitbook/assets/image\_2021-05-28\_13-05-14.png)

#### 3. Launch Beam Wallet

Launch Beam Wallet from the "applications" folder. Please note: Beam is in the approval stages for the desktop wallet app on the Apple store, until then click "open" when the notification below appears:

![](.gitbook/assets/image\_2021-05-28\_13-08-05.png)

### Files and Locations  <a href="#mac-files-and-locations" id="mac-files-and-locations"></a>



| FILE            | LOCATION                                                                                                                                                                          |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Main Executable | /Applications/Beam Wallet.app                                                                                                                                                     |
| Configuration   | N/A                                                                                                                                                                               |
| Logs            | /Users/{your User name}/Library/Application Support/Beam Wallet/logs                                                                                                              |
| Database        | <p>/Users/{your User name}/Library/Application Support/Beam Wallet/&#x3C;version>/wallet.db<br></p><p>/Users/{your User name}/Library/Application Support/Beam Wallet/node.db</p> |
| Settings        | /Users/{your User name}/Library/Application Support/Beam Wallet/settings.ini                                                                                                      |

## Linux

After you have downloaded the file, unzip it. Go to the folder and run `BeamWallet.AppImage`


**Ubuntu 16 is no longer officially supported!**


To run on Ubuntu 16, enter the following commands in Linux terminal:

```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update && sudo apt-get upgrade libstdc++6
```

### Files and Locations  <a href="#linux-files-and-locations" id="linux-files-and-locations"></a>

Files and locations of Beam metadata in your Linux operating system.

| FILE            | LOCATION                                                                                                                                          |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Main Executable | /usr/bin/BeamWallet                                                                                                                               |
| Configuration   | /usr/bin/beam-wallet.cfg                                                                                                                          |
| Logs            | /home/{your User name}/.local/share/Beam Wallet/logs                                                                                              |
| Database        | <p>/home/{your User name}/.local/share/Beam Wallet/&#x3C;version>/wallet.db<br></p><p>/home/{your User name}/.local/share/Beam Wallet/node.db</p> |
| Settings        | /home/{your User name}/.local/share/Beam Wallet/settings.ini                                                                                      |

## Windows

#### 1. Installing Beam Desktop App

Once the application image is downloaded, double-click the icon to start the installation.

When you click on the `Beam-Wallet-6.0.11638.4326.exe` file installer will run. You’ll see a default path to the wallet, which you can change using the action menu. Also, it suggests you read Licenses Terms and Conditions. A default path to the wallet will display on the opened screen.

1. Please read the terms and conditions carefully and click **agree** and install.
2. **Wait** until the installation is complete.
3. Press **run wallet** to start working with the wallet.

![](.gitbook/assets/Screenshot\_87.png)

![](.gitbook/assets/bandicam-2021-05-24-15-41-04-892-\_online-video-cutter.com\_.gif)

### Files and Locations <a href="#windows-files-and-locations" id="windows-files-and-locations"></a>

Files and locations for Windows. Your `wallet.db` file contains wallet metadata, including transaction history and address list.

| FILE            | LOCATION                                                                                                                                          |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Main Executable | \Program Files\Beam Wallet\Beam Wallet.exe                                                                                                        |
| Configuration   | \Program Files\Beam Wallet\beam-wallet.cfg                                                                                                        |
| Logs            | \Users{your User name}\AppData\Local\Beam Wallet\logs                                                                                             |
| Database        | <p>\Users\{your User name}\AppData\Local\Beam Wallet\&#x3C;version>\wallet.db</p><p>\Users\{your User name}\AppData\Local\Beam Wallet\node.db</p> |
| Settings        | \Users{your User name}\AppData\Local\Beam Wallet\settings.ini                                                                                     |
| Dumps           | \Users{your User name}\AppData\Local\Beam Wallet\Beam Wallet.exe0.dmp                                                                             |

