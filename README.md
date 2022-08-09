README
====

Although QuantLib python version is a good candidate for testing purpose, there are still some differences compared with the c++ version (actually python version is just a wrap using [SWIG](https://www.swig.org/) around the c++ version), not to say that some of the latest functions could not be found in the python version. [Quantlib Installation Manual](https://www.quantlib.org/install.shtml) has provided some instruction about how to build and install the library in your machine through outdated `autoconf` tools while there are not much information about how to build through the modern `cmake` and how to integrate it as dynamic library into your c++ project.

This project aims at setting up a c++ cmake-based project for testing the features in QuantLib, and it could also be a template to be extended upon.

# Prerequisite

## Prepare the bin folders

You could install the binary into system default bin folder, however here a customized directory is preferred to make the development version more manageable.

```
mkdir -p ~/cbin/bin ~/cbin/include ~/cbin/lib
```
and expose `~/cbin/bin` in the PATH, eg., add it to the `~/.bash_profile`.

## Install the `boost`

Here in Mac I will install it through `Homebrew`,

```
brew install boost
```

## Install the `QuantLib`

There are more at least three ways to integrate the `QuantLib`.

### Install Dependencies through Homebrew
Install through Homebrew
```
brew install quantlib
```
and use the following CMakeList.txt
```
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(MAIN_APP_PROJECT_NAME
        qltest
        )
set(MAIN_APP_SOURCE_DIR
        src
        )
set(MAIN_APP_SOURCE
        ${MAIN_APP_SOURCE_DIR}/app.cpp
        )
set(MAIN_LIBRARIES_DIR
        ${CMAKE_SOURCE_DIR}/libs
        )

# build
project(${MAIN_APP_PROJECT_NAME})
find_library(QUANTLIB_LIBRARY quantlib HINTS /usr/local/opt)
add_executable(${MAIN_APP_PROJECT_NAME} ${MAIN_APP_SOURCE})
target_link_libraries(${MAIN_APP_PROJECT_NAME}
        ${QUANTLIB_LIBRARY}
        )
target_include_directories(${MAIN_APP_PROJECT_NAME} PUBLIC
        /usr/local/include
        /opt/homebrew/include/
        )

# install
install(TARGETS ${MAIN_APP_PROJECT_NAME} DESTINATION bin)

# gtest
include(FetchContent)
FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)
FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(qltest_test test/app_test.cpp)
target_link_libraries(qltest_test gtest_main)

include(GoogleTest)
gtest_discover_tests(qltest_test)
```

### Two-Steps Installation

You will build the QuantLib by yourself and install it into the customized path
```
git submodule update --init --recursive
cd libs/quantlib
cmake -S . -B build
cmake --build build --clean-first
cmake --install build --prefix ~/cbin
```
and then use a CMakeLists.txt with the following content
```
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(MAIN_APP_PROJECT_NAME
        qltest
        )
set(MAIN_APP_SOURCE_DIR
        src
        )
set(MAIN_APP_SOURCE
        ${MAIN_APP_SOURCE_DIR}/app.cpp
        )
set(MAIN_LIBRARIES_DIR
        ${CMAKE_SOURCE_DIR}/libs
        )
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH ON) # https://stackoverflow.com/questions/47697761/cmake-run-time-error-dyld-library-not-loaded-for-dynamically-linked-resource
# build
project(${MAIN_APP_PROJECT_NAME})
find_library(QUANTLIB_LIBRARY quantlib
        HINTS "~/cbin/lib")
find_path(
        QUANTLIB_INCLUDE
        NAMES "ql/quantlib.hpp"
        PATHS "~/cbin/include"
)
add_executable(${MAIN_APP_PROJECT_NAME} ${MAIN_APP_SOURCE})
target_link_libraries(${MAIN_APP_PROJECT_NAME}
        ${QUANTLIB_LIBRARY}
        )
target_include_directories(${MAIN_APP_PROJECT_NAME} PUBLIC
        /usr/local/include
        /opt/homebrew/include/
        ${QUANTLIB_INCLUDE}
        )

# install
install(TARGETS ${MAIN_APP_PROJECT_NAME} DESTINATION bin)

# gtest
include(FetchContent)
FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)
FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(qltest_test test/app_test.cpp)
target_link_libraries(qltest_test gtest_main)

include(GoogleTest)
gtest_discover_tests(qltest_test)
```
in your root CMakeLists.txt and build the project through

```
cmake -S . -B build
cmake --build build --clean-first
cmake --build build --target test
cmake --install build --prefix ~/cbin
qltest
```

### One-Step Installation

This is the method that the project is using now.

Download the source code
```
git submodule update --init --recursive
```
and then build `QuantLib` through CMake using the following CMakeLists.txt
```
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(MAIN_APP_PROJECT_NAME
        qltest
        )
set(MAIN_APP_SOURCE_DIR
        src
        )
set(MAIN_APP_SOURCE
        ${MAIN_APP_SOURCE_DIR}/app.cpp
        )
set(MAIN_LIBRARIES_DIR
        ${CMAKE_SOURCE_DIR}/libs
        )
list(APPEND CMAKE_INSTALL_RPATH /Users/damon/cbin/lib) # https://stackoverflow.com/questions/43551483/how-to-set-rpath-in-cmake
# build
project(${MAIN_APP_PROJECT_NAME})
add_executable(${MAIN_APP_PROJECT_NAME} ${MAIN_APP_SOURCE})
add_subdirectory("${MAIN_LIBRARIES_DIR}/quantlib")
target_link_libraries(${MAIN_APP_PROJECT_NAME}
        ql_library
        )
target_include_directories(${MAIN_APP_PROJECT_NAME} PUBLIC
        /usr/local/include
        /opt/homebrew/include/
        /Users/damon/cbin/include
        )

# install
install(TARGETS ${MAIN_APP_PROJECT_NAME} DESTINATION bin)

# gtest
include(FetchContent)
FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)
FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(qltest_test test/app_test.cpp)
target_link_libraries(qltest_test gtest_main)

include(GoogleTest)
gtest_discover_tests(qltest_test)
```
and then build the project
```
cmake -S . -B build
cmake --build build
cmake --build build --target test
cmake --install build --prefix ~/cbin
qltest
```

HERE YOU GO!

# How to uninstall

```
cat build/install_manifest.txt | xargs rm
```

# Others

Use `otool` to check the linked libraries

```
otool -L ./build/qltest
```
