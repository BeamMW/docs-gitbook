
Beam Shader Examples can be found in the following directory: https://github.com/BeamMW/beam/tree/master/bvm/Shaders

Download or checkout this folder and follow the steps below

1. Install the latest version of clang for your platform from [here](https://releases.llvm.org/download.html#11.0.0). Use pre-built binaries section

2. From the parent folder run make_shader.bat (for Windows) or make_shader.sh (for Linux or Mac)

Example:  make_shader.bat vault\contract

This operation will create .wasm files in the appropriate folders.

You need to build both contract and app shaders to deploy a new contract, or just an app shader to work with existing contract

