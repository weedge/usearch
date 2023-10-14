# Compiling USearch

This document describes the workflow for advanced USearch users and enthusiasts willing to contribute to the project.
Before building the first time, please pull submodules.

```sh
git submodule update --init --recursive
```

## C++11

Linux:

```sh
sudo apt-get update
sudo apt-get install cmake build-essential libjemalloc-dev
cmake -B ./build_release \
    -DCMAKE_BUILD_TYPE=Release \
    -DUSEARCH_USE_OPENMP=1 \
    -DUSEARCH_USE_SIMSIMD=1 \
    -DUSEARCH_USE_JEMALLOC=1 \
    -DUSEARCH_BUILD_TEST_CPP=1 \
    -DUSEARCH_BUILD_BENCH_CPP=1 \
    -DUSEARCH_BUILD_LIB_C=1 \
    -DUSEARCH_BUILD_TEST_C=1 \
    && \
    make -C ./build_release -j
```

MacOS:

```sh
brew install libomp llvm
cmake -B ./build_release \
    -DCMAKE_C_COMPILER="/opt/homebrew/opt/llvm/bin/clang" \
    -DCMAKE_CXX_COMPILER="/opt/homebrew/opt/llvm/bin/clang++" \
    -DUSEARCH_USE_OPENMP=1 \
    -DUSEARCH_USE_SIMSIMD=1 \
    -DUSEARCH_BUILD_BENCH_CPP=1 \
    -DUSEARCH_BUILD_TEST_CPP=1 \
    && \
    make -C ./build_release -j
```

Linting:

```sh
cppcheck --enable=all --force --suppress=cstyleCast --suppress=unusedFunction \
    include/usearch/index.hpp \
    include/index_dense.hpp \
    include/index_plugins.hpp
```

Testing:

```sh
cmake -DUSEARCH_BUILD_TEST_CPP=1 -B ./build_debug
cmake --build ./build_debug --config Debug
./build_debug/test_cpp
```

## Python 3

Use PyTest to validate the build.
The `-s` option will disable capturing the logs.
The `-x` option will exit after first failure to simplify debugging.

```sh
pip install -e . && pytest python/scripts/ -s -x
```

Linting:

```sh
pip install ruff
ruff --format=github --select=E9,F63,F7,F82 --target-version=py37 python
```

Testing wheel builds locally:

```sh
pip install cibuildwheel
cibuildwheel --platform linux
```

## JavaScript

Node.JS:

```sh
npm install && node --test ./javascript/usearch.test.js
npm publish
```

WebAssembly:

```sh
emcmake cmake -B ./build -DCMAKE_CXX_FLAGS="${CMAKE_CXX_FLAGS} -s TOTAL_MEMORY=64MB" && emmake make -C ./build
node ./build/usearch.test.js
```

If you don't yet have `emcmake` installed:

```sh
git clone https://github.com/emscripten-core/emsdk.git && ./emsdk/emsdk install latest && ./emsdk/emsdk activate latest && source ./emsdk/emsdk_env.sh
```

## Rust

```sh
cargo test -p usearch
cargo publish
```

## Java

```sh
gradle clean build
java -cp . -Djava.library.path="$(pwd)/build/libs/usearch/shared" java/cloud/unum/usearch/Index.java
```

Or step by-step:

```sh
cs java/unum/cloud/usearch
javac -h . Index.java

# Ubuntu:
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/linux cloud_unum_usearch_Index.cpp -o cloud_unum_usearch_Index.o
g++ -shared -fPIC -o libusearch_c.so cloud_unum_usearch_Index.o -lc

# Windows
g++ -c -I%JAVA_HOME%\include -I%JAVA_HOME%\include\win32 cloud_unum_usearch_Index.cpp -o cloud_unum_usearch_Index.o
g++ -shared -o USearchJNI.dll cloud_unum_usearch_Index.o -Wl,--add-stdcall-alias

# MacOS
g++ -std=c++11 -c -fPIC \
    -I../../../../include \
    -I../../../../fp16/include \
    -I../../../../simsimd/include \
    -I${JAVA_HOME}/include -I${JAVA_HOME}/include/darwin cloud_unum_usearch_Index.cpp -o cloud_unum_usearch_Index.o
g++ -dynamiclib -o libusearch.dylib cloud_unum_usearch_Index.o -lc
# Run linking to that directory
java -cp . -Djava.library.path="$(pwd)/java/cloud/unum/usearch/" Index.java
java -cp . -Djava.library.path="$(pwd)/java" cloud.unum.usearch.Index
```

## Objective-C and Swift

```sh
swift build
swift test -v
```

## C 99

There are a few ways to compile the C 99 USearch SDK.
Using the Makefile, specifying the targets you need:

```sh
make -C ./c libusearch_c.so
```

With options:

```sh
make USEARCH_USE_OPENMP=1 USEARCH_USE_SIMSIMD=1 -C ./c libusearch_c.so
```

Using CMake:

```sh
cmake -B ./build_release -DUSEARCH_BUILD_LIB_C=1 -DUSEARCH_BUILD_TEST_C=1 -DUSEARCH_USE_OPENMP=1 -DUSEARCH_USE_SIMSIMD=1 
cmake --build ./build_release --config Release -j
./build_release/test_c
# On Windows:
.\build_release\test_c.exe
```


## GoLang

GoLang bindings are based on C.
So one should first compile the C library, link it with GoLang, and only then run tests.

```sh
make -C ./c libusearch_c.so && mv ./c/libusearch_c.so ./golang/ && cp ./c/usearch.h ./golang/
cd golang && go test -v ; cd ..
```

## Wolfram

```sh
brew install --cask wolfram-engine
```


## Docker

```sh
docker build -t unum/usearch . && docker run unum/usearch
```

For multi-architecture builds and publications:

```sh
version=$(cat VERSION)
docker buildx create --use &&
    docker login &&
    docker buildx build \
        --platform "linux/amd64,linux/arm64" \
        --build-arg version=$version \
        --file Dockerfile \
        --tag unum/usearch:$version \
        --tag unum/usearch:latest \
        --push .
```

## Sub-Modules

Extending metrics in SimSIMD:

```sh
git push --set-upstream https://github.com/ashvardanian/simsimd.git HEAD:main
```
