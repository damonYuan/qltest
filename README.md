README
====

# Prerequisite

Prepare the bin folders

```
mkdir -p ~/cbin/bin ~/cbin/include ~/cbin/lib
```

Install through Homebrew
```
brew install boost
brew install quantlib
```
or build quantlib by yourself
```
git submodule update --init --recursive
cd libs/quantlib
cmake -S . -B build
cmake --build build
cmake --install build --prefix ~/cbin
```
and add
```
include_directories(/Users/damon/cbin/include) # quantlib
link_directories(/Users/damon/cbin/lib) # quantlib
```
in your root CMakeLists.txt,
or build quantlib through CMake as the current version of CMakeLists.txt with
```
add_subdirectory("${MAIN_LIBRARIES_DIR}/quantlib")
target_link_libraries(${MAIN_APP_PROJECT_NAME} ql_library)
```

# How to build, test and install

```
cmake -S . -B build
cmake --build build --target test
cmake --build build
cmake --install build --prefix ~/cbin
qltest
```

# How to uninstall

```
cat build/install_manifest.txt | xargs rm
```