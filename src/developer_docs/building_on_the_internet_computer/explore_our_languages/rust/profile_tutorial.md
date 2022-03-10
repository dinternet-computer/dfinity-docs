# 添加和搜索简单记录（Adding and searching simple records）

在本教程中，你将编写一个`dapp`，它提供一些基本功能来添加和检索由名称（`name`）、描述（`description`）、关键字（`keyword`）数组组成的简单`profile`记录。

该程序支持以下功能：

- `update`函数使你可以添加由`name`、`description`、`keyword`组成的`profile`。
- `getSelf`函数返回与函数调用者关联的主体的`profile`。
- `get`函数执行一个简单的查询以返回与传递给它的`name`匹配的`profile`。对于此函数，入参`name`必须与`profile`的`name`完全相同才返回该`profile`。
- `search`函数能执行更复杂的`profile`查询以返回与任何`profile`字段中指定的全部或部分文本匹配的`profile`。

本教程提供了一个简单示例，说明如何使用`Rust CDK`接口和宏来简化在`rust`中编写`IC`程序。

本教程演示：

- 如何使用`candid`接口描述语言以配置文件的形式作为记录和关键字数组表示稍微复杂的数据。
- 如何编写一个带有部分字符串匹配的简单搜索函数。
- `profile`如何与特定主体关联。

## 开始之前（Before you begin）

------

## 创建一个新项目（Create a new project）

------

## 修改默认项目（Modify the default project）

------

### 替换默认的`dapp`（Replace the default dapp）

## 更新接口文件（Update interface description file）

-----

## 打开本地罐头执行环境（Start the local execution environment）

------

## 注册、构建、部署你的项目（Register, build, and deploy your project）

------

## 调用罐头上的函数（Call functions on the deployed canister）

------

## 为新的主体添加`profile`（Adding profiles for new identities）

-------

## 扩展（Extending the sample dapp）

------

## 停止本地罐头执行环境（Stop the local execution environment）


-------

