# Building Beam

This document will serve as a guide for building Beam and BeamUI projects. Users have the option to download the binaries directly from [GitHub](https://github.com/BeamMW/beam), however, for this walkthrough, users must have _some_ technical knowledge, including basic knowledge of the Command-Line.

## Things you'll need

To avoid any issues, have the following installed on your machine before starting your build:

1. The latest version of [_Git_](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)_._
2. [_CMake_](https://cmake.org/download/) (version 3.13 and newer to generate the build environment).
3. [_Boost library_](https://www.boost.org/users/history/version\_1\_66\_0.html) (version 1.66 and newer_)._
4. [_OpenSSL_ _Library_](https://www.openssl.org/source/).
5. [_Microsoft Visual Studio_](https://visualstudio.microsoft.com/vs/older-downloads/) (2017 and newer; Windows users only).

## First steps

### Downloading the binaries

Launch a terminal window and enter the following command:

```
git clone https://github.com/BeamMW/beam.git
```

### Creating submodules

As a prerequisite, Beam requires the creation of submodules before starting the build. To create submodules, launch your terminal window and enter the following commands:

```
cd beam
git submodule update --init --recursive
```

### Branch Convention

Beam is entirely open-source, and we encourage users interested in contributing or testing our tech to follow the Branch convention below to avoid any issues.

1. `Master` branch is for development.
2. `Testnet` branch is a sandbox for users interested in testing Beam.
3. `Mainnet` branch is the most recent version of each Beam project.

### _CMake_ Options

Beam uses _CMake_ to generate the build environment with several options for building the project. Refer to `CMakeLists.txt` file for more details.

## Windows


This guide assumes you have the latest version [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed on your machine.


1. Install _Microsoft Visual Studio_ with [_CMake_ support](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=msvc-160#building-cmake-projects).
2. Download and install the Boost [prebuilt](https://sourceforge.net/projects/boost/files/boost-binaries/1.68.0/boost\_1\_68\_0-msvc-14.1-64.exe) binaries, adding `BOOST_ROOT` to the _Environment Variables_.
3. Download and install the _OpenSSL_ [prebuilt](https://slproweb.com/products/Win32OpenSSL.html) binaries (`Win64 OpenSSL v1.1.0h` for example). Add `OPENSSL_ROOT_DIR` to the _Environment Variables_.
4. Add `.../boost_1_68_0/lib64-msvc-14.1` to the _System Path_.
5. Open project folder in _Visual Studio_, select your target (`Release-x64` for example, if you downloaded 64bit Boost and _OpenSSL_) and select `CMake -> Build All`.
6.  Open cache folder:

    ![](<../.gitbook/assets/Screen Shot 2021-11-28 at 3.27.04 PM.png>)

    
    For _Visual Studio 2017_ users, go to _**CMake**_** > Cache > Open Cache Folder > `Beam`**

    For _Visual Studio 2019_ users**, Project > **_**CMake**_** cache > Open in explorer** (the binaries can be found in **`beam/..., wallet/cli/..., explorer/... subfolders`**).
    

## Linux


Locate the downloaded Beam node and Command Line (CLI) wallet in the `beam/wallet/cli` folder.


1.  Clone the Beam binaries from [Github](https://github.com/BeamMW) by entering the following command:

    ```
    git clone https://github.com/BeamMW/beam.git
    ```

    
    **Please note**: The above command will clone the latest master branch. The master branch is where the Beam team develops upcoming versions of Beam. Users should select the appropriate [branch](https://github.com/BeamMW/beam/branches) then `git checkout` after cloning the main Beam repository.
    

    To clone the **`Double Doppler 4.0`** release branch, enter the following:

    ```
    cd beam
    git checkout double_doppler_4.0RC
    ```
2.  The standard tutorials for Building beam will create the latest Beam Masternet build (development version, development blockchain). Build the Testnet for testing and development or build the Mainnet for real-world blockchain utilization by editing the `beam/CMakeLists.txt` file lines 151-157. Uncomment (remove the '#' sign) respective definitions for the version Beam you wish to build.

    ```
    # uncomment next line for testnet
    #set(BEAM_TESTNET TRUE)
    # uncomment next line for mainnet
    #set(BEAM_MAINNET TRUE)

    # uncomment next line for swap mainnet
    #set(SWAP_MAINNET TRUE)
    ```

## Ubuntu and desktop server

### Ubuntu  18.04

#### Desktop


The instructions below are valid for a default **Ubuntu 18.04.03 Desktop LTS install**. For older systems, verify that steps 1, 2 & 4 do not overwrite or conflict with more recent versions of `CMake` and `boost` previously installed on your machine.


1.  Install dependencies:

    ```
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost1.67-dev
    ```
2.  Install `CMake`:

    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```
3.  Find your Beam project folder, start the release build:

    ```
    export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j4
    ```

#### Server

1.  Install dependencies:

    ```
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost1.67-dev
    ```
2.  Install `CMake`:

    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```
3.  Find your Beam project folder, start the release build:

    ```
    export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j4
    ```

### Ubuntu 19.10

#### Desktop


Instructions below are valid for a minimal **Ubuntu 19.10 Desktop install** ("update during install" option enabled).


1.  Install dependencies:

    ```
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev 
    ```
2.  Find your Beam project folder, start the release build:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

#### Server

1.  Install dependencies

    ```
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev
    ```
2.  Find your Beam project folder, start the release build:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

## CentOS

### CentOS 7.0


The instructions below are valid for a default **CentOS 7.0-1908 minimal install**. For older systems, verify that steps two and three do not overwrite or conflict with more recent versions of `CMake`and `boost` previously installed on your machine.


1.  Install dependencies:

    ```
    sudo yum update && shutdown -r now
    sudo yum install centos-release-scl yum-utils
    sudo yum-config-manager --enable rhel-server-rhscl-8-rpms
    sudo yum install devtoolset-8-gcc* 
    scl enable devtoolset-8 bash
    sudo yum install git make wget openssl-devel
    ```
2.  Build & install boost. This step could take considerable amount of time

    ```
    wget https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
    tar xzvf boost_1_66_0.tar.gz
    cd boost_1_66_0 && ./bootstrap.sh && sudo ./b2 install && cd ~
    ```
3.  Install `CMake`:

    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```
4.  Find your Beam project folder, start the release build:



    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

### CentOS 8.0


The instructions below are valid for a default **CentOS 8.0-1905 minimal install**. For older systems, verify that step two does not overwrite or conflict with more recent versions of `CMake`and `boost` previously installed on your machine.


1.  Install dependencies:

    ```
    sudo dnf update
    sudo dnf config-manager --set-enabled PowerTools
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install openssl-devel tar wget 
    ```
2.  Install `CMake`:

    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```
3.  Find your Beam project folder, start the release build:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

## Fedora

#### Desktop


The instructions below are valid for a default **Fedora 31-1.9 Workstation install**.


1.  Install dependencies:

    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar 
    ```
2.  Find your Beam project folder, start the release build:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

#### Server


The instructions below are valid for a default **Fedora 31-1.9 Headless Server install**.


1.  Install dependencies:

    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar
    ```
2.  Find your Beam project folder, start the release build:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

## MacOS

1.  Clone the Beam binaries from [Github](https://github.com/BeamMW):

    ```
    git clone https://github.com/BeamMW/beam.git
    ```

    
    **Please note:** The Git command above will clone the latest master branch. The master branch is where the Beam team develops upcoming versions of the Beam.
    

    To clone the `mainnet`, enter the following command:

    ```
    git clone --branch mainnet https://github.com/BeamMW/beam.git
    ```

    To clone the Beam `testnet`, use:

    ```
    git clone --branch testnet https://github.com/BeamMW/beam.git
    ```
2. [Install](https://docs.brew.sh/Installation) `Brew` Package Manager.
3.  To install all necessary packages, enter:

    ```
    brew install openssl cmake
    ```
4.  Remove any existing `Brew` installations of `boost`:

    ```
    brew uninstall --ignore-dependencies boost
    ```
5.  [Download](https://docs.brew.sh/Installation) and install a compatible version of `boost`:

    ```
    curl -O https://raw.githubusercontent.com/Homebrew/homebrew-core/5da5895add2f6b9320d654dd844d4827f6876c8b/Formula/boost.rb
    brew install ./boost.rb
    ```
6.  To set your Environment Variables, enter:

    ```
    - export OPENSSL_ROOT_DIR="/usr/local/opt/openssl@1.1"
    ```
7.  Find your Beam project folder, then enter:

    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```
8.  Use `cpack` to get `.dmg file.` Find Beam project folder and then enter:

    ```
    cpack
    ```
9. Locate the completed Beam node in the `beam/wallet/cli` and `beam/explorer` subfolders
