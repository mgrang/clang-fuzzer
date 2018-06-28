# Building clang-proto-fuzzer

### Checkout and build libprotobuf

TOP=$PWD

```
Login to an Ubuntu 14 machine
sudo apt-get remove libprotobuf-dev
```

```
mkdir protobuf && cd protobuf
git clone https://github.com/google/protobuf.git src
git submodule update --init --recursive
git checkout 3.1.x
cd src
make clean && make distclean
./autogen.sh && ./configure --prefix=$TOP/protobuf/install && make -j32 && make install
```

## Checkout and build libprotobuf-mutator
```
git clone https://github.com/google/libprotobuf-mutator.git

Apply patch:
patch -p1 < libprotobuf-mutator.patch
```

## Checkout and build llvm
```
Use the above downloaded llvm as the host build compiler
wget http://releases.llvm.org/6.0.0/clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-14.04.tar.xz

Checkout community llvm (you may remove polly as it is not needed).

cd clang and apply patch (Remember to update the path to the protobuf PB_PATH in the patch):
patch -p1 < clang.patch
```

## Configure and build the fuzzer
```
export PATH=<path_to_the above_downloaded_llvm>:$PATH
mkdir $TOP/build/llvm && cd $TOP/build/llvm

cmake -GNinja -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="RISCV" -DCMAKE_VERBOSE_MAKEFILE=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DLLVM_USE_SANITIZE_COVERAGE=YES -DLLVM_USE_SANITIZER=Address -DCLANG_ENABLE_PROTO_FUZZER=ON -DCMAKE_PREFIX_PATH=$TOP/install ../../src/llvm

ninja -v clang-proto-fuzzer clang-proto-to-cxx 2>&1 | tee log
```
