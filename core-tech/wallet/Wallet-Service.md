Beam wallet service allows to implement lightweight wallets using BEAM API.

## How to build

Wallet service and related projects are supposed to be run on Linux only. It is possible to build everything for Windows and using Windows but it is not oficially supported. Releases are also provided only for Linux.

1. Install required tools to build the generic BEAM project. Refer [BEAM build instructions](https://github.com/BeamMW/beam/wiki/How-to-build) for detals. For example if you're using Ubuntu 18.04 Desktop you need to execute steps `1. Install dependencies` & `2. Install cmake` from the `Ubuntu 18.04 Desktop` section.

2. Install golang. You need at least v1.13.0 to build the project. If the relevant package is provided with your OS it is better to use it. For example `sudo dnf install golang` on Fedora. If your OS doesn't not provide recent golang packages (like Ubuntu 18.04) use instructions [from the official website](https://golang.org/doc/install).

3. Clone the repository

    ```
    git clone https://github.com/BeamMW/wallet-service.git
    ```

4. Checkout the necessary branch. We use the `master` branch for development. To be able to connect to the testnet you have to checkout the `testnet` branch. To be able to connect to the mainnet you have to checkout the `mainnet` branch.

    ```
    cd wallet-service
    git checkout testnet
    ```

5. Init submodules
    
    ```
    git submodule update --init --recursive
    ```

6. Build `wallet-service` & `sbbs-monitor`. Use build commands from the [BEAM build instructions](https://github.com/BeamMW/beam/wiki/How-to-build) according to your OS. 

    For example for Fedora
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

    For Ububtu 18.04 Desktop
    ```
    export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j4
    ```

    After the build process completes you would have `wallet-service(-testnet/-masternet)` & `sbbs-monitor(-testnet/-masternet)` binaries in the `wallet-service/service` folder. Mainnet binaries do not have the `-BRANCHNAME` postfix.

7. Build `service-balancer`

   ```
   cd service-balancer && go build && cd ..
   ```

    After the build process completes you would have the `service-balancer` binary in the `wallet-service/service-balancer` folder. N.B. At the moment manual builds do not add `-masternet/-testnet` postfix to the `service-balancer` executable.

## Prebuilt binaries

You can get the latest prebuilt wallet service binaries here.

1. Testnet

   * https://builds.beam-mw.com/files/testnet/latest/Release/linux/service-balancer-testnet.tar.gz
   * https://builds.beam-mw.com/files/testnet/latest/Release/linux/wallet-service-testnet.tar.gz
   * https://builds.beam-mw.com/files/testnet/latest/Release/linux/sbbs-monitor-testnet.tar.gz

2. Masternet

   * https://builds.beam-mw.com/files/masternet/latest/Release/linux/service-balancer-masternet.tar.gz
   * https://builds.beam-mw.com/files/masternet/latest/Release/linux/wallet-service-masternet.tar.gz
   * https://builds.beam-mw.com/files/masternet/latest/Release/linux/sbbs-monitor-masternet.tar.gz

## Deployment

At the moment it is advised to use our public API enpoints until the deployment procedure is stabilized and made public. Docker would be available as well as the detailed deployment guide. 

## <a name="how-to-try">How to try</a>

Currently the Wallet Service is deployed for masternet & testnet and provides the following public Service Balancer endpoints

1. Testnet

   wss://web-wallet-testnet.beam.mw/ws

2. Masternet

   wss://web-wallet-masternet.beam.mw/ws

## API Guide

Wallet service exposes its API only via JSONRpc v2.0 over WebSockets. To access the API you should

1. Login to the balancer and get the API enpoint. Check the [how to try section](#how-to-try) for the list of our public balancer endpoints.
2. Communicate with the API enpoint using the Wallet Service API.

Too keep user 'secrets' at the user's side Wallet Service doesn't ask for the seed phrase. Instead it expects the client to implement a set of methods that deal with secrets/outputs creation/signing, i.e. user-side keykeeper. We provide default WebAssembly implementation of the keykeeper for JS/Web usage. Check the [key keeper](#wasm-key-keeper) section for details.

### Service Balancer API

Service Balancer API provides the following methods:

- [login](#login)
- [logout](#logout)
- [subscribe](#subscribe)
- [unsubscribe](#unsubscribe)

In addition to the Web Sockets API Service Balancer provides the following HTTP(S) endpoints:

- [/](#balancer-hello)
- [/status](#balancer-status)

### login

Login to the service balancer and get the wallet service endpoint

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "login",
    "params":
    {
        "WalletID": "wallet id",
        "SbbsAddress": "",
        "SbbsAddressPrivate": ""
    }
}
```
 * WalletID - wallet id, get it from `keykeeper.getWalletID` method based on the generated seed 
 * SbbsAddress - reserved for future use, leave empty for now
 * SbbsAddressPrivate - reserved for future use, leave empty for now
  
```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": {
        "endpoint": "web-wallet-testnet.beam.mw?service=30000"
    }
}
```
   * endpoint - returns wallet service endpoint used to access Walet Service API & Beam Wallet API. You should manually append the web socket protocol prefix `wss://` in front of it.

### subscribe

Subscribe to the BBS address notifications to be notified about incoming transaction. Reserved for future use

### unsubscribe

Unsubsribe from the BBS address notifications. Reserved for future use

**<a name="balancer-hello">balancer root (/)</a>**

Just a 'hello world' endpoint. Returns the `Hello! This is the wallet service balancer` string. It is intended to be used for alive checks.

**<a name="balancer-status">/status</a>**

Status endpoint. Returns json with the internal service balancer methods.

```
/status?secret=secret
```

Expects secret (password) to be provided. Secret is configured in the balancer's `config.json` file. For testent & masternet this endpoint is public and can be accessed by anyone:

   * https://web-wallet-testnet.beam.mw/status?secret=secret
   * https://web-wallet-masternet.beam.mw/status?secret=secret

### Wallet Service API

Wallet Service API provides the following methods:

- [create_wallet](#create_wallet)
- [open_wallet](#open_wallet)
- [change_password](#change_password)
- [calc_change](#calc_change)
- [release](#release)
- [ping](#ping)
- [Wallet API methdos](#wallet-api-methods)

Wallet service expect a set of keykeper-related methods to be implemented by the client. Check the [Wallet Service Client keykeeper methods](#keykeeper-client-metods) for details.

### create_wallet 

Create new wallet

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "create_wallet",
    "params":
    {
        "pass": "wallet password",
        "ownerkey": "wallet owner key",
    }
}
```
 * pass - initial wallet password
 * ownerkey - wallet owner key, get it from key keeper

```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": "wallet id"
}
```
   * result - returns wallet id used to identify the wallet in other API calls   
    
### open_wallet

Open wallet with the given ID and password.


```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "open_wallet",
    "params":
    {
        "id": "<wallet ID>",
        "pass": "<wallet password>"
    }
}
````
   * id - wallet id returned by the [create_wallet](#create_wallet) method or by the `keykeeper.getWalletID` method
   * pass - wallet password used to create wallet

```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": "adc18c1afbb74674b5853fb4350df70a"
}
```
   * result - unique session id. Reserved for the future use

### change_password

Change wallet password. Wallet must be already opened when this method is called.

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "change_password",
    "params":
    {
        "new_pass": "new password"
    }
}
````
   * new_pass - new wallet password


```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": "done"
}
```
   * result - "done" on success

### release

Close (release) the currently opened wallet. Usually it is not necessary to call this method. Wallet would be automatically closed as soon as the web socket connection is released.

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "release"
}

````
   This method does not have any parameters. 

```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": "done"
}
```
   * result - "done" on success

### calc_change

Service method used to calculate change for the given transaction amount

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "calc_change",
    "params":
    {
        "amount": 100
    }
}
```
   * amount - transaction amount in groth

```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": {
        "change": 499999900,
        "change_str": "499999900"
    }
}
```
   * change - transaction change in groth
   * change_str - string representation of the `change` parameter

### ping

Just a convenience ping (are you alive?) method. It is recommended not to use this method but to rely on the underlying websocket infrastructure which already implements ping-pong control frames.

```json
<--
{
    "jsonrpc": "2.0",
    "id": "123",
    "method": "ping"
}

````
   This method does not have any parameters. 

```json
-->
{
    "jsonrpc": "2.0",
    "id": "123",
    "result": "pong"
}
```
   * result - "pong" on success

### <a name="wallet-api-methods">wallet API methods</a>

Wallet service also implements all the BEAM Wallet Protocol API methods. Please consult the [Beam wallet protocol API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API) section for details.

### <a name="wasm-key-keeper">WebAssembly Keykeeper</a>

[Get latest testnet version](https://builds.beam.mw/testnet/2020.06.16/MinSizeRel/wasm/wasm-key-keeper.tar.gz)

#### Keykeeper initialization example:

```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
	Module().then((Module) => console.log(`Key Keeper module successfully loaded.`))
</script>
</html>
```

#### <a name="keykeeper-client-metods">Wallet Service client keykeeper methods</a>

- [get_kdf](#keykeeper-methods)
- [get_slots](#keykeeper-methods)
- [create_output](#keykeeper-methods)
- [sign_receiver](#keykeeper-methods)
- [sign_sender](#keykeeper-methods)
- [sign_split](#keykeeper-methods)

These methods are supported by the default WASM keykeeper. Wallet Service Client should just implement a simple proxy:

```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
    Module().then((Module) => {
        var connection = null;
        var endpoint = 'ws://url-to-service';
        var seed = Module.KeyKeeper.GeneratePhrase();
	var keykeeper = new Module.KeyKeeper(seed);
        connection = new WebSocket(endpoint);
        connection.onmessage = e =>  {
	    var data = JSON.parse(e.data);
            if(data.method) {
	        var res = keykeeper.invokeServiceMethod(e.data);
		connection.send(res);
	    }
       }
    })
</script>
</html>
```

Method | Description | Arguments | Return
------------ | ------------- | ------------ | -------------
getOwnerKey | get owner key from password | wallet password (string) | owner key (string)
ConvertTokenToJson | get token data | token (string) | PeerID (string), PeerWalletIdentity (string), TransactionType (number), tx_id (number)
getSendToken | generate token | sbbsAddressHex (string), identityStrHex (string), amountBase64 (string) | key (string)
getWalletID | get wallet id | - | wallet id (string)
GeneratePhrase | generate seed phrase | - | seed phrase (string)
IsValidPhrase | seed phrase validation | - | is seed valid (boolean)


#### Minimal Wallet implementation example:

```
﻿<!doctype html>
<html lang="en">
<head>
	<title>Beam Wallet Client</title>

	<meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

	<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
</head>

<body>
	<div class="container">
		<h1 class="text-center">Beam Wallet Client</h1>
		<hr>

		<hr>
		<div id="login-panel">
		  <span class="badge badge-danger">status: logging in, please wait...</span>
		</div>

		<div id="connecting-panel" style="display: none;">
			<span class="badge badge-danger">status: connecting to the Wallet Service, please wait...</span>
		</div>

		<form id="create-wallet-panel" style="display: none;">
			<div class="form-group">
				<span class="badge badge-success">status: connected to the Wallet Service</span>
			</div>
			<div class="form-group">
				<label for="seed-phrase-input">Generated seed phrase:</label>
				<input disabled type="text" class="seed-phrase-input form-control" placeholder="Enter seed phrase">
			</div>
			<div class="form-group">
				<label for="wallet-pass-input">Enter wallet password:</label>
				<input type="password" class="wallet-pass-input form-control" placeholder="Enter wallet password" value="456">
			</div>
			<div class="form-group">
				<button type="button" class="create-wallet-button btn btn-primary">
					<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" style="display: none;"></span>
					Create new wallet
				</button>
			</div>
		</form>

		<form id="open-wallet-panel" style="display: none;">
			<div class="form-group">
				<span class="badge badge-success">status: connected to the Wallet Service</span>
			</div>
			<div class="form-group">
				<label for="wallet-pass-input">Enter wallet password:</label>
				<input type="password" class="wallet-pass-input form-control" placeholder="Enter wallet password" value="456">
			</div>
			<div class="form-group">
				<button type="button" class="open-wallet-button btn btn-primary">
					<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" style="display: none;"></span>
					Open wallet
				</button>
				or
				<button type="button" class="delete-wallet-button btn btn-danger">
						<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" style="display: none;"></span>
						Delete wallet
				</button>
			</div>
		</form>

		<form id="wallet-panel" style="display: none;">
			<div class="form-group">
				<span class="badge badge-success">status: connected to the Wallet Service</span>
			</div>
			<div class="form-group">
				<div>Available: <span class="available"></span></div>
				<div>Sending: <span class="sending"></span></div>
				<div>Receiving: <span class="receiving"></span></div>
			</div>
			<div class="form-group">
				Current height: <span class="current_height"></span>
			</div>
			<hr>
			<h3>Send</h3>
				<div class="form-group">
					<label for="receiver-address-input">Enter receiver address:</label>
					<input type="text" class="receiver-address-input form-control" placeholder="Enter receiver address" value="19f3b39d925ce49f19f67c3787b54bfe4414d490f0acf42b58527a23608427bbb34">
				</div>
				<div class="form-group">
					<label for="amount-input">Enter amount (GROTH):</label>
					<input type="text" class="amount-input form-control" placeholder="Enter amount">
				</div>
				<button type="button" class="send-button btn btn-danger">
					Send
				</button>				
			<hr>
			<h3>Transactions</h3>
			<table class="table transactions">
				<thead>
					<tr>
						<th>#</th>
						<th>created</th>
						<th>receiver</th>
						<th>sender</th>
						<th>amount</th>
						<th>status</th>
					</tr>
				</thead>
				<tbody></tbody>
			</table>
			<hr>
			<h3>Addresses</h3>
			<table class="table addresses">
				<thead>
					<tr>
						<th>#</th>
						<th>address</th>
						<th>comment</th>
						<th>create time</th>
						<th>duration</th>
						<th>expired</th>
						<th>own</th>
					</tr>
				</thead>
				<tbody></tbody>
			</table>
			<div id="push-panel" style="display:none">
			<button class="btn btn-secondary" id="push-enable" style="display:none">Enable Push Notifications</button>
			<button class="btn btn-secondary" id="push-disable" style="display:none">Disable Push Notifications</button>
			<span id="push-blocked" style="display:none">Push notifications are blocked by user</span>
			</div>
			<hr>
			<h3>UTXO</h3>
			<table class="table utxo">
				<thead>
					<tr>
						<th>#</th>
						<th>amount</th>
						<th>id</th>
						<th>maturity</th>
						<th>status</th>
						<th>type</th>
					</tr>
				</thead>
				<tbody></tbody>
			</table>
			<hr>
			<button type="button" class="close-wallet-button btn btn-secondary">
				Close wallet
			</button>
		</form>		
	
	</div>
</body>
	<script src='wasm-key-keeper.js'></script>
	<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
	<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
	<script type="module">
	import WebPush from './webpush.js'
	Module().then(function(Module) {
		let keykeeper = null
		var loginConnection = null
		var connection = null
		var webpush = null
		var defaddr = null

		login()

		// If you change walletID/seed you should ALWAYS relogin
		// If you lose loginConnection or connection, ALWAYS relogin
		// Both loginConnection & connection should be active
		function login() {
			//keykeeper = initKeyKeeper("giraffe usage sad arrest sound chimney file select monster head clown ask")
			//localStorage.seed = "giraffe usage sad arrest sound chimney file select monster head clown ask"
			//localStorage.id = keykeeper.getWalletID()

			if (localStorage.id && localStorage.seed) {
				console.log(`Stored WalletID:${localStorage.id}`)
				console.log(`Stored Seed: ${localStorage.seed}`)
				keykeeper = initKeyKeeper(localStorage.seed)
				console.log(`Keykeeper created`);
			} else {
				let seed = Module.KeyKeeper.GeneratePhrase()
				console.log(`New Seed: ${seed}`)
				keykeeper = initKeyKeeper(seed)
				console.log(`New WalletID: ${keykeeper.getWalletID()}`)
				$('#create-wallet-panel .seed-phrase-input').val(seed)
			}

			console.log("Logging in...")
			loginConnection = new WebSocket('ws://127.0.0.1:20000/ws')

			loginConnection.onclose = () => {
				console.log('login_ws: connection lost')
				loginConnection = null
				relogin()
			}

			loginConnection.onerror = error => {
				console.log(`login_ws: connection error: ${JSON.stringify(error)}`)
			}

        	loginConnection.onopen = () => {
				console.log('login_ws: connected...')

				let loginParams = {
					WalletID: keykeeper.getWalletID()
				}

				loginConnection.send(JSON.stringify({
					jsonrpc: "2.0",
					id: 123,
					method: "login",
					params: loginParams
				}))
			}

			loginConnection.onmessage = e => {
				let data = JSON.parse(e.data)

				if (data.result) {
					console.log(`login_ws: result is ${JSON.stringify(data.result)}`)
					if (data.id === 123) {
						console.log("login_ws: OK, endpoint is ", data.result.endpoint)
						let endpoint = ['ws://', data.result.endpoint].join('');
						start(endpoint)
					}
				} else {
					console.log("login_ws: failed")
					if (data.error) {
						console.log(`login_ws: error code:${data.error.code} text:${data.error.data}`)
					}
				}
			}
		}

		function relogin () {
			console.log('relogin in 5 seconds...')
			setTimeout(() => login(), 5000)
			showPanel('#connecting-panel')
		}

		function start(url) {
			console.log("STARTING...")
			showPanel('#connecting-panel')
			connection = new WebSocket(url)

			connection.onerror = error => {
				console.log(`connection error: ${JSON.stringify(error)}`)
			}

			connection.onopen = () => {
				console.log('Connected to the Wallet Servce...')
				if (localStorage.id) {
					showPanel('#open-wallet-panel')
				} else {
					showPanel('#create-wallet-panel')
				}
			}

			connection.onclose = () =>
			{
				console.log('connection lost...')
				connection = null
				loginConnection.close()
			}
		}

		function showPanel(id) {
			['#login-panel', '#connecting-panel', '#create-wallet-panel', '#open-wallet-panel', '#wallet-panel'].forEach(name => $(name).hide())
			$(id).show()
		}

		function initKeyKeeper(seed) {
			return new Module.KeyKeeper(seed);
      	}

      	function onkeykeeper(data) {
			console.log(`<<< keykeeper request: ${data}`)
			let res = keykeeper.invokeServiceMethod(data)
			console.log(`>>> keykeeper response: ${res}`)
			connection.send(res);
		}

      	/*function refresh()
		{
			if(localStorage.id) {
	  			showPanel('#open-wallet-panel')
			}
			else
			{
				$('#create-wallet-panel .seed-phrase-input').val(Module.KeyKeeper.GeneratePhrase())
				showPanel('#create-wallet-panel')
			}
		}*/

      	$('#create-wallet-panel .create-wallet-button').click(() =>
		{
			let seed = $('#create-wallet-panel .seed-phrase-input').val()
			console.log(`Creating new wallet with seed phrase: ${seed}`)

			var walletPass = $('#create-wallet-panel .wallet-pass-input').val()
			var ownerKey = keykeeper.getOwnerKey(walletPass)
			console.log('ownerKey is: data:application/octet-stream;base64,' + ownerKey)

			$('#create-wallet-panel .create-wallet-button')
				.attr('disabled', true)
				.find('.spinner-border').show()

			connection.onmessage = e =>  {
				console.log(`got response: ${e.data}`)

				var data = JSON.parse(e.data)

				if(data.result && data.result.length)
				{
					console.log(`result is: ${data.result}`)

					localStorage.clear()
					localStorage.seed = seed
					localStorage.id = data.result

					$('#create-wallet-panel .create-wallet-button')
						.attr('disabled', false)
						.find('.spinner-border').hide()

					showPanel('#open-wallet-panel')
				}
				else if(data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if(data.method)
					onkeykeeper(e.data)
			}

			connection.send(JSON.stringify(
			{
				jsonrpc:'2.0',
				id: 0,
				method: 'create_wallet',
				params:
				{
					pass: walletPass,
					ownerkey: ownerKey
				}
			}))
		})

		$('#open-wallet-panel .delete-wallet-button').click(() => {
			if (confirm('Are you sure want do delete current wallet data?')) {
				pushNotifyUnsub(defaddr)
				localStorage.clear()
				connection.close()
			}
		})

		$('#wallet-panel .close-wallet-button').click(() => {
			if (confirm('Are you sure want do close the wallet?')) {
				connection.close()
			}
		})

		$('#open-wallet-panel .open-wallet-button').click(() => {
			console.log(`Opening wallet with seed phrase: ${localStorage.seed}`)
			let walletPass = $('#open-wallet-panel .wallet-pass-input').val()
			let walletID = keykeeper.getWalletID()

			console.log(`walletID: ${walletID}`)
			connection.onmessage = e => {
				console.log(`open wallet got response: ${e.data}`)
				let data = JSON.parse(e.data)
				if (data.result && data.result.length) {
					console.log(`wallet session: ${data.result}`)
					showPanel('#wallet-panel')
					showWalletStatus()
				} else if (data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if (data.method)
					onkeykeeper(e.data)
			}

			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 0,
				method: 'open_wallet',
				params: {
					id: localStorage.id,
					pass: walletPass
				}
			}))
		})

		function showWalletStatus() {
			// unsubscribe if any
      		pushNotifyUnsub(defaddr)

			connection.onmessage = e => {
				let data = JSON.parse(e.data)
				if (data.result) {
					$("#wallet-panel .available").text(data.result.available)
					$("#wallet-panel .sending").text(data.result.sending)
					$("#wallet-panel .receiving").text(data.result.receiving)
					$("#wallet-panel .current_height").text(data.result.current_height)
					showAddresses()
				} else if (data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if (data.method)
					onkeykeeper(e.data)
			}
			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 0,
				method: 'wallet_status'
			}))
		}

		function showAddresses() {
			connection.onmessage = e => {
				let data = JSON.parse(e.data)
				if(data.result)	{
					let body = $('#wallet-panel .addresses > tbody').empty()
					data.result.forEach && data.result.forEach((item, index) => {
						if (index === 0) defaddr = item
						console.log(JSON.stringify(item))
						body.append(`<tr>
										<td>${index+1}</td>
										<td>${item.address}</td>
										<td>${item.comment}</td>
										<td>${item.create_time}</td>
										<td>${item.duration}</td>
										<td>${item.expired}</td>
										<td>${item.own}</td>
									</tr>`)
					})
					if (defaddr) {
						showPush()
					}
					showUtxo()
				}
				else if(data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if(data.method)
					onkeykeeper(e.data)
			}

			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 0,
				method: 'addr_list',
				params: {
					own: true
				}
			}))
		}
		
		async function pushNotifySub(addr) {
      		return new Promise((resolve, reject) => {
				if (!addr) reject()
				return webpush.notifyServer(true, loginConnection, {
					SbbsAddress: keykeeper.getSbbsAddress(addr.own_id_str),
					SbbsAddressPrivate: keykeeper.getSbbsAddressPrivate(addr.own_id_str),
					ExpiresAt: addr.create_time + addr.duration
				})
			})
		}

		async function pushNotifyUnsub(addr) {
      		return new Promise((resolve, reject) => {
      			if (!addr) reject()

      			if (!webpush.userSubscribed) {
					console.log('unsub: no previous subscription')
					reject()
					return
				}
      			return webpush.notifyServer(false, loginConnection, {
					SbbsAddress: keykeeper.getSbbsAddress(addr.own_id_str),
					SbbsAddressPrivate: keykeeper.getSbbsAddressPrivate(addr.own_id_str)
				})
			})
		}

		function showPush() {
			webpush = new Proxy(new WebPush(), {
				set: function(obj, prop, value) {
					if (prop === "subscription") {
						if (value) {
							$("#push-enable").hide()
							$("#push-disable").show()
						} else {
							$("#push-enable").show()
							$("#push-disable").hide()
						}
						$("#push-panel").show()
					}
					obj[prop] = value
					return true
				}
			})

			if (webpush.blockedByUser) {
				console.log("notifications blocked by user")
				$("#push-blocked").show()
				$("#push-panel").show()
				// may be push was disabled in settings
				// remove any previous subscriptions
				// TODO: check the same on service worker start (if possible)
				pushNotifyUnsub(defaddr)
			} else {
				console.log("notifications enabled, notifying server");
				(async () => {
					if (await webpush.register(loginConnection)) {
						await pushNotifySub(defaddr)
						console.log("Push notification - OK")
					}
				})()
			}

			$("#push-enable").click(() => {
				(async () => {
					if (await webpush.subscribe()) {
						await pushNotifySub(defaddr)
					}
				})()
				return true
			})

			$("#push-disable").click(() => {
				(async () => {
					await pushNotifyUnsub()
					await webpush.unsubscribe()
				})()
				return true
			})
		}

		function showUtxo() {
			connection.onmessage = e => {
				let data = JSON.parse(e.data)
				if (data.result) {
					let body = $('#wallet-panel .utxo > tbody').empty()
					data.result.forEach && data.result.forEach((item, index) => body.append(`<tr>
							<td>${index + 1}</td>
							<td>${item.amount}</td>
							<td>${item.id}</td>
							<td>${item.maturity}</td>
							<td>${item.status_string}</td>
							<td>${item.type}</td>
							</tr>`))
					showTransactions()
				} else if (data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if (data.method)
					onkeykeeper(e.data)
			}

			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 0,
				method: 'get_utxo'
			}))
		}

		function showTransactions() {
			connection.onmessage = e => {
				let data = JSON.parse(e.data)
				if (data.result) {
					let body = $('#wallet-panel .transactions > tbody').empty()
					data.result.forEach && data.result.forEach((item, index) => {
						body.append(`<tr>
							<td>${index + 1}</td>
							<td>${new Date(item.create_time * 1000).toLocaleString()}</td>
							<td>${item.receiver.slice(0, 32)}...</td>
							<td>${item.sender.slice(0, 32)}...</td>
							<td>${item.value}</td>
							<td>${item.status_string}</td>
							</tr>`)
					})
				} else onkeykeeper(e.data)
				if ($('#wallet-panel').is(":visible"))
					setTimeout(showWalletStatus, 5000)
			}

			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 0,
				method: 'tx_list'
			}))
		}

		$('#wallet-panel .send-button').click(() => {
			let addr = $('#wallet-panel .receiver-address-input').val()
			if (addr.length === 0) {
				alert('Please, enter valid address.')
				return
			}

			let amount = $('#wallet-panel .amount-input').val()
			if (isNaN(parseInt(amount))) {
				alert('Please, enter valid amount.')
				return
			}

			if (confirm(`Are you sure want send ${amount} groth to ${addr}?`)) {
				$('#wallet-panel .amount-input').val('')
				send(parseInt(amount), addr)
			}
		})

		function send(amount, address) {
			console.log(`!!! sending ${amount} groth to ${address}...`)
			connection.onmessage = (e) => {
				let data = JSON.parse(e.data)
				if (data.result)
					console.log(`result is ${data.result}`)
				else if (data.error)
					console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
				else if (data.method)
					onkeykeeper(e.data)
			}
			connection.send(JSON.stringify({
				jsonrpc: "2.0",
				id: 123,
				method: "tx_send",
				params: {
					value: amount,
					fee: 100,
					address: address,
					comment: "thank you!"
				}
			}))
		}

        function createNewTestAddr() {
			connection.send(JSON.stringify({
				jsonrpc: '2.0',
				id: 123,
				method: 'create_address',
				params:
						{
							expiration: '24h',
							comment: 'John Smith'
						}
			}))
		}
	})
	</script>
</html>
```

## Reference Wallet Implementation

We provide the reference web wallet implementation that combines all the described APIs together and yields the working chrome extension. 

* Check out the [web-wallet repository](https://github.com/BeamMW/web-wallet) for the source code

* Install the testnet [chrome extension](https://chrome.google.com/webstore/detail/beam-web-wallet-testnet/ilhaljfiglknggcoegeknjghdgampffk?hl=en) 