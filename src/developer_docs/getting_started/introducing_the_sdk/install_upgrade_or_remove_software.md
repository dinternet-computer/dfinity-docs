# 安装、升级或移除`SDK`（Install, upgrade, or remove software）

如快速入门中所述，你可以通过在终端`shell`中运行命令行下载并安装最新版本的`dfinity canister sdk`软件包。本节的主题是提供有关安装、升级和删除该`SDK`的有关信息。

## 在终端直接安装最新版（Install the latest directly from a terminal）

-----

下载并安装的步骤：

1. 在本地电脑打开一个终端。
2. 运行下面的命令安装`SDK`：
    ``` bash
    sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
    ```

## 在终端中安装指定版本的`SDK`

-----

如果你需要安装特定版本的`SDK`，例如，针对以前的版本进行测试，你可以修改安装命令以包含一个特定的版本：

1. 在本地电脑打开一个终端。
2. 将`DFX_VERSION`环境变量设置为你要作为`curl`命令前缀安装`dfinity canister SDK`包的版本：
    ``` bash
    DFX_VERSION=0.9.3 sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
    ```

> 注意，如果你使用`DFX_VERSION`环境变量来安装尚未公开提供的`SDK`版本，请参阅本文以了解更改的概述。

## 到底安装了什么（What gets installed）

------

`Dfinity canister SDK`安装脚本会在本地计算机的默认位置安装多个组件。下表描述了安装脚本安装的开发环境组件：

|组件|描述|默认位置|
|-----|-----|------|
|dfx|`dfx`命令行工具（`cli`）|`/usr/local/bin/dfx`|
|moc|`motoko`的运行时编译器|`~/.cache/dfinity/versions/<VERSION>/moc`|
|replica|`IC`的本地二进制文件|`~/.cache/dfinity/versions/<VERSION>/replica`|
|uninstall.sh|用来删除`SDK`的脚本|`~/.cache/dfinity/uninstall.sh`|
|versions|缓存目录，其中包含你安装的每个版本的`SDK`的子目录|`~/.cache/dfinity/versions`|

### 每个版本的子目录里的核心组件（Core components in a versioned directory）

`~/.cache/dfinity/versions`目录存储`SDK`的一个或多个版本化子目录。每个版本子目录包含特定版本的`SDK`所需的所有目录和文件。例如，如果你列出`~/.cache/dfinity/versions/0.9.3`目录的内容，你将看到以下核心组件：

``` text
otal 349192
drwxr-xr-x  17 pubs  staff       544 Mar 15 11:55 .
drwxr-xr-x   4 pubs  staff       128 Mar 25 14:36 ..
drwxr-xr-x  49 pubs  staff      1568 Mar 15 11:55 base
drwxr-xr-x  20 pubs  staff       640 Mar 15 11:55 bootstrap
-r-x------   1 pubs  staff  66253292 Mar 15 11:55 dfx
-r-x------   1 pubs  staff  10496256 Dec 31  1969 ic-ref
-r-x------   1 pubs  staff   5663644 Dec 31  1969 ic-starter
-r-x------   1 pubs  staff      9604 Dec 31  1969 libcharset.1.0.0.dylib
-r-x------   1 pubs  staff     38220 Dec 31  1969 libffi.7.dylib
-r-x------   1 pubs  staff    668300 Dec 31  1969 libgmp.10.dylib
-r-x------   1 pubs  staff    958248 Dec 31  1969 libiconv.2.4.0.dylib
-r-x------   1 pubs  staff      4200 Dec 31  1969 libiconv.dylib
-r-x------   1 pubs  staff     96900 Dec 31  1969 libz.1.2.11.dylib
-r-x------   1 pubs  staff  15417684 Dec 31  1969 mo-doc
-r-x------   1 pubs  staff  14634020 Dec 31  1969 mo-ide
-r-x------   1 pubs  staff  15111508 Dec 31  1969 moc
-r-x------   1 pubs  staff  49404128 Dec 31  1969 replica
```

### `Motoko`基础库目录（Motoko base directory）

`SDK`的版本化子目录中的`base`目录包含与该版本的`SDK`兼容的`motoko`基础库模块。由于`Motoko`基础库正在快速迭代，你应该只使用与你已安装的`SDK`版本一起打包的基础库模块。

### `Bootstrap`目录（Bootstrap directory）

`Bootstrap`目录包含已弃用的`Web`服务器代码。从版本`0.7.0`开始，代理可以调用`HTTP`中间件服务器而不是引导代码。此更改使罐头能够直接响应`HTTP`请求并更像传统的基于`Web`的应用程序一样运行。

## 升级到最新版本（Upgrading to the latest version）

-------

如果在初始安装后有新版本的`SDK`可供下载，你应尽早安装更新版本，以便尽快获得最新的修复和增强功能。你可以使用`dfx upgrade`命令将你当前安装的版本与可供下载的最新版本进行比较。如果有更新版本的`dfx`可用，`dfx upgrade`命令会自动下载并安装最新版本。

请注意，你无需在安装新版本之前卸载软件。但是，如果你想执行全新安装而不是升级，你可以先按照删除软件一节中的说明卸载软件，然后重新运行下载和安装命令。

有关最新版本中的功能和修复信息，请参阅发行说明。

## 卸载软件（Removing the software）

-----

安装`SDK`时，安装脚本会将所需的二进制文件放在本地缓存目录中并创建缓存。你可以通过运行位于`~/.cache`文件夹中的卸载脚本从本地计算机中删除`SDK`二进制文件和缓存。

例如：

``` bash
~/.cache/dfinity/uninstall.sh
```

如果你因为想立即重新安装干净版本的`dfx`而卸载，你可以运行以下命令：

``` bash
~/.cache/dfinity/uninstall.sh && sh -ci "$(curl -sSL https://sdk.dfinity.org/install.sh)"
```
