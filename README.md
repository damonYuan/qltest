README
====

# Prerequisite

```
brew install boost
brew install quantlib
```

# How to build, test and install

```
git submodule update --init --recursive
cmake -S . -B build
cmake --build build
ctest --test-dir build
cmake --install build --prefix ~/cbin
qltest
```

# How to uninstall

```
cat build/install_manifest.txt | xargs rm
```