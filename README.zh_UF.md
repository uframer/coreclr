.NET Core通用语言运行时（CoreCLR）
===========================

这个代码库包含 [.NET Core](http://dotnet.github.io) 运行时的完整代码。如果你刚接触 .NET Core ，那么请从 [About .NET](https://docs.microsoft.com/en-us/dotnet/articles/about/) 入手，它很快会把你引向 [.NET Core Tutorials](https://docs.microsoft.com/en-us/dotnet/articles/core/getting-started) 。


你可以将 .NET Core 看做是 *敏捷版 .NET* 。简单来说它类似于Windows系统自带的 [Desktop .NET Framework](https://en.wikipedia.org/wiki/.NET_Framework) 的子集，但是它跨平台（Windows/Linux/macOS）并且跨体系（x86/x64/arm），而且可以作为应用程序的一部分部署，因此也能够在修正bug或者添加功能时更方便的更新。  

## 如果你只是想使用 .NET Core

绝大多数用户并不需要从源代码自己构建 .NET Core ，在我们支持的平台上都已经提供了经过了测试的预编译版本。你可以从 [.NET Core Getting Started](http://dotnet.github.io/getting-started/) 页面下载到最新的 **released** 版 .NET Core SDK 。如果你需要最新（每日构建）版本的 .NET Core 安装程序，可以去 [latest Installers of .NET Core and .NET Core SDK](https://github.com/dotnet/cli#installers-and-binaries) 页面下载。

## 想了解代码之外的内容？

除了提供源代码，这个代码库还扮演着 .NET Core 相关内容纽带的角色：

 * 想要**进一步学习** .NET 运行时的内部机制？请阅读 [Documentation on the .NET Core Runtime](Documentation/README.md) 。
 * 想要**报告一个问题**或者提供反馈？请阅读 [Issues and Feedback Page](Documentation/workflow/IssuesFeedbackEngagement.md) 页面。
 * Want to **chat** with other members of the CoreCLR community?  See the [Chat Section](Documentation/workflow/IssuesFeedbackEngagement.md#Chat-with-the-CoreCLR-community) page.
 * Need a **current build** or **test results** of the CoreCLR repository?   See the [Official and Daily Builds](Documentation/workflow/OfficalAndDailyBuilds.md) page.
 * If you want powerful search of the source code for both CoreClr and CoreFx see [.NET Source Code Index](https://source.dot.net)

## 这个代码库能够构建出什么？

.NET Core 十分依赖 [Nuget](https://en.wikipedia.org/wiki/NuGet) 包管理器，后者负责打包、分发并管理软件组件的版本。参见 [https://www.nuget.org/](https://www.nuget.org/) 。现在你只需要知道，Nuget是一个将软件组件打包成 `*.nupkg` 包（其实是一个ZIP档案）的系统，这些包可以被 *发布* 到一个本地文件路径或者是一个URL上（例如， https://www.nuget.org/ ）。有一些工具（例如，Nuget.exe、Visual Studio、dotnet.exe）可以从配置文件（.csproj）中获知如何从这些发布地址查找并下载应用程序所需的软件包。   

具体来讲，最好认为这个代码库保存的是如下Nuget包的源代码：
 
 * **Microsoft.NETCore.Runtime.CoreCLR** - 包括对象分配器、垃圾收集器、类加载器、类型系统、interop和 .NET类库 中最最基础的内容（例如，System.Object, System.String ...）

它还包括下面紧密相关的支持包的源代码：

 * **Microsoft.NETCore.Jit** - [.NET Intermediate language (IL)](https://en.wikipedia.org/wiki/Common_Intermediate_Language) 的JIT编译器
 * **Microsoft.NETCore.ILAsm** - [.NET Intermediate language (IL)](https://en.wikipedia.org/wiki/Common_Intermediate_Language) 的汇编器
 * **Microsoft.NETCore.ILDAsm** - [.NET Intermediate language (IL)](https://en.wikipedia.org/wiki/Common_Intermediate_Language) 的反汇编器
 * **Microsoft.NETCore.TestHost** - This contains the corehost.exe program, which is a small wrapper 
   that uses the .NET Runtime to run IL DLLs passed to it on the command line.
 * **Microsoft.TargetingPack.Private.CoreCLR** - A set of assemblies that represent the compile time surface 
   area of the class library implemented by the runtime itself.

## 同[CoreFX](https://github.com/dotnet/corefx)代码库的关系

By itself, the `Microsoft.NETCore.Runtime.CoreCLR` package is actually not enough to do much.
One reason for this is that the CoreCLR package tries to minimize the amount of the class library that it implements.
Only types that have a strong dependency on the internal workings of the runtime are included (e.g, 
`System.Object`, `System.String`, `System.Thread`, `System.Threading.Tasks.Task` and most foundational interfaces).
Instead most of the class library is implemented as independent Nuget packages that simply use the .NET Core 
runtime as a dependency.    Many of the most familiar classes (`System.Collections`, `System.IO`, `System.Xml` and 
so on), live in packages defined in the [dotnet/corefx](https://github.com/dotnet/corefx) repository.

But the main reason you can't do much with CoreCLR is that **ALL** of the types in the class library **LOOK** 
like they are defined by the CoreFX framework and not CoreCLR.   Any library code defined here 
lives in a single DLL called `System.Private.CoreLib.dll` and as its name suggests is private (hidden).
Instead for any particular PUBLIC type defined in CoreCLR, we found the 'right' package in CoreFX where it naturally 
belongs and use that package as its **public publishing** point.   That 'facade' package then forwards references 
to the (private) implementation in `System.Private.CoreLib.dll` defined here.
For example the *`System.Runtime`* package defined in CoreFX declares the PUBLIC name for types like 
`System.Object` and `System.String`.   Thus from an applications point of view these types live in `System.Runtime.dll`. 
However, `System.Runtime.dll` (defined in the CoreFX repo) forwards references ultimately to `System.Private.CoreLib.dll` 
which is defined here.

Thus in order to run an application, you need BOTH the `Microsoft.NETCore.Runtime.CoreCLR` Nuget package 
(defined in this repository) as well as  packages for whatever you actually reference that were defined 
in the CoreFX repository (which at a minimum includes the `System.Runtime` package).    You also need some 
sort of 'host' executable that loads the CoreCLR package as well as the CoreFX packages and starts your code (typically 
you use `dotnet.exe` for this).   

These extra pieces are not defined here, however you don't need to build them in order to use the CoreCLR 
Nuget package you create here.   There are already versions of the CoreFX packages published on 
https://www.nuget.org/ so you can have your test application's project.json specify the CoreCLR you 
built and it will naturally pull anything else it needs from the official location https://www.nuget.org/ to 
make a complete application.  More on this in the [Using Your Build](Documentation/workflow/UsingYourBuild.md) page.

--------------------------
## 用GIT克隆CoreCLR代码库

The first step in making a build of the CoreCLR Repository is to clone it locally.   If you already know
how to do this, just skip this section.  Otherwise if you are developing on windows you can see
[Setting Up A Git Repository In Visual Studio 2015](https://github.com/Microsoft/perfview/blob/master/documentation/SettingUpRepoInVS2015.md)
for for instructions on setting up.  This link uses a different repository as an example, but the issues (do you fork or not) and
the procedure are equally applicable to this repository.  

--------------------------
## 构建代码库

The build depends on Git, CMake, Python and of course a C++ compiler.  Once these prerequisites are installed
the build is simply a matter of invoking the 'build' script (`build.cmd` or `build.sh`) at the base of the 
repository.  

不同系统上安装组件的方式不同，请参考下面针对每种系统的页面。目前不支持各个操作系统之间的交叉编译（只支持从X64到ARM的交叉编译）。如果目标平台是某个系统，就必须在这种系统上构建。

 * [Windows构建指南](Documentation/building/windows-instructions.md)
 * [Linux构建指南](Documentation/building/linux-instructions.md)
 * [macOS构建指南](Documentation/building/osx-instructions.md)
 * [FreeBSD构建指南](Documentation/building/freebsd-instructions.md) 
 * [NetBSD构建指南](Documentation/building/netbsd-instructions.md)

构建分为两种类型（*buildTypes*）：

 * Debug（默认） - 这样编译出的运行时带有额外的运行时检查（断言）。这些检查These checks slow 
   runtime execution but are really valuable for debugging, and is recommended for normal development and testing.  
 * Release - This compiles without any development time runtime checks.  This is what end users will use but 
   can be difficult to debug.   Passing 'release' to the build script select this.  

In addition, by default the build will not only create the runtime executables, but it will also 
build all the tests.   There are quite a few tests so this does take a significant amount of time
that is not necessary if you want to experiment with changes.   You can submit the building
of the tests with the 'skiptests' argument to the build script.

Thus to get a build as quickly as possible type the following (using `\` as the directory separator, use `/` on Unix machines)
```bat
    .\build skiptests 
```
which will build the Debug flavor which has development time checks (asserts), or 
```bat 
    .\build release skiptests
```
to build the release (full speed) flavor.  You can find more build options with build by using the -? or -help qualifier.   

## Using Your Build

The build places all of its generated files under the `bin` directory at the base of the repository.   There 
is a `bin\Log` directory that contains log files generated during the build (Most useful when the build fails).
The the actual output is placed in a directory like this 

* bin\Product\Windows_NT.x64.Release

Where you can see the operating system and CPU architecture, and the build type are part of the name.   While
the 'raw' output of the build is sometimes useful, normally you are only interested in the Nuget packages 
that were built, which are placed in the directory 

* bin\Product\Windows_NT.x64.Release\.nuget\pkg

These packages are the 'output' of your build.   

There are two basic techniques for using your new runtime.

 1. **Use dotnet.exe and Nuget to compose an application**.   See [Using Your Build](Documentation/workflow/UsingYourBuild.md) for 
 instructions on creating a program that uses 
 your new runtime by using the NuGet packages you just created and the'dotnet' command line interface.  This
 is the expected way non-runtime developers are likely to consume your new runtime.    

 2. **Use corerun.exe to run an application using unpackaged Dlls**. This repository also defines a simple host called
 corerun.exe that does NOT take any dependency on NuGet.   Basically it has to be told where to get all the
 necessary DLLs you actually use, and you have to gather them together 'by hand'.   This is the technique that
 all the tests in the repo use, and is useful for quick local 'edit-compile-debug' loop (e.g. preliminary unit testsing).
 See [Executing .NET Core Apps with CoreRun.exe](Documentation/workflow/UsingCoreRun.md) for details on using 
 this technique.  

## Editing and Debugging

Typically users run through the build and use instructions first with an unmodified build, just to familiarize
themselves with the procedures and to confirm that the instructions work.   After that you will want to actually
make modifications and debug any issues those modifications might cause.   See the following links for more.   

 * [Editing and Debugging](Documentation/workflow/EditingAndDebugging.md) and
 * [Documentation on the .NET Core Runtime](Documentation/README.md)

## Running Tests 

After you have your modification basically working, and want to determine if you have broken anything it is 
time to runt tests.  See [Running .NET Core Tests](Documentation/workflow/RunningTests.md) for more. 

## Contributing to Repository 

Looking for something to work on? The list 
of [up-for-grabs issues](https://github.com/dotnet/coreclr/labels/up-for-grabs) is a great place to start.

Please read the following documents to get started.

* [Contributing Guide](Documentation/project-docs/contributing.md)
* [Developer Guide](Documentation/project-docs/developer-guide.md)

This project has adopted the code of conduct defined by the [Contributor Covenant](http://contributor-covenant.org/) 
to clarify expected behavior in our community. For more information, see the [.NET Foundation Code of Conduct](http://www.dotnetfoundation.org/code-of-conduct).

-------------------
## Related Projects

As noted above, the CoreCLR Repository does not contain all the source code that makes up the .NET Core distribution.
Here is a list of the other repositories that complete the picture.  

* [dotnet/corefx](https://github.com/dotnet/corefx) - Source for the most common classes in the .NET Framework library.
* [dotnet/core-setup](https://github.com/dotnet/core-setup) - Source code for the dotnet.exe program and the policy logic
to launch basic .NET Core code (hostfxr, hostpolicy) which allow you to say 'dotnet SOME_CORE_CLR_DLL' to run the app.  
* [dotnet/cli repo](https://github.com/dotnet/cli) - Source for build time actions supported by dotnet.exe Command line Interface (CLI).
Thus this is the code that runs when you do 'dotnet build', 'dotnet restore' or 'dotnet publish'.
* [dotnet/core-docs](https://github.com/dotnet/core-docs) - Master copy of documentation for 
[http://docs.microsoft.com/en-us/dotnet/](https://docs.microsoft.com/en-us/dotnet/)

## See Also

* [Dotnet.github.io](http://dotnet.github.io) is a good place to discover .NET Foundation projects.
* .NET Core is a [.NET Foundation](http://www.dotnetfoundation.org/projects) project.
* [.NET home repo](https://github.com/Microsoft/dotnet) links to 100s of .NET projects, from Microsoft and the community.
* The [.NET Core repo](https://github.com/dotnet/core) links to .NET Core related projects from Microsoft.
* The [ASP.NET home repo](https://github.com/aspnet/home) is the best place to start learning about ASP.NET Core.

## Important Blog Entries

* [Announcement of .NET Core Open Source Project](http://blogs.msdn.com/b/dotnet/archive/2014/11/12/net-core-is-open-source.aspx)
* [Introducing .NET Core](http://blogs.msdn.com/b/dotnet/archive/2014/12/04/introducing-net-core.aspx)
* [Announcement of CoreCLR](http://blogs.msdn.com/b/dotnet/archive/2015/02/03/coreclr-is-now-open-source.aspx)

## License

.NET Core (including the coreclr repo) is licensed under the [MIT license](LICENSE.TXT).
