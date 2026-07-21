# Beam Web Wallet Starter Kit [working draft]

Web wallet starter allows you easily integrate Beam with your service, create a bunch of wallets and manage them... (TODO)
## Web wallet scheme
Browser + Key Keeper <==> Web Service
(TODO: here will be detailed scheme).

Let's do a simple integration step by step with the examples.

## Obtain binaries
You can download prebuild binaries from our server with nightly builds [https://builds.beam-mw.com/master/](https://builds.beam-mw.com/master/) or you can build all the parts yourself, here are build instructions (TODO)

At the end, you need `wallet-service-masternet`, `wasm-key-keeper.js` and `wasm-key-keeper.wasm`

## Load Key Keeper to the client
Put the code near `wasm-key-keeper.js` and start any static server.
```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
	Module().then(() => console.log(`Key Keeper module successfully loaded.`))
</script>
</html>
```
If all is OK you will see `Key Keeper module successfully loaded.` message in the browser console.

## Connect to the Wallet Service
Run the service with the following parameters `wallet-service-masternet -n eu-node04.masternet.beam.mw:8100 --port 8080`.
```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
	Module().then(() => 
	{
		console.log(`Key Keeper module successfully loaded.`)
		
		let connection = new WebSocket('ws://127.0.0.1:8080')
		connection.onopen = () => console.log(`Connected to the Wallet Servce.`)
	})
</script>
</html>
```
We are using WebSockets here, because we need two-way communication with the service and service can request any keykeeper method, to sign transaction for example.

## Init Key Keeper with a seed phase
```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
	Module().then(Module => 
	{
		console.log(`Key Keeper module successfully loaded.`)
		
		let connection = new WebSocket('ws://127.0.0.1:8080')
		connection.onopen = () => 
		{
			console.log(`Connected to the Wallet Service.`)

			// generate seed phrase
			let seed = Module.KeyKeeper.GeneratePhrase()
			console.log(`New Seed: ${seed}`)

			// init Key Keeper with the seed phrase
			let keykeeper = new Module.KeyKeeper(seed)
		}
	})
</script>
</html>
```

## Create wallet
To create wallet you have to call `create_wallet` jsonrpc api method with `pass` and `ownerkey` parameters.
```js
{
	jsonrpc:'2.0',
	id:'<request ID>',
	method:'create_wallet',
	params:
	{
		pass:'<wallet password>'
		ownerkey:'<owner key>',
	}
}
```
Here is an example:
```html
<!DOCTYPE html>
<html>
<script src='wasm-key-keeper.js'></script>
<script type="text/javascript">
    Module().then(Module => 
    {
        console.log(`Key Keeper module successfully loaded.`)
        
        let connection = new WebSocket('ws://127.0.0.1:8080')
        connection.onopen = () => 
        {
            console.log(`Connected to the Wallet Service.`)

            // generate seed phrase
            let seed = Module.KeyKeeper.GeneratePhrase()
            console.log(`New Seed: ${seed}`)

            // init Key Keeper with the seed phrase
            let keykeeper = new Module.KeyKeeper(seed)

            connection.onmessage = e => 
            {

                let data = JSON.parse(e.data)

                if(data.result)     console.log(`Wallet is successfully created, your ID: ${data.result}`)
                else if(data.error) console.log(`error occured, code:${data.error.code} text:${data.error.data}`)
                else if(data.method) connection.send(keykeeper.invokeServiceMethod(e.data))
            }

            let walletPassword = '12345678'
            let ownerKey = keykeeper.getOwnerKey(walletPassword)
            console.log(`Owner Key: ${ownerKey}`)

            // create a wallet request
            connection.send(JSON.stringify({
                jsonrpc: '2.0',
                id: 0,
                method: 'create_wallet',
                params:
                {
                    pass: walletPassword,
                    ownerkey: ownerKey
                }
            }))
        }
    })
</script>
</html>
```
Now, when you got your wallet ID, you can call any [Wallet API](api/README.md#api) method, like check status, create an address or do a transaction.

## Open wallet
Open wallet with given ID and password.
```js
{
	jsonrpc:'2.0',
	id:'<request ID>',
	method:'open_wallet',
	params:
	{
		id:'<wallet ID>',
		pass:'<wallet password>'
	}
}
```
