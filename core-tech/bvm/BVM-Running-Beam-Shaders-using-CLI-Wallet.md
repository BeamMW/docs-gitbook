
You can run shader commands using Beam CLI wallet by following the steps below

## Prerequisites

1. Make sure you have Beam node running and connected to Beam network (preferred) [documentation](https://documentation.beam.mw/en/latest/rtd_pages/user_beam_node_guide.html)
2  Make sure you have Beam CLI wallet connected to your Beam node or to one of the Beam bootstrap nodes [documentation](https://documentation.beam.mw/en/latest/rtd_pages/user_cli_wallet_guide.html)

As you remember there are two types of Shaders:
1. **Contract Shaders** - implement Smart Contract functionality, are store on the blockchain and are running on Beam nodes.
2. **App Shaders**      - implement Smart Contract API and are running in Beam Wallet.

All examples refer to a sample application 'mydapp' which has two Shaders, App Shader (app.wasm) and Contract Shader (contract.wasm)

## Deploy new Contract Shader to blockchain

Just like all other operations, deploying a Contract also 

Example: 
```
beam-wallet-testnet.exe shader --shader_app_file=mydapp\app.wasm --shader_args="role=manager,action=create" --shader_contract_file=mydapp\contract.wasm -n 127.0.0.1:8501
```

## List APP Shader API

You can see all commands supported by the App Shader by running it without any parameters 
```
beam-wallet-testnet.exe shader --shader_app_file=mydapp\app.wasm -n 127.0.0.1:8501
```