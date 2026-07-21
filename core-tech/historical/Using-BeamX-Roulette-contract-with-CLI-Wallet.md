> **Historical Document** — This guide covers the BeamX Roulette contract on the experimental BeamX testnet. It is preserved for historical reference only.

Let's play some games!

In this example we will interact with the contract that 

As usual we assume that you have read [BeamX Getting Started Guide](BeamX-Getting-Started.md) and [Using BeamX Faucet contract with CLI Wallet](Using-BeamX-Faucet-contract-with-CLI-Wallet.md)

We also assume that you have the app.wasm and contract.was in the shaders/roulette folder under the 'beamx' folder where you keep your wallet binary file.

If so, we are all set to spin our Roulette.

Well, it's a kind of roulette. In our case your bet is on any number between 0 and 36 (including both 0 and 36). If you guess the number exactly you receive 100 tokens (which are actually new confidential assets on Beam chain). This actually means that in order to create a new Roulette contract you would need to lock the new asset type deposit (which in this chain is configured to be 1000 Beam)

1. How do I play?

Since Roulette contract has an implicit owner (who is the dealer) there could be many roulettes out there. To find all of them we will run the following command:

`beam-wallet-beamx.exe shader --shader_app_file shaders\roulette\app.wasm --shader_args="role=player,action=bet,cid=ac16be1cb673ace9423d32fe20893e1639fd560e35e6b3433c77bee831647372,iSector=15" -n 127.0.0.1:8501`

To check the status of your bet run

`beam-wallet-beamx.exe shader --shader_app_file shaders\roulette\app.wasm --shader_args="role=player,action=check,cid=ac16be1cb673ace9423d32fe20893e1639fd560e35e6b3433c77bee831647372" -n 127.0.0.1:8501`

To get your winning run:

`beam-wallet-beamx.exe shader --shader_app_file shaders\roulette\app.wasm --shader_args="role=player,action=take,cid=ac16be1cb673ace9423d32fe20893e1639fd560e35e6b3433c77bee831647372" -n 127.0.0.1:8501`



