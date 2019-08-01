# Classic Robotic Enterprise Framework implemented as custom activity
Repository containing UIPath REFramework library, documentation and examples.

The library can be installed from Go (https://go.uipath.com/component/reframework-as-library) or by downloading the .nupkg file from https://github.com/emanueldogaruUiPath/REFramework/tree/master/bin and adding it to an UiPath Studio local directory.

The library should work with UiPath 2018.4 and later

## Overview
The goal of REFramework is to facilitate the development of business automations by providing a ready-to-use, well-tested, off-the-shelf architecture. Without having to worry about the robot architecture, the developer can focus on the business process to implement, while REFramework takes care of important topics such as: 
- exception handling 
- retry mechanism 
- configuration mechanism 
- logging capabilities

Implemented as custom activity, the REFramework brings a new level of simplicity, a clearer separation of Business and Architecture layers and a centralized distribution model, while remaining fully compatible with classic REFramework.

## Benefits

- Fully compatibility with Classic Reframework
- Reusability
- Centralized code (as library)
- Real separation between Architecture and Process Layer
- Hides complexity from entry-level users, while providing the same functionality
- Easier to deal with workflow arguments type
- Samples generation
- Edit config arguments directly from UiPath
- Config file from json file
- Config file from asset
- Enforce argument naming convention (potential even more enforcing of best practices)
- Separation of InitTransactionData and InitAllApplications
- New hooks (EndProcess, HandleTransactionStatus, HandleEachException)
- Timeouts parameter for each workflow
- More granularity regarding queue items