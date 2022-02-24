# 在 `Internet Computer` 上部署静态网页

开始之前，请确保你安装了`Dfinity Canister SDK`（即`dfx`）、配置好了`Cycles`钱包。

## 创建一个项目

让我们试着创建一个简单的静态页面，然后用`dfx`把它部署到`Internet Computer`主网。

1. 创建一个名为`static-ic-website`的文件夹。
2. 在`static-ic-website`文件夹里，创建一个新的文件夹，名字为`assets`。
3. 在`assets`文件夹里，创建4个文件：
    - `index.html`
    - `page-2.html`
    - `script.js`
    - `style.css`

## 添加内容

让我们从`index.html`文件开始，复制下面的代码到刚才创建的文件中：

**index.html**

``` html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Static IC Website</title>
        <link rel="stylesheet" href="style.css">
    </head>
    <body>
        <h1>My First IC Website</h1>
        <p>Styles are loaded from a stylesheet</p>
        <p id="dynamic-content"></p>
        <a href="page-2.html">Page 2</a>
        <script src="script.js"></script>
    </body>
</html>
```

**page-2.html**

``` html
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Page 2</title>
   <link rel="stylesheet" href="style.css">
</head>
<body>
   <h1>This is page 2</h1>
   <a href="/">Back to index</a>
</body>
</html>
```

**script.js**

``` js
document.querySelector("#dynamic-content").innerText =
 "This paragraph is dynamically rendered using JavaScript";
```

**style.css**

``` css
body {
   font-family: sans-serif;
   font-size: 1.5rem;
}

p:first-of-type {
   color: #ED1E79;
}
```

## 配置`dfx`

你需要手动配置`dfx`来让它能够上传这些内容到罐头中，这样罐头部署到`Internet Computer`才有意义。在项目的根目录`static-ic-website`中，创建一个新文件`dfx.json`，复制下面的内容粘贴到里面：

**dfx.json**

``` json
{
   "canisters": {
       "website": {
           "type": "assets",
           "source": ["assets"]
       }
   }
}
```

这个`json`做了两件事：

1. 告诉`dfx`如何创建新的罐头。
2. 上传`assets`文件夹到罐头中

如果你想添加新的资源文件夹，你可以直接把它添加到`assets`目录中，或者，创建一个新的目录，然后把这个新的目录添加到`dfx.json`的`source`数组里。

## 部署你的网站

确保你在项目的根目录（`dfx.json`所在的路径），运行一下命令：

``` bash
dfx deploy --network ic
```

不出意外的话，终端会输出下面这样的信息表示部署成功：

``` text
...

Uploading assets to asset canister...
Starting batch.
Staging contents of new and changed assets:
  /index.html 1/1 (501 bytes)
  /index.html (gzip) 1/1 (317 bytes)
  /page-2.html 1/1 (373 bytes)
  /page-2.html (gzip) 1/1 (258 bytes)
  /script.js 1/1 (117 bytes)
  /style.css 1/1 (102 bytes)
Committing batch.
Deployed canisters.
```

## 查看你的网站

通过下面的命令获取罐头`id`：

``` bash
dfx canister --network ic id website
```

复制这个`id`，插入到`https://<canister-id>.ic0.app`，并在浏览器打开就能查看你刚刚部署好的网页！