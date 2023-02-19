---
title: 【NPM】从 0 开始发布一个 NPM 包
---

# 从 0 开始发布一个 NPM 包

## 建立一个项目

首先，建立一个空的项目目录，进入目录。

```bash
>$ mkdir my-project # 创建目录

>$ cd my-project # 进入目录
```

然后初始化一个项目

```bash
>$ npm init -y # 初始化项目
```

运行之后，在目录下会生成一个`package.json`文件。

生成出来的目录结构如下：

```json
{
    "name": "my-project",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC"
}
```

到这里，就初始化了一个最简单的项目。

## 编写代码

加入辅助插件，项目所需的依赖，编写代码。

所有模块编写完以后，应该从主入口文件导出，只有导出之后，在安装该项目为依赖之后，才能从`node_modules`中直接导入使用。

## 构建代码

如果构建一个插件，可以使用`webpack`或者是`rollup`进行构建。

## 重写入口

在`package.json`中修改入口。把文件指向到对应的构建产物中。

```json
{
    //cjs入口
    "main": "${dest}/index.cjs.js",
    //esm入口
    "module": "${dest}/index.esm.js",
    //umd入口
    "browser": "${dest}/index.min.js",
    //types入口
    "typings": "$${dest}/index.d.ts"
}
```

`main`字段指定的是`commonJs`模块的入口，一般情况下，做出来的`npm`包最好是`commonJs`为主入口。

`browser`字段指定的是`umd`模块的入口，一般`umd`模块是通过浏览器中的`<script></script>`标签引入的。

`module`字段指定的是`es6`模块的入口，随着浏览器的兼容性不断提高，`es6`模块的支持也越来越好。

`typings`字段指定的是该项目的`*.d.ts`声明文件。如果项目使用`typescript`编写的话，都可以从代码中生成，会给予用户更多的自动补全。

## 指定 File

在`package.json`中指定哪些文件是发布的

```json
{
    "files": ["${dest}"]
}
```

## 发布/更新

在发布之前，我们要进行登录。

登录后则进行发布。

```bash
>$ npm login

>$ npm publish
```

## 撤销发布

如果发布错误，则可以进行撤销发布操作

```bash
>$ npm unpublish <package name> --force
```

## 使用

我们发布了`npm`包之后，我们可以在其他项目中进行安装

```bash
>$ npm install [package]
```

通过引用，则直接可以在项目中使用所编写的功能。
