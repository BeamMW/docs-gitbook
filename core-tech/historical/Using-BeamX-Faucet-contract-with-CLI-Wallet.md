> **Historical Document** — This guide covers the BeamX Faucet contract on the experimental BeamX testnet. It is preserved for historical reference only.

NOTE: If you have not installed BeamX please follow the instructions in the [Getting Started Guide](BeamX-Getting-Started.md)

Make sure you have downloaded the latest version of the '[faucet](https://github.com/BeamMW/beam/tree/beamX/bvm/Shaders/faucet)' contract folder and put it under the 'shaders' folder as instructed. You need app.wasm to use the faucet and contract.wasm to deploy your own contract.

In this section we will use BeamX Faucet contract to get some coins on the BeamX chain.

1. Open the Command Prompt (on Windows), Shell (on Linux) or Terminal (on MacOS) and change directory to the location of your BeamX CLI wallet.

2. Run the following command (assmuing app.wasm is located in shaders/faucet/app.wasm)

`beam-wallet-beamx.exe shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=view"`

NOTE: We assume that your wallet is initialized and your node is running on local machine at port 8501 and that faucet app.wasm is located in the shaders/faucet folder. If your parameters are different please adjust them accordingly. Follow the [Getting Started Guide](BeamX-Getting-Started.md) for more detailed instructions.

The result of this command will be the list of *contract ids* of shaders of type 'faucet' that are currently deployed on the network. The reason that there can be more than one such shader is because the same Application Shader (app.wasm) can interact with several different Contract Shaders (contract.wasm) of the same type. 

3. Now let's try to withdraw 3 beam coins from a specific faucet by running the following command

`beam-wallet-beamx.exe shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=my_account,action=withdraw,cid=c327a42e9037d060b8158d635990a53ea4cde2e217ed70eb5faf93cac22e4771,amount=300000000" -n 127.0.0.1:8501`

You can replace the cid parameter with a different contract id that you have received as a result of step 2.

This operation can result in several different outcomes:

1. Each faucet has a limit on how many Beam coins you can withdraw each time within a give period. Both of these are 'constructor parameters' of the shader that are specified when it is  If you are under that limit for that period you should receive the Beams that you have requested.
2. If you are over the limit you will either receive nothing or a Magical Error 17, depending on the contract implementation.
3. You will also receive the Magical Error 17 if something else is wrong with the contract.

4. But how do I know what are the contract parameters?

For that we will run the following command:

`beam-wallet-beamx.exe shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=manager,action=view_params,cid=c327a42e9037d060b8158d635990a53ea4cde2e217ed70eb5faf93cac22e4771" -n 127.0.0.1:8501`

As a result you will get something like this:

Shader output: "": {"params": {"backlogPeriod": 10,"withdrawLimit": 500000000}}

This means that from this faucet, you can extract up to 5 Beam every 10 blocks

5. But what if I want to create my own faucet? 

No problem, however there are few things that you need to know:

1. Since faucet is not an 'ownable' contract (it will be explained later, but basically it means that it does not have an implicit owner) you can not create another faucet contract with the same parameters (see [BeamX Smart Contracts Guide](../bvm/BVM-Beam-Smart-Contracts.md) for more details on that). Therefore you should just invent a couple of parameters of your own, which should not be difficult enough in this case.
2. Since you are deploying a new contract on BeamX blockchain, you need to make sure you have the appropriate contract.wasm file for it. We will assume that you have it in the same folder as the app.wasm

Now that these two things are out of the way, let's run the following command:

`beam-wallet-beamx.exe shader --shader_app_file shaders/faucet/app.wasm -n 127.0.0.1:8501 --shader_args="role=manager,action=create,backlogPeriod=13, withdrawLimit=520000000" --shader_contract_file shaders/faucet/contract.wasm`

Assuming no one before you deployed a faucet that allowed to get up to 5.2 Beams every 13 blocks, you deployment will be successful and you will get a new contract id in returned value.

No all that is left to do is to deposit some funds into your faucet and let people of the world know it exists (just give them the id)

`beam-wallet-beamx.exe shader --shader_app_file shaders\faucet\app.wasm --shader_args="role=my_account,action=deposit,cid=c327a42e9037d060b8158d635990a53ea4cde2e217ed70eb5faf93cac22e4771,amount=10000000000" -n 127.0.0.1:8501`

This command will deposit 100 Beams from your wallet, assuming you got them

6. Nice, how can I see all of the commands the Shader supports?

Good question. Just run the following command:

`beam-wallet-beamx.exe shader --shader_app_file shaders\faucet\app.wasm -n 127.0.0.1:8501`

Basically, if you don't provide any shader parameters you will get a JSON describing all parameters and their values.