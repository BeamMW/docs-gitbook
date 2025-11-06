---
description: Hardware Wallet Support Overview
---

# Hardware Wallets

Beam supports integration with popular **hardware wallets** to enhance security and protect private keys from exposure on your host machine.  
This section provides an overview of hardware wallet support, installation, and usage.

Currently, Beam supports **Ledger** and **Trezor** devices with varying levels of functionality.

> ⚠️ Smart contract transactions are **not yet supported** for hardware wallets.

---

## Supported Devices

| Manufacturer | Model | Support Status | Notes |
|---------------|--------|----------------|-------|
| **Ledger** | Nano S | ✅⚠️ Supported (slow performance) | Discontinued but compatible |
|  | Nano S Plus | ✅ Recommended | Full support and stable performance |
|  | Nano X | 🚧 Not supported | No sideloading support until official approval |
|  | Stax | 🚧 Not supported | No development environment available |
| **Trezor** | Model One / Model T | 🧩 Experimental | Under investigation for unofficial release |

---

## Getting Started

To use a hardware wallet with Beam:

1. Make sure you have the **latest version** of the Beam Desktop or CLI wallet installed.  
2. Connect your hardware device to your computer.
3. The Beam wallet should automatically detect and communicate with it — no extra configuration is required on **Windows**, **macOS**, or **Linux**.

> 🐧 **Linux users:** You may need to set appropriate USB permissions to allow Beam access to your hardware wallet.  
> Follow the [Linux setup instructions](https://github.com/BeamMW/app-beam/files/10475866/ledger-hid.zip) to install the necessary udev rules.

---

## Installation & Setup Guides

- [**Ledger Setup Guide →**](./Ledger.md)  
  Learn how to install the Beam app on Ledger devices, including sideloading instructions and device-specific notes.

- [**Trezor Setup Guide →**](./Trezor.md)  
  Step-by-step guide to preparing and using Trezor devices with Beam.

---

## Usage Notes

Hardware wallets are used in Beam not only for **sending**, but also for **receiving** transactions.  
If your hardware device **auto-locks**, Beam will not be able to process incoming transactions until you unlock it.

### Recommendations

- **Receiver:**  
  If you expect to receive transactions continuously, consider disabling auto-lock temporarily.  
  Otherwise, keep it enabled for security.

- **Sender:**  
  Use standard transactions when the receiver is online.  
  If not, use **offline** or **max privacy** transaction types — these are non-interactive and do not require the receiver’s participation.

---

## Learn More

- [Beam Wallet Downloads](https://beam.mw/downloads)
- [Beam GitHub Organization](https://github.com/BeamMW)
- [Telegram](https://t.me/BeamPrivacy)
