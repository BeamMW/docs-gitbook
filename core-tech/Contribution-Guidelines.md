# Beam Contribution Guidelines

## Introduction

Beam is an open source project, and as such welcomes developers to contribute.
<br>
In order to simplify and organize this process we have written this short contribution guide that explains the key principles of contributing to Beam.

For any questions you might have regarding the process please contact Beam CTO at alex at beam.mw or @bigromanov on Telegram or @BeamCTO on Twitter.

For more specific questions please contact the developer team on Telegram: https://t.me/beamdevsupport

## Code Style

Before contributing C++ code, please read the [Beam C++ Style and Conventions](Beam-Cpp-Style-And-Conventions) guide. It covers naming conventions, memory ownership, error handling, serialization, the IO/reactor model, logging, and CMake module structure used across `core/`, `node/`, `wallet/`, `bvm/`, and `utility/`.

## Projects overview

We are planning on initially supporting contributions to the following projects: 

1. Beam Desktop UI Wallet (C++ / QT) - described in this document
2. Beam Web Wallet Chrome Extension (JS / Angular) instructions coming soon

### Beam Desktop UI Wallet

This is a UI for the Beam Desktop Wallet. It uses the underlying libwallet library and wraps it with user interface implemented with QT framework. 

To contribute to this project please follow the steps below:

1. Setup your dev environment, checkout and build the project using the [Building Instructions](https://github.com/BeamMW/beam-ui/wiki/How-to-build-Beam-desktop-UI)

2. Explore tasks in the currently active project on the [projects page](https://github.com/BeamMW/beam-ui/issues).

3. Look for tasks marked with the ['help wanted' label](https://github.com/BeamMW/beam/labels/help%20wanted). Of course you are also welcome to contribute to other issues as well

4. Write a comment clarifying that you are starting to work on the task and providing a very rough estimate how much time you expect to spend on it.

5. If the task does not have a clearly assigned bounty in the task description please contact @bigromanov on Telegram, or email alex at beam.mw