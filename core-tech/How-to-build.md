Starting from version 5.0 we separated core beam library project and its UI.

Desktop UI is a separte git repostotry now [https://github.com/BeamMW/beam-ui](https://github.com/BeamMW/beam-ui).

For earlier versions, build instructions are located [here](https://github.com/BeamMW/beam/wiki/How-to-build(old)).

# Before build

1. Ensure that you have `git` installed
1. `beam-ui` and `beam` project use `CMake` to generate build environment. The minimal supported version of `CMake` is **3.13** 
1. Download and install `boost` library. The minimal supported version is 1.66
1. Download and install `OpenSSL` library

# Common part

1. Clone Beam sources before building  
    ```
    git clone https://github.com/BeamMW/beam.git
    ```  

1. Beam has submodules 
    ```
    cd beam
    git submodule update --init --recursive
    ```

## CMake options
Beam uses CMake to generate a build. Beam has several options to say how to build the project, which feature to turn ON/OFF.
Please, look reference to root `CMakeLists.txt` file.

# Windows
1. Install Visual Studio >= 2017 with CMake support.
1. Download and install Boost prebuilt binaries https://sourceforge.net/projects/boost/files/boost-binaries/1.75.0/boost_1_75_0-msvc-14.1-64.exe, also add `BOOST_ROOT` to the _Environment Variables_.
1. Download and install OpenSSL prebuilt binaries https://slproweb.com/products/Win32OpenSSL.html (`Win64 OpenSSL v1.1.0h` for example) and add `OPENSSL_ROOT_DIR` to the _Environment Variables_.
1. Add `.../boost_1_75_0/lib64-msvc-14.1` to the _System Path_.
1. Open project folder in Visual Studio, select your target (`Release-x64` for example, if you downloaded 64bit Boost and OpenSSL) and select `CMake -> Build All`.
1. Open cache folder
   ```
   Visual Studio 2017:
   Go to CMake -> Cache -> Open Cache Folder -> beam

   Visual Studio 2019:
   Project -> CMake Cache -> Open in Explorer

   You'll find binaries in beam/..., wallet/cli/..., explorer/... subfolders.
   ```

# Linux 

**Please read carefully the points below**

**1.** After finishing your build you'll find the the `beam-node` binary (node) in the `beam/beam` folder, `beam-wallet` (cli wallet) in the `beam/wallet/cli` folder.

**2.** You need to clone beam sources before building
```
git clone https://github.com/BeamMW/beam.git
```
The command above checks out the latest master branch where all the development happens. Usually it is not what you want to build. Choose an appropriate branch [here](https://github.com/BeamMW/beam/branches) and check out it after you have the main repository cloned. For example the command below checks out the Double Doppler 4.0 release branch
```
cd beam
git checkout double_doppler_4.0RC
```

**3.** By default instructions produce masternet builds (development version, development blockchain). To be able to build testnet (testing) or mainnet (real-world blockchain) you need to edit the `beam/CMakeLists.txt` file lines 151-157. Uncomment (remove the '#' sign) respective definitions.
```
# uncomment next line for testnet
#set(BEAM_TESTNET TRUE)
# uncomment next line for mainnet
#set(BEAM_MAINNET TRUE)

# uncomment next line for swap mainnet
#set(SWAP_MAINNET TRUE)
```

## Ubuntu 18.04 Desktop

Instructions below are valid for a clean default Ubuntu 18.04.03 Desktop LTS install. If you're building on an aged system please check that steps 1, 2 & 4 do not overwrite/conflict with more recent versions of cmake and boost that might have been installed before. You can omit cmake/boost installation if you already have more recent versions.

1. Install dependencies
    ```
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost-all-dev
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the Beam project folder and start the release build
    ```
    export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j 4
    ```

## Ubuntu 18.04 Server

Instructions below are valid for a clean default Ubuntu 18.04.03 Server LTS with all the updates applied during install. If your're building on an aged system please check that steps 1 & 2 do not overwrite/conflict with more recent versions of cmake and boost that might have been installed before. You can omit cmake/boost installation if you already have more recent versions.

1. Install dependencies
    ```
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost1.67-dev
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the Beam project folder and start the release build
    ```
    export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j 4
    ```

## Ubuntu 19.10 Server

Instructions below are valid for a default Ubuntu 19.10 Server install with updates applied during install.

1. Install dependencies
    ```
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

## Ubuntu 19.10 Desktop

Instructions below are valid for a minimal Ubuntu 19.10 Desktop install with 'update during install' option enabled.

1. Install dependencies
    ```
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev 
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

## Centos 7.7

Instructions below are valid for a clean CentOS 7.0-1908 minimal install. If your're building on an aged system please check that steps 2 & 3 do not overwrite more recent versions of cmake and boost that might have been installed before.

1. Install dependencies 
    ```
    sudo yum update && shutdown -r now
    sudo yum install centos-release-scl yum-utils
    sudo yum-config-manager --enable rhel-server-rhscl-8-rpms
    sudo yum install devtoolset-8-gcc* 
    scl enable devtoolset-8 bash
    sudo yum install git make wget openssl-devel
    ```

1. Build & install boost. This step could take considerable amount of time
    ```
    wget https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
    tar xzvf boost_1_66_0.tar.gz
    cd boost_1_66_0 && ./bootstrap.sh && sudo ./b2 install && cd ~
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the beam sources folder and start the release build 
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

## CentOS 8

Instructions below are valid for a clean CentOS 8.0-1905 minimal install. If your're building on an aged install please check that step 2 does not overwrite more recent version of cmake that might have been installed before.

1. Install dependencies
    ```
    sudo dnf update
    sudo dnf config-manager --set-enabled PowerTools
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install openssl-devel tar wget 
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the beam sources folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

## Fedora 31 Workstation

Instructions below are valid for a default Fedora 31-1.9 Workstation install.

1. Install dependencies
    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar 
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

## Fedora 31 Server

Instructions below are valid for a default Fedora 31-1.9 Headless Server install.

1. Install dependencies
    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```

# Mac
1. Download the source code by using: 
    ```
    git clone https://github.com/BeamMW/beam.git
    ```

1. Install Brew Package Manager.

1. Install necessary packages using:  
    ```
    brew install openssl cmake
    ```
1. Remove any existing brew installations of boost:  
    ```
    brew uninstall --ignore-dependencies boost
    ```
1. Download and install a compatible version of boost:  
    ```
    brew install boost@1.76
    brew link --force --overwrite boost@1.76
    ```
1. Set your Environment Variables by using the following:  
    ```
    - export OPENSSL_ROOT_DIR="/usr/local/opt/openssl@3"
    ```
1. Go to Beam project folder and call  
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j 4
    ```
1. Use `cpack` to get .dmg file. Go to Beam project folder and call 
    ```
    cpack
    ```
1. You'll find binaries in `beam/wallet/cli`, and `beam/explorer` subfolders.

