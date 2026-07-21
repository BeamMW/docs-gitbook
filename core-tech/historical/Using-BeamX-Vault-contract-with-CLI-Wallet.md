> **Historical Document** — This guide covers the BeamX Vault contract on the experimental BeamX testnet. It is preserved for historical reference only.

NOTE: If you haven't already, please review [BeamX Getting Started Guide](BeamX-Getting-Started.md) and [Using BeamX Faucet contract with CLI Wallet](Using-BeamX-Faucet-contract-with-CLI-Wallet.md)

In this section we will learn how to use BeamX Vault example. It is very similar to the Faucet example, only there are no limitation on deposits and withdrawals. It's basically just a simple vault contract you can keep money in.

As usual we assume that you have the app.wasm and contract.was in the shaders/vault folder under the 'beamx' folder where you keep your wallet binary file.

If so, you can use the following list of commands:

## Create contract:

`beam-wallet-beamx.exe shader --shader_manager_file shaders/vault/app.wasm --shader_args="role=manager,action=create" --shader_contract_file shaders/vault/contract.wasm -n 127.0.0.1:8501`

## List all contracts of this type

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm --shader_args="role=manager,action=view" -n 127.0.0.1:8501`



## List all shader commands

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm -n 127.0.0.1:8501`

## View my account

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm --shader_args="role=my_account,action=view,cid=7965a18aefaf3050ccd404482eb919f6641daaf111c7c4a7787c2e932942aa91" -n 127.0.0.1:8501`


## Deposit 1 beam

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm --shader_args="role=my_account,action=deposit,cid=7965a18aefaf3050ccd404482eb919f6641daaf111c7c4a7787c2e932942aa91,amount=100000000" -n 127.0.0.1:8501`

## Withdraw

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm --shader_args="role=my_account,action=withdraw,cid=7965a18aefaf3050ccd404482eb919f6641daaf111c7c4a7787c2e932942aa91,amount=100000000" -n 127.0.0.1:8501`

## View all accounts

`beam-wallet-beamx.exe shader --shader_app_file shaders/vault/app.wasm --shader_args="role=all_accounts,action=view,cid=7965a18aefaf3050ccd404482eb919f6641daaf111c7c4a7787c2e932942aa91" -n 127.0.0.1:8501`