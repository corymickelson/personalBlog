---
title: Native Nodejs Development Environment Setup
date: 2018-01-27 14:58:21
tags:
---
### A beginners guide to setting up a native addon development environment

By the end of this tutorial you should be able to start a new native nodejs project and build, run, and debug with 
Qt Creator.

## Pre-reqs
The following should be available through your native package manager.
 - If you do not already have Qt creator installed, please go install [it](https://www.qt.io/download-qt-for-application-development) now.
 - Install the latest stable version of nodejs (at the time of writing that is 8.9.3)
 - A C/C++ compiler, gnu or clang
 - CMake
 - GDB


## Project setup

In this step we are going to create a new nodejs project, install required dependencies, and create a minimal CMakeLists.txt file.
This project is using node-addon-api, you can swap this for nan if that is your preference.

 - Make your project directory, cd into it and `npm init`
 - Install cmake-js, and node-addon-api `npm i cmake-js node-addon-api`
 - Create CMakeLists.txt file in root of project

In your CMakeLists.txt file add the following:
``` cmake
cmake_minimum_required(VERSION 3.2)
project(intro)
file(GLOB SOURCE_FILES 
    "src/*.h"
    "src/*.cc")
add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES 
    PREFIX ""
    SUFFIX ".node")
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_SOURCE_DIR}/node_modules/node-addon-api 
    ${CMAKE_SOURCE_DIR}/node_modules/node-addon-api/src
    ${CMAKE_JS_INC})
target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_JS_LIB})
```

For more information on cmake-js, read the [docs](https://github.com/cmake-js/cmake-js)

The above CMakeLists.txt file is defining a project
``` cmake
project(intro)
```
linking required header files and source files
``` cmake
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_SOURCE_DIR}/node_modules/node-addon-api 
    ${CMAKE_SOURCE_DIR}/node_modules/node-addon-api/src
    ${CMAKE_JS_INC})
target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_JS_LIB})
``` 
setting properties of the output, and defining the build as a shared library.
``` cmake
add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES}) #shared library
set_target_properties(${PROJECT_NAME} PROPERTIES 
    PREFIX ""
    SUFFIX ".node") # will output an intro.node file
```

In the CMakeLists.txt file we declared our source files as `"src/*.h"` and `"src/*.cc"`, if you haven't already go ahead and create a src directory, and add files src/addon.h and src/addon.cc
It seems to also be convention to have all our javascript library code in a `lib` directory, you can add this now as well as a main.js file in the lib folder.

Our project should appear as such:
```
project
    lib
        main.js
    src
        addon.h
        addon.cc
    package.json
    CMakeLists.txt
```

## C++ Development Environment
Qt will be our primary ide for writing and debugging native code. 

Import our project
 - press `ctrl-o` (open file or project)
 - find and select the CMakeLists.txt file we just created

This will open the project configuration pane. Select the compiler you would like to use, or just select the qt default and press the
`configure project` button.

![Image of qt import project configuration](../../../../../images/import_configure.png)

We should now be in the qt editor. This is the place we will do all of our C/C++ editing. This tutorial will not go into writing native addons. 

Add the following to addon.cc

``` cpp
#include <napi.h>

Napi::String Method(const Napi::CallbackInfo& info) {
  Napi::Env env = info.Env();
  return Napi::String::New(env, "world");
}

Napi::Object Init(Napi::Env env, Napi::Object exports) {
  exports.Set(Napi::String::New(env, "hello"),
              Napi::Function::New(env, Method));
  return exports;
}

NODE_API_MODULE(hello, Init)
```

and the following to addon.js
``` js
var addon = require('bindings')('hello');

console.log(addon.hello()); // 'world'
```

## Building
Building our addon is simple with the help of cmake-js. To configure a default build in Qt, select 'Project mode' identified by the
wrench icon.

![Image of Qt project build settings](../../../../../images/build_settings.png)

Under build settings select `Add build step` -> `Custom Process Step`. 

![Image of Qt custom build step](../../../../../images/custom_build_step.png)

Set the command to cmake-js executable, which will be under either your project node modules or global node modules.
Assuming we are using project node modules your path will look something like this: `node_modules/cmake-js/bin/cmake-js`.
Set arguments to `build -D`. This is just saying that we want to create a debug build. Lastly set the working directory to `%{sourceDir}` 
to set the working directory to our project root 

## Running
We've configured our build configuration now all that is left is our run configuration.
On the same `Project mode` view we were in for our build configuration, select the run tab on the left side bar directly under the build tab.

![Image of Qt run configuration](../../../../../images/run_configuration.png)

Set the executable to the path to your nodejs executable. Here I am using nvs and my executable is locatec under .nvs directory. 
Next set the command line arguments to your entry point javascript file.
Lastly set the working directory to `%{sourceDir}`.

## Debugging
We're finally here, stepping through our native node code. To start debugging, set some breakpoints and hit F5. 
This will initiate our custom build step, and one completion of our build will call our run configuration on the javascript file
specified in the project mode run settings.

![Image Qt Debug](../../../../../images/debug_view.png)

And that is it. I hope you found this helpful, this is a excerpt from my project [NoPoDoFo](https://github.com/corymickelson/NoPoDoFo).