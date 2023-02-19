---
title: 【NPM】package.json
---

# package.json

在现在的前端开发中，模块化是必不可少的了，在前端项目中，都有一个熟悉的身影 ———— `package.json`。

一个前端项目就是由一个`package.json`文件开始。

## 是什么

`package.json`是一个项目的清单文件，里面可以是一个工具的配置项，也可以是一个项目的描述，同时它也是模块化的软件包的清单列表。

`package.json`是一个`json`格式的文件，所有在该文件中需要使用标准的`json`文件格式进行声明。

在他人需要知道项目的详情时，则先会读取`package.json`中的信息。

## 结构

最原始的`package.json`的内容

```json
{}
```

因为在应用程序中，`package.json`中的内容并没有强制要求，所以在一个简单的应用程序中，可以不描述任何内容。

### name

表示该项目的**名称**

#### 命名规则

-   长度必须`小于214个字符`
-   如果不是作用域下的包，则不能以`.`或者`_`开头
-   名称中不能包含任何非`url`安全的字符

#### 例子

```json
{
    "name": "my-first-package"
}
```

### version

表示该项目的**版本号**

#### 规则

如果给定的版本号为 `MAJOR.MINOR.PATCH`，则会增加：

-   `MAJOR` 当您进行不兼容的 `API` 更改时
-   `MINOR` 当您以向后兼容的方式添加功能时
-   `PATCH` 补丁版本，当你使向后兼容的 `bug` 修复

预发布和构建元数据的其他标签可以作为 `MAJOR.MINOR.PATCH` 格式的扩展使用。

#### 例子

```json
{
    "version": "1.0.0"
    // "version:": "1.0.0-alpha.1"
    // "version:": "1.0.0-beta.1"
    // "version:": "1.0.0-rc.1"
}
```

### description

表示该项目的**描述**

### license

表示该项目的许可证

如果是开源项目，则应包含这个选项。

#### 例子

```json
{
    "license": "MIT"
}
```

### main

表示模块入口文件

这个路径是相对于包的根目录

如果为空，则默认是`index.js`

#### 例子

```json
{
    "main": "src/index.js"
}
```

### browser

表示浏览器使用的模块

如果是浏览器中使用的模块，而不是`node.js`中可使用的，应该用这个字段来代替`main`字段。

#### 例子

```json
{
    "browser": "src/index.min.js"
}
```

### module

表示项目提供的 es6 模块的入口

与`main`字段可以共存

#### 例子

```json
{
    "module": "src/index.es.js"
}
```

### typings

表示项目提供 `typescript` 声明文件入口

#### 例子

```json
{
    "typings": "src/index.d.ts"
}
```

### types

同 `typings`

### bin

表示脚本命令

-   `key` 为命令名称
-   `value` 为执行的文件

若指定了该字段，则在作用域范围内，调用该命令会映射执行指定的文件。

#### 例子

```json
{
    "bin": {
        "my-test": "src/bin.js"
    }
}
```

```bash
>$ my-test #相当于执行 node src/bin.js
```

### scripts

表示可执行的命令

可以理解用一段脚本用一个别名来代替。

指定后，在使用`npm run <script>`映射脚本命令

#### 特殊情况

如果存在`pre`和`post`开头的别名，则在执行该 `script` 之前或者之后执行命令。

#### 例子

```json
{
    "scripts": {
        "predev": "echo 'pre'",
        "dev": "npm install && npm run test",
        "postdev": "echo 'post'"
    }
}
```

```bash
>$ npm run dev # 相当于执行

#先运行 predev
>$ echo 'pre'

#再运行 dev
>$ npm install && npm run test

# 最后运行 postdev
>$ echo 'post'
```

### dependencies

表示项目依赖项

**应该把需要打包后使用的依赖项放到该配置中**

例如: `vue`/`react`等等

#### 规则

-   `key`为依赖项名称
-   `value`为依赖项版本(需要符合 semver)

```json
{
    "dependencies": {
        "foo": "1.0.0 - 2.9999.9999",
        "bar": ">=1.0.2 <2.1.2",
        "baz": ">1.0.2 <=2.3.4",
        "boo": "2.0.1",
        "qux": "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
        "asd": "http://asdf.com/asdf.tar.gz",
        "til": "~1.2",
        "elf": "~1.2.3",
        "two": "2.x",
        "thr": "3.3.x",
        "lat": "latest",
        "dyl": "file:../dyl"
    }
}
```

#### 例子

```json
{
    "dependencies": {
        "react": "^18.0.0"
    }
}
```

### devDependencies

表示项目开发依赖项

**应该把不需要打包后使用的依赖项放到该配置中**

例如一些项目管理工具 `eslint` / `prettier` 等等

#### 规则

同`dependencies`

#### 例子

```json
{
    "devDependencies": {
        "eslint": "^8.0.0"
    }
}
```

### peerDependencies

表示该项目强依赖的依赖项

表示该项目需要安装指定版本才能正常使用

**包管理工具不一定会自动下载该依赖项，每个包管理工具的策略不一样**

#### 规则

同`dependencies`

#### 例子

```json
// 表示该项目需要安装 react 18.0.0以上版本才能正常使用
{
    "peerDependencies": {
        "react": ">= 18.0.0"
    }
}
```

### optionalDependencies

表示该项目可选择的依赖项

#### 规则

同`dependencies`

#### 例子

```json
{
    "optionalDependencies": {
        "react": ">= 18.0.0"
    }
}
```

### engines

表示该项目需要依赖的 `node` 版本

#### 例子

```json
{
    "engines": {
        "node": ">=0.10.3 <15"
    }
}
```

### os

表示该项目需要依赖的操作系统

#### 例子

```json
{
    "os": ["darwin", "linux"]
}
```

```json
{
    "os": ["!win32"]
}
```

### cpu

表示该项目只能在哪种类型的 `cpu` 下才能运行

#### 例子

```json
{
    "cpu": ["x64", "ia32"]
}
```

### private

表示该项目是不是私有的

如果该项设置为`true`，则不允许发布到 `npm` 上

## 总结

以上的结构并不是完整的结构，是一些比较常用的配置项。

通过了解这些配置项的用途，我们在看他人的项目时，可以帮助我们更快的了解到项目的内容，走出阅读的第一步。
