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