# 下载中心（Download Center）

目前，你可以直接从本地电脑的终端下载`Dfinity Cniaster Software Development Kit`（`SDK`）。使用此安装选项，你可以以任何用户身份登录，无需其它软件。

## 直接从终端安装最新版本（Install the latest directly from a terminal）

-----

从终端直接下载和安装：

1. 打开终端。例如，在`MacOS`上打开`Applications`文件夹，然后打开`Utilities`并双击`Terminal`。
2. 通过运行以下命令下载并安装`SDK`：
    ``` bash
    sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
    ```

## 从终端安装指定版本（Install a specific version from a terminal）

-----

如果你想安装特定版本，例如，要针对以前的版本进行测试，你可以修改安装命令以包含一个版本。

要从终端`shell`下载和安装特定版本：

1. 打开终端。
2. 将`DFX_VERSION`环境变量设置为你要作为`curl`命令前缀安装的`DFINITY Canister SDK`包的版本。
    例如，要安装`0.9.2`版，你可以运行下面的命令：
    ``` bash
    DFX_VERSION=0.9.2 sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
    ```
    > 注意，如果你使用`DFX_VERSION`环境变量来安装尚未公开提供的`DFINITY Canister SDK`版本，请参阅[本文](https://smartcontracts.org/docs/http-middleware.html)以了解更改的概述。

## 下一步（Next steps）

-----

有关后续步骤的信息，请参阅[`SDK`开发人员工具](https://smartcontracts.org/docs/developers-guide/sdk-guide.html)中的[教程](https://smartcontracts.org/docs/developers-guide/tutorials-intro.html)。

要在开始试验代码之前了解有关为`IC`编写程序的更多信息，请参阅[概念](https://smartcontracts.org/docs/developers-guide/concepts/concepts-intro.html)。

有关使用`Motoko`为`IC`编写程序的信息，请参阅[`Motoko`编程语言指南](For information about writing programs for the Internet Computer using Motoko, see the Motoko Programming Language Guide.)。