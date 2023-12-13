# Shader SDK (WIN)

This tutorial demonstrates How-to set up Beam Shader SDK on the Windows operating system.

{% hint style="warning" %}
The Shader SDK's current version can be characterized as an _**early developer preview**_, with a high possibility of encountering unusual issues and bugs.
{% endhint %}

### Prerequisites

To follow along, make sure you have [Microsoft Visual Studio](https://visualstudio.microsoft.com/) installed. For this tutorial, I am using 2022 edition, but the community edition will work.

Additionally, [Python 3.x](https://www.python.org/downloads/) must be installed and available in the systems path. Verify Python is installed by entering  `python` in the command line.

### Building Shader SDK

1. [**Clone Shader SDK repository**](https://github.com/BeamMW/shader-sdk)**.** 
2. Open Developer Command Prompt for VS. This can be done from either Tools Menu within the VS IDE or by finding an entry in the Start menu items
3. Change directory to the location of the shader-sdk and run: `shade init`

This will operation will install and build all the necessary components.

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Creating a new Shader Project

1. Launch the Command Prompt, creating a new folder for your Shader project. We will use the `vault` folder for this tutorial:

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

2\. Locate `shader-sdk` and `wasi-sdk`folders, and enter the following:

`set BEAM_SHADER_SDK=c:\Code\shader-sdk`

`set WASI_SDK_PREFIX=C:\Code\shader-sdk\wasi-sdk-14.0`

3. To initialize the structure of your Shader, enter  `shade create_project <project name>`:

For our `<project name>`, use `vault`, i.e.,:

`c:\Code\shader-sdk\shade create_project vault`

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

If initialized correctly, your Shader project structure should resemble the following:

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>Vault project structure</p></figcaption></figure>



### Building the Shader Project

In Visual Studio, locate **Open A Local Folder** from the **Start** menu:

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Navigate to your Shader project folder, click **Select Folder** to initialize a **CMake project**, creating `CMakeLists.txt` and `CMakeLists.json` files will be seen in the **Solution Explorer** for your Shader project:

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption><p>Vault Solution Explorer</p></figcaption></figure>

Select `wasm32-Release` build configuration:

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

This will build `.wasm` files following this folder structure:

`C:\Code\vault\build\wasi\shaders`

{% hint style="warning" %}
If you are encountering errors, try the following before reaching out to support:

1. Open Windows Explorer and delete the `out` folder.
2. In the  **Project** menu, select **Delete Cache and Reconfigure.**
3. Rebuild `wasm32-Release` project.
{% endhint %}

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>



You can begin coding your Shader project!
