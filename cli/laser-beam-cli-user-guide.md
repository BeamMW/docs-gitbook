---
description: >-
  Laser Beam is a direct link (or "channel") between two Beam wallets that
  allows instant payments. Laser Beam is not available on Desktop or Mobile
  wallets.
---

# Laser Beam CLI User Guide

To perform an instant transaction, both parties must submit Beam coins to an open Laser Channel. The coins will remain locked while the channel is open, only committing the channel's last state to the blockchain when Laser Channel is closed, thus, completing the payment.

## Create a Laser Channel

{% hint style="info" %}
**The`laser_fee`for both parties is 100 Groth's.**

Both the Sender and Receiver must agree on the`locked_amount`to perform an instant payment.
{% endhint %}

### Opening a Laser Channel between two wallets

Using Alice and Bob as an example, let's go over setting up a direct channel between two wallets.

To create a channel, Alice enters the following command:

```
$ ./beam-wallet laser --laser_receive --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>
```

The output will generate a channel address that Alice can send to Bob, it should be similar to the following:

```
I 2020-06-04.13:23:35.342 Beam Wallet
I 2020-06-04.13:23:35.343 Rules signature: 0-5547a592195f4cd4, 10-2fdbbbb74ac57c55, 20-67131c58aa9a6b85
Enter password: *
I 2020-06-04.13:23:36.039 wallet sucessfully opened...
I 2020-06-04.13:23:36.260 New address generated:

333133cddf316e40f3352e9dae6bae568e97052c1b73531b263a695170658cd13e6
```

Using the channel address he received from Alice, Bob will insert the channel address right after `laser_address`, entering the following command:

```
$ ./beam-wallet laser --laser_open --laser_address <address> --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>
```

If Bob incorrectly entered the values previous command, then Alice will receive an “_Incoming connection with incorrect…_” message on her wallet.

### How it will look in Alice's wallet

```
$ ./beam-wallet-masternet.exe laser --laser_receive --laser_my_locked_amount 1 --laser_remote_locked_amount 2 --laser_fee 100 -n 127.0.0.1:1000
```

#### Open Laser Channel

```
Laser Channels:
channel Id                        |aMy    |aTrg     |state     |fee       |valid till
7a0f1a491fe95f3fe453a298dd62ba12  |1      |2        |Open      |0.000001  |55966
```

### How it will look in Bob's wallet

```
$ ./beam-wallet laser --laser_open --laser_address 333133cddf316e40f3352e9dae6bae568e97052c1b73531b263a695170658cd13e6 --laser_my_locked_amount 2 --laser_remote_locked_amount 1 --laser_fee 100 -n 127.0.0.1:1000
```

#### Open Laser Channel

```
Laser Channels:
channel Id                        |aMy    |aTrg     |state     |fee       |valid till
7a0f1a491fe95f3fe453a298dd62ba12  |2      |1        |Open      |0.000001  |55966
```

## Sending Beam

Now that Alice and Bob have established a Laser Channel between their wallets, they can send direct payments

### Establish Laser Channel connection

Using `laser_listen`, Bob will confirm that the channel ID is open by entering the following command:

```
$ ./beam-wallet laser --laser_listen [channel id 1,channel id 2, ... channel id N]
```

### Send a direct payment

To send her Beam coins to Bob, Alice will enter the following command:

```
$ ./beam-wallet-masternet laser --laser_send <amount in beam> --laser_channel <channel id>
```

After sending the payment, Alice will receive a message about changing her channel balance.

### How it will look in Alice's wallet

#### Sending Bob Beam coins

```
$ ./beam-wallet laser --laser_send 0.2 --laser_channel 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000
```

#### Payment Sent and changing channel balance

```
D 2020-06-04.14:19:35.166 ### Bbs mesage out ###
D 2020-06-04.14:19:35.166 Channel:7a0f1a491fe95f3fe453a298dd62ba12 state Open. Last Revision: 2. My balance: 220000000 / Total balance: 300000000
D 2020-06-04.14:19:35.167 Save channel: 7a0f1a491fe95f3fe453a298dd62ba12
I 2020-06-04.14:19:35.200 Update finished: 7a0f1a491fe95f3fe453a298dd62ba12
D 2020-06-04.14:19:35.487 OnMined() diff: 0
D 2020-06-04.14:19:35.488 Receiver::OnComplete
```

### How it will look in Bob's wallet

#### Verifying Channel ID

```
$ ./beam-wallet laser --laser_listen 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000»
```

#### Payment received

```
D 2020-06-04.14:19:35.166 ### Bbs mesage out ###
D 2020-06-04.14:19:35.166 Channel:7a0f1a491fe95f3fe453a298dd62ba12 state Open. Last Revision: 2. My balance: 220000000 / Total balance: 300000000
D 2020-06-04.14:19:35.167 Save channel: 7a0f1a491fe95f3fe453a298dd62ba12
I 2020-06-04.14:19:35.200 Update finished: 7a0f1a491fe95f3fe453a298dd62ba12
D 2020-06-04.14:19:35.487 OnMined() diff: 0
D 2020-06-04.14:19:35.488 Receiver::OnComplete
```

## View open Laser Channels

To print out a list of all open Laser Channels and channel details, enter the following command:

```
./beam-wallet laser --laser_channels_list 
```

The ouput should be similar to following:

```
Laser Channels:

channel Id                        |aMy       |aT        |state     |fee       |valid till
7a0f1a491fe95f3fe453a298dd62ba12  |2.2       |0.8       |Open      |0.000001  |57464
b9236abe78ab5747ca955189df079d2b  |1.0000005 |1.0000005 |Closed    |0.000001  |0
```

## Closing and deleting a Channel

While both users must be online when first creating a Laser Channel, users have multiple options for closing an open channel.

Continuing with our examples of Bob and Alice, we will go over closing a Laser Beam Channel.

### Closing a channel when both parties online

Using`laser_listen` Bob will check the channel ID's current state. Bob enters the following command:

```
$ ./beam-wallet laser --laser_listen 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000
```

To close the channel, Alice will enter the following command:

```
$ ./beam-wallet laser --laser_close 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000
```

### Closing a channel when only one party is online

Bob and Alice have agreed to close the open Laser Beam Channel but Bob's internet connection can be unreliable, and is often offline fordays at a time.

Alice can close the open channel without Bob by entering following command:

```
$ ./beam-wallet laser --laser_drop 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000
```

{% hint style="danger" %}
**`laser_close`and`laser_drop`commands only closes the connection between the two Beam wallets.**

**Laser Beam Channels with "Closed" status are deletable after 24 hours or 1440 blocks.**

**Channels with "Waiting" or "OpenFailed" status are ready for immediate deletion. To delete the Laser Beam Channel, follow the steps below**:
{% endhint %}

### Delete closed Laser Beam Channel

To delete a channel from your wallet database, enter the following command:

```
 ./beam-wallet-masternet laser --laser_delete <channel id 1,channel id 2, ... channel id N
```

Alice deletes the Laser Beam Channel from her`wallet.db`file by entering:

```
$ ./beam-wallet laser --laser_delete 7a0f1a491fe95f3fe453a298dd62ba12 -n 127.0.0.1:1000
```
