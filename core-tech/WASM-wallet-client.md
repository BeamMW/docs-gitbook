# Overview
WASM wallet client is a thin wrapper around Beam client library built into WASM using Emscripten toolchain. This wrapper allows to run BEAM wallet inside any browser supporting WebAssembly and it provides the regular [BEAM wallet API](api/README.md) to communicate with it from external(javascript) code. 

Starting from version 7.3.13702 all network types are handled by a single  [npm](https://www.npmjs.com/package/beam-wasm-client) package.


 Older versions are available via `npm` for different network types:
* [mainnet](https://www.npmjs.com/package/beam-wasm-client)
* [testnet](https://www.npmjs.com/package/beam-wasm-client-testnet)
* [dappnet](https://www.npmjs.com/package/beam-wasm-client-dappnet)
* [masternet (developer)](https://www.npmjs.com/package/beam-wasm-client-masternet)

# API
WASM wallet client module exports the following classes:
* [WasmWalletClient](#WasmWalletClient) - the main object provides a set of static and member methods to control the wallet. It could be used to implement BEAM wallet as a browser extension

	Method | Description
	--- | ---
	[Constructors](#Constructors) |  Creates new regular wallet object
	[Headless constructors](#Headless-constructors) |  Creates new headless wallet object
	[startWallet](#startWallet) |  Starts the wallet in the background thread
	[stopWallet](#stopWallet) |  Asynchronously stops the wallet running in the background
	[isRunning](#isRunning) |  Checks if the wallet is running
	[isHeadless](#isHeadless) |  Checks if the wallet is `headless`, i.e. without master and owner keys
	[sendRequest](#sendRequest) |  Sends [API](api/README.md) request to the wallet 
	[subscribe](#subscribe) |  Subscribes for API responses
	[unsubscribe](#unsubscribe) |  Unsubscribes from response notifications 
	[setSyncHandler](#setSyncHandler) |  Sets synchronization handler, allows tracking sync progress
	[setApproveSendHandler](#setApproveSendHandler) |  Sets handler which allows to approve or reject any send operation initiated by DAPPs
	[setApproveContractInfoHandler](#setApproveContractInfoHandler) |  Sets handler which allows to approve or reject any operation which requires user's attention from application shader
	[createAppAPI](#createAppAPI) |  Asynchronously creates new application wallet API for given application
	[importRecovery](#importRecovery) |  Asynchronously imports recovery data
	[GeneratePhrase](#GeneratePhrase) |  Generates new seed phrase
	[IsAllowedWord](#IsAllowedWord) |  Checks if given word is in the dictionary of the words allowed to be used in seed phrases
	[IsValidPhrase](#IsValidPhrase) |  Validates given seed phrase
	[ConvertTokenToJson](#ConvertTokenToJson) |  Converts given BEAM address to json
	[ConvertJsonToToken](#ConvertJsonToToken) |  Packs transaction parameters presented as JSON object into BEAM address
	[MountFS](#MountFS) |  Asynchronously mounts WASM filesystem to the roor of IndexDB
	[CreateWallet](#CreateWallet) |  Creates new wallet database
	[DeleteWallet](#DeleteWallet) |  Deletes given wallet database from IndexDB
	[IsInitialized](#IsInitialized) |  Ensures that database was created
	[CheckPassword](#CheckPassword) |  Tests if given password fits to the database
* [AppAPI](#AppAPI) - a proxy API object, it gives limited wallet API for external web applications, which want to work with BEAM wallet 

	Method | Description
	--- | --- 
	[callWalletApi](#callWalletApi) | Allows to call wallet API methods from the application
	[setHandler](#setHandler) | Sets handler to receive a response for API request

* [AppAPICallback](#AppAPICallback) - a callback object for applications, it allows the wallet to control the action which the application wants to perform.
        
	Method | Description
	--- | --- 
	[sendApproved](#sendApproved) | Approves send request from application
	[sendRejected](#sendRejected) | Rejects send request from application
	[contractInfoApproved](#contractInfoApproved) | Approves contract call 
	[contractInfoRejected](#contractInfoRejected) | Rejects contract call

# WasmWalletClient

## GeneratePhrase
```javascript
WasmWalletClient.GeneratePhrase()
```
Generates new seed phrase

### Return value
* seed phrase, a string of 12 words from the dictionary separated by ` `

### Example 
```javascript
 var phrase = Module.WasmWalletClient.GeneratePhrase()
 console.log('seed phrase is: ', phrase);
 // OUTPUT: seed phrase is:  legend hurdle erode ribbon pass exit basket doll sorry version muscle brain
```

## IsAllowedWord
```javascript
WasmWalletClient.IsAllowedWord(word : String)
```
Checks if given word is in the dictionary of the words allowed to be used in seed phrases

### Parameters
* `word` : the word to be checked 

### Return value
* `true` if `word` is in the dictionary otherwise `false`

### Example 
```javascript
  if (Module.WasmWalletClient.IsAllowedWord('hurdle')) {
      console.log("Word is allowed");
  }
```

## IsValidPhrase
```javascript
WasmWalletClient.IsValidPhrase(phrase : String)
```
Validates given seed `phrase` 

### Parameters
* `phrase` : a string of words separated by ` `

### Return value
* `true` if seed `phrase` is valid otherwise `false`


## ConvertTokenToJson
```javascript
WasmWalletClient.ConvertTokenToJson(token : String)
```
Converts given BEAM address to json

### Parameters
* `token` : address to to unpack data

### Return value
* json object with address parameters unpacked from given string

### Notes
* in beam address(token) is binary packed set the key-value parameters presented as `base58` string


## ConvertJsonToToken
```javascript
WasmWalletClient.ConvertJsonToToken(json : String)
```
Packs transaction parameters presented as JSON object into BEAM address

### Parameters
* `json` : parameters of the transaction

### Return value
* `base58` encoded string of packed parameters

## MountFS
```javascript
WasmWalletClient.MountFS(callback : function)
```
Asynchronously mounts WASM filesystem to the roor of IndexDB

### Parameters
* `callback` : mounting completion handler, if an error occurred, it will be provided as a parameter to this function.

### Return value
* none

### Notes
* This method should be called before any action which implies work with filesystem

### Example 
```javascript
 Module.WasmWalletClient.MountFS(function(error) {
   if (error != null) {
     console.log("mounted");
     var walletClient = new Module.WasmWalletClient("/beam_wallet/wallet.db",
                                                  "123",
                                                  "eu-node01.masternet.beam.mw:8200");
   }
 }
```

## CreateWallet
```javascript
WasmWalletClient.CreateWallet(seedPhrase : String, database : String, password : String)
```
Creates new wallet database

### Parameters
* `seedPhrase`: seed pharse for the wallet
* `database` : path to the database in IndexedDB
* `password` : password to the new wallet database

### Return value
* none

### Notes
* Ensure that `MountFS()` has been called before

### Example 
```javascript
 let phrase = Module.WasmWalletClient.GeneratePhrase();
 Module.WasmWalletClient.CreateWallet(phrase, "/beam_wallet/wallet.db", "123");
```

## DeleteWallet
```javascript
WasmWalletClient.DeleteWallet(database : String)
```
Deletes given wallet database from IndexDB

### Parameters
* `database` : path to the database file

### Return value
* none

### Notes
* Ensure that `MountFS()` has been called before

## IsInitialized
```javascript
WasmWalletClient.IsInitialized(database : String)
```
Ensures that database was created

### Parameters
* `database` : the path to the database

### Return value
* `true` if database is created and initialized, otherwise `false`

### Notes
* Ensure that `MountFS()` has been called before


## CheckPassword
```javascript
WasmWalletClient.CheckPassword(database : String, password : String, callback : function)
```
Tests asynchronously if given password fits to the database

### Parameters
* `database` : path to the database
* `password` : password to test
* `callback` : asynchronously returns the result of the test

### Return value
* none

### Notes
* Ensure that `MountFS()` has been called before

### Example
```javascript
  Module.WasmWalletClient.CheckPassword("/beam_wallet/wallet.db", "13", (res) => {
  if (res)
    console.log("Password is correct")
  else
    console.log("Password is not correct")
  })
```

## Constructors
```javascript
WasmWalletClient(database : String, password : String, nodeURL : String);
WasmWalletClient(database : String, password : String, nodeURL : String, network : Network);
```
Creates new wallet client object

### Parameters
* `database` : path to encrypted database in browser's IndexDB
* `password` : password to the database
* `nodeURL` : URL to BEAM node to communicate with
* `network` : network type, acceptable values are, available from version 7.3.13702

```javascript
Module.Network.mainnet;
Module.Network.masternet;
Module.Network.testnet;
Module.Network.dappnet;
```

### Return value
* object of the wallet client

### Notes
* wallet client can communicate with node over Web Sockets only, ensure that node located by `nodeURL` has WebSocket proxy enabled
* ensure that `MountFS()` has been called before
* if no `network` param is passed `mainnet` client is created

### Example 
```javascript
 var mainnetWalletClient = new Module.WasmWalletClient("/beam_wallet/wallet.db",
                                                "123",
                                                "eu-node01.mainnet.beam.mw:8200");
 var dappnetWalletClient = new Module.WasmWalletClient("/beam_wallet/wallet.db",
                                                "123",
                                                "eu-node01.dappnet.beam.mw:8200",
                                                Module.Network.dappnet);
```


## Headless constructors
```javascript
WasmWalletClient(nodeURL : String);
WasmWalletClient(nodeURL : String, network : Network); // available from version 7.3.13702
```
Creates new headless wallet client object. It doesn't have a master key. The wallet with such a database can communicate with the node, but cannot make transactions or detect any UTXO event. It can generate public keys, but they all are temporary and provided only for the reason to have viewer access to dapps

### Parameters
* `nodeURL` : URL to BEAM node to communicate with
* `network` : network type, acceptable values are

```javascript
Module.Network.mainnet;
Module.Network.masternet;
Module.Network.testnet;
Module.Network.dappnet;
```

### Return value
* object of the wallet client

### Notes
* wallet client can communicate with node over Web Sockets only, ensure that node located by `nodeURL` has WebSocket proxy enabled
* headless wallet doesn't require `MountFS()` to be called before

### Example 
```javascript
 var mainnetWalletClient = new Module.WasmWalletClient("eu-node01.mainnet.beam.mw:8200");
 var walletClient = new Module.WasmWalletClient("eu-node01.masternet.beam.mw:8200", Module::Network::masternet);
```

## startWallet
```javascript
function startWallet()
```
Starts the wallet in the background thread

### Notes
* wallet client can communicate with node over Web Sockets only, ensure that node located by `nodeURL` has WebSocket proxy enabled

## stopWallet
```javascript
function stopWallet(callback : function)
```
Asynchronously stops the wallet running in the background

### Parameters
* `callback` : calls when wallet has stopped. In this callback it is safe to delete the wallet database

### Return value
* none

### Example 
```javascript
  wc.stopWallet(()=> {
    console.log("is running: " + wc.isRunning()) // false
  }
```

## isRunning
```javascript
function isRunning()
```
Checks if the wallet is running

### Parameters
* none

### Return value
* `true` if the wallet is running, `false` otherwise

## isHeadless
```javascript
function isHeadless()
```
Checks if the wallet is `headless`, i.e. without master and owner keys

### Parameters
* none

### Return value
* `true` if the wallet is headless, `false` otherwise


## sendRequest
```javascript
function sendRequest(jsonRequest : String)
```
Sends [API](api/README.md) request to the wallet 

### Parameters
* `jsonRequest` : API request

### Return value
* none

### Notes
* to get response you have to [subscribe](#subscribe) before

### Example 
```javascript
 walletClient.sendRequest(JSON.stringify({
     jsonrpc: '2.0',
     id: 5,
     method: 'wallet_status'
 }));
```

## subscribe
```javascript
function subscribe(callback : function)
```
Subscribes for API responses

### Parameters
* `callback` : function which is called when response arrived

### Return value
* index of the subscription

### Example 
```javascript
  var i = walletClient.subscribe((r)=> {
      console.log("response: " + r)
  });
```

## unsubscribe
```javascript
function unsubscribe(index : Number)
```
Unsubscribes from response notifications 

### Parameters
* `index` : index of the subscription

### Return value
* none


## setSyncHandler
```javascript
function setSyncHandler(handler : function)
```
Sets synchronization handler, allows to track sync progress

### Parameters
* `handler` : called each time wallet notifies about sync progress

### Return value
* none

### Example 
```javascript
 walletClient.setSyncHandler((done, total) => {
    console.log("sync [" + done + "/" + total + "]");
 });
```

## setApproveSendHandler
```javascript
function setApproveSendHandler(handler : function)
```
Sets handler which allows to approve or reject any send operation initiated by DAPPs

### Parameters
* `handler` : function called each time application wants to send assets from the wallet

### Return value
* none

### Example 
```javascript
 walletClient.setApproveSendHandler((request, info, cb)=>{
     console.log("Request: " + request);
     console.log("Info: " + info);
     cb.sendApproved(request);
     //cb.sendRejected(request);
 }) 
```

## setApproveContractInfoHandler
```javascript
function setApproveContractInfoHandler(handler : function)
```
Sets handler which allows to approve or reject any operation which requires user's attention from application shader

### Parameters
* `handler` 

### Return value
* none

### Example 
```javascript
 walletClient.setApproveContractInfoHandler((request, info, amounts, cb)=>{
     console.log("Request: " + request);
     console.log("Info: " + info);
     cb.contractInfoApproved(request);
     //cb.contractInfoRejected(request);
 }) 
```

## createAppAPI
```javascript
function createAppAPI(appid : String, appname : String, callback : function);
function createAppAPI(apiver: String, minapiver : String, appid : String, appname : String, callback : function);
function createAppAPI(apiver: String, minapiver : String, appid : String, appname : String, callback : function, privilegeLevel : Integer);
```
Asynchronously creates new application wallet API for given application

### Parameters
* `apiver` : desired version of API, acceptable values are strings like: "6.0", "7.3" or "current" if you want to use the latest version
* `minapiver` : a fallback version of API in case if desired API version is inaccessible
* `appid` : ID of the application
* `appname` : the name of the app
* `callback` : the callback with API object in the case of success
* `privilegeLevel` : level of privilege for app shader running via requested API, possible values are `0`, `1`, `2`, the higher value the higher privilege, available from version 7.3.13702

### Example 
```javascript
 console.log("calling API...");
 wc.createAppAPI("appid", "appname", (api)=>{
    api.setHandler((r)=> {
       console.log("API response: " + r)
    })
    api.callWalletApi(JSON.stringify({
       jsonrpc: '2.0',
        id: 5,
        method: 'wallet_status'
    }));        
 });
```

## importRecovery
```javascript
function importRecovery(recoveryData: Uint8Array, callback: function(error, done, total))
```
Asynchronously imports recovery data

### Parameters
* `recoveryData` : downloaded recovery data
* `callback` : callback function, which allows to handle errors via `error` parameter and to report progress using `done` and `total`

### Example 
```javascript
 // download function should be defined by user
 download("recovery.bin", function(error, data) {
    if (error == null) {
        console.log("downloaded recovery")
        walletClient.importRecovery(data, function(error, done, total) {
            if (error == null) {
                console.log(`Restoring ${done}/${total}`)
            } else {
                console.log(`Failed to recover: ${error}`)
            }
        });
        
    } else {
        console.log("failed to download recovery")
    }

});
```

# AppAPI
## callWalletApi
## setHandler

# AppAPICallback
## sendApproved
## sendRejected
## contractInfoApproved
## contractInfoRejected
