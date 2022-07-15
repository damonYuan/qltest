README
====

Although QuantLib python version is a good candidate for testing purpose, there are still some differences compared with the c++ version, not to say that some of the latest functions could not be found. This project aims at setting up a c++ cmake-based project for testing the features in QuantLib, and it could also be a project template to be extended on for all kinds of reasons.

# Prerequisite

## Prepare the bin folders

You could install the binary into system default bin folder, however here a customized directory is preferred to make the development version more manageable.

```
mkdir -p ~/cbin/bin ~/cbin/include ~/cbin/lib
```
and add `~/cbin` into the PATH.

## Install Dependencies through Homebrew
Install through Homebrew
```
brew install boost
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

## Install through Build, Install and Import
`boost` is still required.
```
brew install boost
```
and then you will build the QuantLib by yourself and install it into the customized path
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
in your root CMakeLists.txt

## Install through CMAKE lib

`boost` is still required.
```
brew install boost
```
and then download the source code
```
git submodule update --init --recursive
```
and then build quantlib through CMake as the current version of CMakeLists.txt in this project. Then you could install it through

```
cmake -S . -B build
cmake --build build --clean-first
cmake --build build --target test
cmake --install build --prefix ~/cbin
qltest
```

Here you go!

# How to uninstall

```
cat build/install_manifest.txt | xargs rm
```

# Others

Use `otool` to check the linked libraries

```
otool -L ./build/qltest
```