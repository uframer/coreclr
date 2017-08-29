在OS X上构建CoreCLR
=====================

This guide will walk you through building CoreCLR on OS X. We'll start by showing how to set up your environment from scratch.

环境
===========

These instructions were validated on OS X Yosemite, although they probably work on earlier versions. Pull Requests are welcome to address other environments.

If your machine has Command Line Tools for XCode 6.3 installed, you'll need to update them to the 6.3.1 version or higher in order to successfully build. There was an issue with the headers that shipped with version 6.3 that was subsequently fixed in 6.3.1.

Git设置
---------

克隆CoreCLR和CoreFX代码库：

```sh
dotnet-mbp:git richlander$ git clone https://github.com/dotnet/coreclr
# Cloning into 'coreclr'...

dotnet-mbp:git richlander$ git clone https://github.com/dotnet/corefx
# Cloning into 'corefx'...
```

本指南假设你将coreclr和corefx分别克隆到`~/git/coreclr`和`~/git/corefx`中。如果你使用了不同的位置，那么请在执行后续的命令时多加注意。在本指南中，我会在执行每个命令时标出当前所在的目录。

CMake
-----

CoreCLR has a dependency on CMake for the build. You can download it from [CMake downloads](http://www.cmake.org/download/).

或者，你也可以通过[Homebrew](http://brew.sh/)安装CMake：

```sh
dotnet-mbp:~ richlander$ brew install cmake
```

ICU
---
ICU（International Components for Unicode）也是构建所必须的。你可以通过[Homebrew](http://brew.sh/)安装它。

```sh
brew install icu4c
brew link --force icu4c
```

OpenSSL
-------
CoreFX加密库基于OpenSSL构建。OS X自带的OpenSSL版本（0.9.8）已经不再被官方支持，因此我们需要安装一个新版本。你可以通过[Homebrew](http://brew.sh)安装。

```sh
brew install openssl

# We need to make the runtime libraries discoverable, as well as make
# pkg-config be able to find the headers and current ABI version.
#
# Ensure the paths we will need exist
mkdir -p /usr/local/lib/pkgconfig

# The rest of these instructions assume a default Homebrew path of
# /usr/local/opt/<module>, with /usr/local being the answer to
# `brew --prefix`.
#
# Runtime dependencies
ln -s /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib /usr/local/lib/
ln -s /usr/local/opt/openssl/lib/libssl.1.0.0.dylib /usr/local/lib/

# Compile-time dependences (for pkg-config)
ln -s /usr/local/opt/openssl/lib/pkgconfig/libcrypto.pc /usr/local/lib/pkgconfig/
ln -s /usr/local/opt/openssl/lib/pkgconfig/libssl.pc /usr/local/lib/pkgconfig/
ln -s /usr/local/opt/openssl/lib/pkgconfig/openssl.pc /usr/local/lib/pkgconfig/
```

构建运行时和Microsoft Core Library
============================================

To Build CoreCLR, run build.sh from the root of the coreclr repo.

```sh
dotnet-mbp:~ richlander$ cd ~/git/coreclr
dotnet-mbp:coreclr richlander$ ./build.sh
```

After the build is completed, there should some files placed in `bin/Product/OSX.x64.Debug`. The ones we are interested in are:

- `corerun`: The command line host. This program loads and starts the CoreCLR runtime and passes the managed program you want to run to it.
- `libcoreclr.dylib`: The CoreCLR runtime itself.
- `mscorlib.dll`: Microsoft Core Library.

构建Framework
===================

```sh
dotnet-mbp:corefx richlander$ ./build.sh
```

构建完成后，你可以在`bin`目录看到结果。
