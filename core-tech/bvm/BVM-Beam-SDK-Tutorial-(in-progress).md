# Prerequisites

* git
* cmake >= 3.13
* python >= 3.4
* ninja

# Preparation

1. clone shader-sdk repository:
```console
git clone https://github.com/BeamMW/shader-sdk.git
```

2. init shader-sdk with `init` command:
```console
cd shader-sdk
./shade init 
```
> On Windows 
>```console
>cd shader-sdk
>shade init
>```

3. go to the directory you want to generate a new project 
```console
cd <project-root>
```

3. create new contract with `create_project` command:
```console
python shader-sdk/scripts/manager.py create_project HelloWorld --newdir
```
> you can omit `--newdir` flag, in this case the project will be generated right into your `ProjectRoot` directory

4. jump into your new contract's directory:
```console
cd HelloWorld
```

# Development

1. open contract.h with your favourite text editor or IDE:
```console
nvim shaders/contract.h
```
> On Windows you can simply call context menu on your project folder and choose `Open with Visual Studio`. Project folder has generated `CMakeSettings.json` file and it allows to compile contract and app shaders right from Visual Studio using `wasm32-Release` configuration.

2. add structure with params for contract's Method_2:
```
diff --git a/shaders/contract.h b/shaders/contract.h
index 89c926e..100c36e 100644
--- a/shaders/contract.h
+++ b/shaders/contract.h
@@ -5,11 +5,20 @@
 namespace HelloWorld {
 #include "contract_sid.i"
 
+	constexpr size_t MSG_MAXSIZE = 256;
+
 #pragma pack(push, 1)
 
	struct InitialParams {
		static const uint32_t METHOD = 0;
	};
 
+	struct SendMsgParams {
+		static const uint32_t METHOD = 2;
+
+		char msg[MSG_MAXSIZE];
+		PubKey user;
+	};
+
 #pragma pack(pop)
 }
```

3. save and close `contract.h`:

4. open `contract.cpp` with your favorite text editor:
```console
nvim shaders/contract.cpp
```

5. write Method_2:
```
diff --git a/shaders/contract.cpp b/shaders/contract.cpp
index ae11ee1..e80c870 100644
--- a/shaders/contract.cpp
+++ b/shaders/contract.cpp
@@ -13,7 +13,7 @@ BEAM_EXPORT void Dtor(void*)
	Env::DelVar_T(0);
 }
 
-BEAM_EXPORT void Method_2(void*)
+BEAM_EXPORT void Method_2(const HelloWorld::SendMsgParams& params)
 {
-
+	Env::SaveVar_T(params.user, params.msg);
 }
```

6. save and close `contract.cpp`

7. open `app.cpp` with your favorite text editor
```console
nvim shaders/app.cpp
```

8. add 2 actions: send_msg, get_my_msg
```
diff --git a/shaders/app.cpp b/shaders/app.cpp
index 3414d9f..501e769 100644
--- a/shaders/app.cpp
+++ b/shaders/app.cpp
@@ -59,6 +59,30 @@ void On_action_view_contract_params(const ContractID& cid)
	Env::DocGroup gr("params");
 }
 
+void On_action_send_msg(const ContractID& cid)
+{
+	HelloWorld::SendMsgParams params;
+
+	Env::DocGetText("msg", params.msg, sizeof(params.msg));
+	Env::DerivePk(params.user, &cid, sizeof(cid));
+
+	Env::GenerateKernel(&cid, HelloWorld::SendMsgParams::METHOD, &params, sizeof(params), nullptr, 0, nullptr, 0, "Sending msg to contract", 0);
+}
+
+void On_action_get_my_msg(const ContractID& cid)
+{
+	Env::Key_T<PubKey> key;
+	key.m_Prefix.m_Cid = cid;
+
+	Env::DerivePk(key.m_KeyInContract, &cid, sizeof(cid));
+
+	char msg[HelloWorld::MSG_MAXSIZE];
+	Env::VarReader::Read_T(key, msg);
+
+	Env::DocGroup root("");
+	Env::DocAddText("Your message", msg);
+}
+
 BEAM_EXPORT void Method_0()
 {
	 Env::DocGroup root("");
@@ -84,7 +108,14 @@ BEAM_EXPORT void Method_0()
		 {
			 Env::DocGroup grRole("user");
			 {
+				Env::DocGroup grMethod("send_msg");
+				Env::DocAddText("cid", "ContractID");
+				Env::DocAddText("msg", "string");
			 }
+			{
+				Env::DocGroup grMethod("get_my_msg");
+				Env::DocAddText("cid", "ContractID");
+			}
		 }
	 }
 }
@@ -92,6 +123,8 @@ BEAM_EXPORT void Method_0()
 BEAM_EXPORT void Method_1()
 {
	const Actions_map_t VALID_USER_ACTIONS = {
+		{"send_msg", On_action_send_msg},
+		{"get_my_msg", On_action_get_my_msg},
	};
 
	const Actions_map_t VALID_MANAGER_ACTIONS = {
```

9. save and exit

# Building

1. build your shader:
```console
make
```

2. fix compilation errors if any

# Deploying & Testing (in progress)

1. jump into your beam directory:
```console
cd beam
```

2. export your owner key:
```console
wallet/cli/beam-wallet-masternet -n=eu-node01.masternet.beam.mw:8100,eu-node02.masternet.beam.mw:8100,eu-node04.masternet.beam.mw:8100,eu-node04.masternet.beam.mw:8100 export_owner_key
```

3. start your local node:
```console
beam/beam-node-masternet --peer=eu-node01.masternet.beam.mw:8100,eu-node02.masternet.beam.mw:8100,eu-node04.masternet.beam.mw:8100,eu-node04.masternet.beam.mw:8100 --port=8501 --fast_sync=on --owner_key={your_owner_key}
```

4. deploy your contract:
```console
wallet/cli/beam-wallet-masternet -n localhost:8501 shader --shader_app_file={path_to_your_shader_directory}/shaders/app.wasm --shader_contract_file={path_to_your_shader_directory}/shaders/contract.wasm --shader_args="role=manager,action=create_contract"
```

5. get your contract's CID:
```console
wallet/cli/beam-wallet-masternet -n localhost:8501 shader --shader_app_file={path_to_your_shader_directory}/shaders/app.wasm --shader_args="role=manager,action=view_contracts"
```
Output:
```
Shader output: "contracts": [{"cid": "fde65e7875de624e3e460d426c0288c3b587ac42840b21f65dfef73f52936b3d","Height": 23}]
```

6. test your methods:
```console
wallet/cli/beam-wallet-masternet -n localhost:8501 shader --shader_app_file={path_to_your_shader_directory}/shaders/app.wasm --shader_args="role=user,action=send_msg,msg=HelloWorld\!,cid={your_cid}"
```
```console
wallet/cli/beam-wallet-masternet -n localhost:8501 shader --shader_app_file={path_to_your_shader_directory}/shaders/app.wasm --shader_args="role=user,action=get_my_msg,cid={your_cid}"
```

Congratulations! You just wrote your first contract :)