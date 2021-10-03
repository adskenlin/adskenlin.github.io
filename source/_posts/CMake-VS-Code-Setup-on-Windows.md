---
title: CMake VS Code Setup on Windows
date: 2021-08-13 02:26:38
tags: 
- CMake
- VS Code
---
# CMake VS Code Setup on Windows
In this article, I would like to introduce how to use CMake with VS Code on Windows.
## What's CMake
----
CMake(Cross Platform Make) is free and open-source software for build automation, testing, packaging and installation of software by using a compiler-independent method. It's widely used in software projects. CMake offers higher-level mechanisms like external library detection and configuration (i.e. automatically setting up the defines, include directories and link files for working with a 3rd party library), support for generating installation (and packaging), integration with the CTest test utility and so on.


## Installation
----
On Windows, CMake could be directly downloaded und installed with their [official website](https://cmake.org/download/).
After Installation, in order to use `cmake` in cmd, you have to add the path to your environments' variables following:


    1. Click Start, type Accounts in the Start search box, and then click User Accounts under Programs.

    2. If you are prompted for an administrator password or for a confirmation, type the password, or click Allow.

    3. In the User Accounts dialog box, click Change my environment variables under Tasks.

    4. Make the changes that you want to the user environment variables for your user account, and then click OK.
To verify installation, you could type `cmake --version` in your cmd/powershell.

In VS Code, make sure the `Extensions` named `CMake` and `CMake Tools` successfully installed.

## Setup
----
At the beginning, create a folder and use VS Code to open it. We'll use this folder to test the functions of CMake.  

Additionally, make sure you've correct configuration for compiler path in `C/C++: Edit Configurations(UI)`(use ctrl+shift+P and type in). In my case, the path is `C:/msys64/mingw64/bin/gcc.exe`.

It's highly recommanded that a C++ project with CMake use the folder architecture as follows:
```bash
|- test_folder
    |--.vscode
    |    |-- c_cpp_properties.json
    |    |-- tasks.json
    |    └-- launch.json
    |-- build
    |-- include
    |    └-- your-head-files
    |-- src
    |    └-- source_code.cpp
    └-- CMakeLists.txt
```
if you don't know how to get `c_cpp_properties.json`, `tasks.json`, `launch.json`, please refer to [here](https://code.visualstudio.com/docs/cpp/config-msvc) and [Debug using launch.json](https://code.visualstudio.com/docs/cpp/launch-json-reference) for help.

The core part is how to write the `CMakeLists.txt`:
Actually `cmake` is using `CMakeLists.txt` to execute build. A simple example:
```cmake
cmake_minimum_required (VERSION 2.8...3.21.1)

project(project_name)

include_directories(${CMAKE_SOURCE_DIR}/include)

aux_source_directory(${CMAKE_SOURCE_DIR}/src DIR_SRC)

add_executable (project_name ${DIR_SRC})
```
The variables and commands for `CMakeLists.txt` will be introduced in next part.

Now we have all requirements for cmake, let's try:
```bash
cd build
cmake ..
make
```
If you don't have `make` installed, here is a short cut to make it work:
1. Go to the folder where you install you compiler MinGW, in my case it's under `C:\msys64\mingw64\bin`.
2. Find a file named `mingw32-make.exe` and make a copy in the same foler.
3. Rename the copy to `make.exe`.
4. Check with `make --version` in cmd or powershell.

If the codes above work, you'll see *.exe created under `/build`, cmake works successfully.

## Useful Variables and Commands for CMakeLists.txt
----
The meaning of following words are able to be found under this [documentaion](https://cmake.org/cmake/help/latest/index.html)
+ Variables: 
```cmake
CMAKE_SOURCE_DIR

PROJECT_SOURCE_DIR

CMAKE_CURRENT_SOURCE_DIR

CMAKE_MODULE_PATH

EXECUTABLE_OUTPUT_PATH

LIBRARY_OUTPUT_PATH

PROJECT_NAME
```
+ Commands:
```cmake
// setup project name and supported progamming language
project(projectname [CXX] [C] [Java])

// setup the supported cmake version 
CMAKE_MINIMUM_REQUIRED(VERSION versionNumber [FATAL_ERROR])

// setup variables
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

// find all source code files and make them into a list
aux_source_directory(dir VARIABLE)

// add subdirectory to the build
add_subdirectory(source_dir [binary_dir][EXCLUDE_FROM_ALL])

// find head files
include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)

// Adds an executable target called <name> to be built from the source files listed
add_executable(<name> [source1] [source2 ...])

// ...
```
## Reference:
[1] [CMake Wiki](https://en.wikipedia.org/wiki/CMake) \
[2] [What are the benefits/purposes of cmake?](https://stackoverflow.com/questions/19686223/what-are-the-benefits-purposes-of-cmake) \
[3] [CMake Documentation](https://cmake.org/cmake/help/latest/index.html#)



