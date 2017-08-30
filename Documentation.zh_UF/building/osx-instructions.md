在OS X上构建CoreCLR
=====================

这份指南会一步一步带你在OS X上构建CoreCLR。让我们从设置编译环境开始。

环境
===========

下面的步骤在OS X Yosemite上验证通过，但在之前的版本上应该也行。欢迎提交针对其他环境的Pull Request。

如果安装的是Command Line Tools for XCode 6.3，那么需要升级到6.3.1或者更高的版本才能完成构建。6.3版的头文件有个问题，在6.3.1里修复了。

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

CoreCLR的构建过程依赖于CMake。你可以从 [CMake downloads](http://www.cmake.org/download/) 下载。

或者，你也可以通过 [Homebrew](http://brew.sh/) 安装CMake：

```sh
dotnet-mbp:~ richlander$ brew install cmake
```

ICU
---
ICU（International Components for Unicode）也是构建所必须的。你可以通过 [Homebrew](http://brew.sh/) 安装它。

```sh
brew install icu4c
brew link --force icu4c
```

OpenSSL
-------
CoreFX加密库基于OpenSSL构建。OS X自带的OpenSSL版本（0.9.8）已经不再被官方支持，因此我们需要安装一个新版本。你可以通过 [Homebrew](http://brew.sh) 安装。

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

在coreclr代码库的根目录运行`build.sh`构建CoreCLR：

```sh
dotnet-mbp:~ richlander$ cd ~/git/coreclr
dotnet-mbp:coreclr richlander$ ./build.sh
```

构建完成后，生成的文件放在`bin/Product/OSX.x64.Debug`中。我们感兴趣的文件有：

- `corerun`：命令行宿主程序。这个程序加载并启动CoreCLR运行时，然后运行你交给它的托管程序。
- `libcoreclr.dylib`：CoreCLR运行时。
- `mscorlib.dll`：Microsoft Core Library。

构建Framework
===================

```sh
dotnet-mbp:corefx richlander$ ./build.sh
```

构建完成后，你可以在`bin`目录看到结果。
