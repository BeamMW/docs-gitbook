Starting with v6.0 BEAM wallet supports API versioning using the `--api_version` option. Please choose the corresponding API version in the list below to get the description of the available methods.

* [version 6.0](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.0)
* [version 6.1](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.1)
* [version 6.2](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.2)
* [version 7.0](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v7.0)
* [version 7.1](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v7.1)
* [version 7.2](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v7.2)
* [version 7.3](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v7.3)
* [version 7.4](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v7.4)

The sections below apply to the API versions starting with 6.0 if the other is not explicitly stated.

## General

Wallet API has the same structure as Node Stratum API protocol (JSON RPC 2.0 over TCP connection) and should have an online connection to the node. However, it can work over HTTP using the `--use_http=1` option. Send POST requests to `http://x.x.x.x:port/api/wallet` in this case.

## Running Wallet API

Place `wallet-api` binary located in the `wallet` folder near your `wallet.db` and run it. You can specify options in CLI or using `wallet-api.cfg` file. Minimal options required are your `wallet.db` password (`--pass`) and node (`-n=`) to connect to. Please notice that `node.db` by default is expected to be located in the current working directory (`$PWD`). You can change this using the `--wallet_path=` option.

### Wallet API general options

```
  -h [ --help ]                         list of all options
  -p [ --port ] arg (=10000)            port to start server on
  -n [ --node_addr ] arg                address of node
  --wallet_path arg (=wallet.db)        path to the wallet file
  --pass arg                            password for the wallet
  --use_http arg (=0)                   use JSON RPC over HTTP
  --ip_whitelist arg                    IP whitelist
  --log_cleanup_days arg (=5)           old logfiles cleanup period(days)
  --node_poll_period arg (=0)           Node poll period in milliseconds. Set
                                        to 0 to keep a connection. Anyway poll
                                        period would be no less than the
                                        expected rate of blocks if it is less
                                        then it will be rounded up to block
                                        rate value.
  --enable_assets                       enable confidential assets transactions
  --enable_lelantus                     enable Lelantus MW transactions
  --api_version                         specify API version. Available since v6.0. Acceptable values: current, 6.0, 6.1 &c.

User authorization options:
  --use_acl arg (=0)                use Access Control List (ACL)
  --acl_path arg (=wallet_api.acl)  path to ACL file

TLS protocol options:
  --use_tls arg (=0)                use TLS protocol
  --tls_cert arg (=wallet_api.crt)  path to TLS certificate
  --tls_key arg (=wallet_api.key)   path to TLS private key
  --tls_request_cert (=false)       request client's certificate for verification for client authentication
  --tls_reject_unauthorized (=true) server will reject any connection which is not authorized with the list of supplied CAs
```

### wallet-api.cfg file

The `wallet-api.cfg` file is another way to supply options for the API server. All the options listed above are supported. The `wallet-api.cfg` file is expected to be located in the current working directory (`$PWD`). Here is an example of the `wallet-api.cfg` file:

```
# password for the wallet
pass=1

# address of node
node_addr=127.0.0.1:8888

# port to start server on
port=10000

# path to wallet file
wallet_path=/home/user/wallet.db

# use JSON RPC over HTTP
use_http=1
```

## Running API Server in TCP mode

By default API launches in TCP mode. For example: 

`./wallet-api --node_addr=172.104.249.212:8101 --pass=123`

## Accessing API Server via TCP

Here is an example NodeJS code to get all UTXOs from the Wallet API server that is running in TCP mode

```js
var net = require('net');

var client = new net.Socket();
client.connect(10000, '127.0.0.1', function() {
	console.log('Connected');
	client.write(JSON.stringify(
		{
			jsonrpc: '2.0',
			id: 123,
			method: 'get_utxo',
			params: {}
		}) + '\n');
});

var acc = '';

client.on('data', function(data) {
	acc += data;

	// searching for \n symbol to find end of response
	if(data.indexOf('\n') != -1)
	{
		var res = JSON.parse(acc);

		console.log('Received:', res);

		client.destroy(); // kill client after server's response
	}
});

client.on('close', function() {
	console.log('Connection closed');
});
```

## Running API Server in HTTP mode 

Run via cli with '--use_http=1': 

`./wallet-api --node_addr=172.104.249.212:8101 --pass=123 --use_http=1`

