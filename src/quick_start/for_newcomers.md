# 准备工作

如果你是第一次在`Internet Computer`上开发，你首先需要一些额外的步骤搭建开发环境。

> 注意，这里假设你使用的是MacOS。

在安装`Dfinity Canister SDK`之前，你需要了解：

- 如何打开一个终端，以及在终端中运行命令。
- 如何安装包和相关依赖。
- 创建新目录、在目录之间跳转。
- 查看和更新环境变量。

# 打开一个终端

在`MacOS`上打开一个终端的步骤：

1. 打开`Finder`。
2. 点击`Application`，打开`Utilities`，双击`Terminal`。或者，同时按下`cmd` + `spacebar`打开搜索栏，输入`terminal`也能打开终端。
3. 在终端中输入`pwd`然后回车，查看终端当前所处的目录。
    ``` bash
    pwd
    ```    

# 安装相关的包

`Homebrew`是`MacOS`上的包管理器，它使得在`MacOS`上安装和更新软件包非常简单。`Homebrew`可能需要你手动安装。

`nodejs`这个包提供`javascript`的前端开发的运行时环境和模块。`nodejs`不是必须的，除非你想为你的程序开发一个好看的前端。

> 注意，如果你的开发环境不是`MacOS`而是`Linux`，请选择相应的包管理器，比如`apt`。

## 检查和安装软件包

1. 在终端，通过下面这条命令检查你是否安装了`Homebrew`包管理器：
    ``` bash
    brew --version
    ```

    如果返回正常的版本信息，说明你已经安装了`Homebrew`，否则，输入下面的命令安装`Homebrew`：
    ``` bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
    ```    
2. 在终端，通过下面这条命令检查你是否安装了`nodejs`：
    ``` bash
    node --version
    ```
    如果没有返回版本信息，通过下面这条命令安装`nodejs`：
    ``` bash
    brew install node
    ```

# 创建工作目录

通常情况下，为了简介，你需要为每个新项目创建一个新的工作目录。

1. 在终端输入下面的命令，创建一个名为`ic-projects`的目录：
    ``` bash
    mkdir ic-projects
    ```
2. 进入该目录：
    ``` bash
    cd ic-projects
    ```

# 查看`PATH`

`PATH`环境变量是操作系统中非常重要的环境变量：它指明了操作系统中可执行程序的存放位置。所以你需要将`Dfinity Canister SDK`————`dfx`的路径添加到`PATH`，以至于操作系统能感知到`dfx`的位置，这样终端和其他程序就能调用`dfx`了。

将`dfx`添加到`PATH`环境变量的步骤：

1. 确定`dfx`的位置：
    ``` bash
    which dfx
    ```
2. 添加`dfx`到`PATH`环境变量中：
    ``` bash
    export PATH="<path-to-directory-for-dfx>:$PATH"
    ```