Or add 'use_http=1' to the wallet-api.cfg to enable HTTP mode.


## Accessing API Server via HTTP

You can perform POST requests using CURL. Here is an example to get the current *wallet status*.

```
curl -d '{"jsonrpc":"2.0","id":1,"method":"wallet_status"}' 
     -H "Content-Type: application/json" 
     -X POST http://x.x.x.x:port/api/wallet
```

## TLS encryption

Add `--use_tls=1` to enable TLS encryption. You have supply the valid 3rd-party certificate or create a self signed certificate. Pass them to the API server using `--tls_cert` and `--tls_key` options.

For testing you can download the sample certificate and key files from [here](https://beamx.gitbook.io/beam-node-user-guide/mining-mode/mining-using-external-miner#certificate-and-keys)  or clicking the links below:  

[Certificate  File](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-MahNa0IaGxTr8GkB5rz%2F-Mb6iKWRWPQD7ljMB61A%2F-Mb6q79H8fH8zbfxJrei%2Fstratum.crt?alt=media&token=29cad8ec-2911-446e-b1d4-991401e19d0d)  
[Certificate  Private Key  File](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-MahNa0IaGxTr8GkB5rz%2F-Mb6iKWRWPQD7ljMB61A%2F-Mb6qjg8lMk1JOdjnnpG%2Fstratum.key?alt=media&token=4c1d1228-fa6d-4f50-a7d0-020966529204)

## IP whitelisting

Add `--ip_whitelist=192.168.0.1,192.168.0.2` parameter to restrict access to the API server to certain IP addresses. Connections from other IP address would be rejected.

## Users authorization

API methods can have access rights control if you enable _Access Control List_ (ACL) using `--use_acl=1` option. Below is the list of all API methods with the access rights required to call the respective method:

```
create_address       - write access
validate_address     - read access
addr_list            - read access
delete_address       - write access
edit_address         - write access
tx_send              - write access
tx_status            - read access
tx_split             - write access
tx_asset_issue       - write access
tx_asset_consume     - write access
tx_asset_info        - write access
tx_cancel            - write access
get_utxo             - read access
tx_list              - read access
wallet_status        - read access
get_asset_info       - read access
generate_tx_id       - read access
export_payment_proof - read access
verify_payment_proof - read access
calc_change          - read access
invoke_contract      - write access
process_invoke_data  - write access
block_details        - read access
```
 
ACL file should look like a list with the access keys and `read/write` rights:

```
472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67 : write
bd39333a66a8b7cb3804b5978d42312c841dbfa03a1c31fc2f0627eeed6e43f2 : read
f287176bdd517e9c277778e4c012bf6a3e687dd614fc552a1ed22a3fee7d94f2 : read
```

Don't forget to send user `key` in every JSONRPC request to the API if access control is enabled:

``` json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "wallet_status",
    "key": "bd39333a66a8b7cb3804b5978d42312c841dbfa03a1c31fc2f0627eeed6e43f2"
}
```

## API Errors Handling

Errors are returned as:

```
  error: {
    code: -32003,
    data: "It's not your own address.",
    message: 'Invalid address.'
  }
```

where 

* `code` is a unique identifier of the error
* `message` textual description of the error
* `data` is the contextual data for the error, usually a more detailed description for general error codes

To get the list of the possible error codes check the respective API version.

## Assets Support

v6.0+ API implements confidential assets support. To ensure backward compatibility by default API does not return asset entities (coins, balances, txs &c). Asset operations (send, receive, issue, consume &c.) are disabled as well and if requested for an asset would fail. Also, all incoming transactions that involve assets would be rejected by the Wallet API as well. If you do not enable assets support your old code should be able to use the new API without any changes.

**To enable assets start your API server with the `--enable_assets` option or set this flag to true in the config file.**

Methods that can return asset entities (for example `wallet_status` or `get_utxo`) might have different return formats if assets are enabled. Please check the respective methods' documentation for details.

In most method responses an additional `asset_id` field is returned by default even if assets support is not enabled. It is always 0 for BEAM and can be safely ignored.

## IPFS Support

String with v6.3 BEAM comes with built-in IPFS support. Please refer to the following pages for more details

* [BEAM IPFS Support](https://github.com/BeamMW/beam/wiki/BEAM-IPFS-Support)
* [v6.3 API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.3)
