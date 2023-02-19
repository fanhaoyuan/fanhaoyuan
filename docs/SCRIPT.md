---
title: 【HTML】HTML 中的 JavaScript
---

# HTML 中的 JavaScript

## script 元素

在 `HTML` 中使用 `JavaScript` 的主要方法是使用 `script` 标签。

### 属性

#### src

表示要执行的外部脚本。

##### 例子

```html
<script src="index.js"></script>
```

#### async

表示立即下载脚本，但是**异步**解析该脚本。**只对外部脚本有效。**

所以这个脚本的解析过程不会阻塞其他资源或者脚本的加载。

但是使用 `async` 属性，并不能保证脚本的加载完成顺序。所以如果有依赖关系的脚本，不应使用该属性。

##### 例子

```html
<!-- async 属性无效 -->
<script async>
    (function () {
        console.log('function');
    })();
</script>
```

```html
<!-- async 属性有效 -->
<script src="index.js" async></script>
```

#### charset

表示 `src` 属性指定的字符集。

这个属性基本上很少使用。

##### 例子

```html
<script src="index.js" charset="utf-8"></script>
```

#### crossorigin

表示相关请求的跨域共享配置。

默认是**不使用**跨域共享配置。

该项有两个有效值：

-   `anonymous` 表示该文件请求不必设置凭证。
-   `use-credentials` 表示该文件请求需要包含凭证。

如果该项是空值或者是无效值，则默认使用`anonymous`。

##### 例子

```html
<script src="index.js" crossorigin></script>
<script src="index.js" crossorigin="anonymous"></script>
<script src="index.js" crossorigin="use-credentials"></script>
```

#### defer

表示立即下载脚本，但是延迟解析脚本。除了**IE7 之前版本，均只对外部脚本有效。**

指定该值后，脚本会延迟到文档解析完成和显示之后再执行。

##### 例子

```html
<!-- defer 属性无效 -->
<script defer>
    (function () {
        console.log('function');
    })();
</script>
```

```html
<!-- defer 属性有效 -->
<script src="index.js" defer></script>
```

#### integrity

表示允许对接收到的资源文件进行校验，确定资源文件的完整性。

这个属性可以用作校验 `CDN` 返回的脚本，确保内容不会提供恶意内容。

#### type

表示脚本的内容类型。一般情况下，值为`text/javascript`。

如果这个值为`module`，在脚本中可以出现`import`/`export`模块关键字。

### 使用方式

#### 行内脚本

行内脚本是直接在 `script` 标签中写脚本内容。

```html
<script>
    function say() {
        console.log('hello world!');
    }
</script>
```

在标签中，会从上至下解析脚本内容。

要注意是，在代码中不能出现`</script>`字符。因为出现了这个字符，浏览器会认为脚本内容已经结束，会导致浏览器解析错误。

如果想要出现`</script>`内容，可以进行转义。

```html
<script>
    function say() {
        console.log('<\/script>');
    }
</script>
```

#### 外部脚本

在`script`标签中使用`src`属性，可以引入外部脚本。跟行内脚本一致的是，加载外部脚本会阻塞文档中其他内容，直至解析完成。

如果使用了`src`属性，会把行内的脚本内容忽略掉。

### 使用位置

#### 在 head 中

如果`script`标签放到`head`标签里面，则会在文档渲染之前，下载并解析完成。

这样会有一个问题，就是如果有很多`script`标签在`head`中，会解析很长时间，导致页面是空白状态，也就是所说的白屏时间长。

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <script>
            function say() {
                console.log('hello world!');
            }
        </script>
        <script src="index.js"></script>
        <title>Document</title>
    </head>
</html>
```

#### 在 body 中

为了解决白屏时间过长的问题，可以把`script`标签放到`body`标签中

这样会在处理脚本内容之前完全渲染页面。从而缩短白屏时间。

所以一般推荐不需要立即执行的脚本放到`body`中。

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <script src="index.js"></script>
        <title>Document</title>
    </head>
    <body>
        <script>
            function say() {
                console.log('hello world!');
            }
        </script>
    </body>
</html>
```

### 延迟加载

一般情况下，脚本会按顺序来加载并解析，如果我们有不相互依赖的脚本，我们可以使用`defer`属性推迟加载。

脚本会在`DOMContentLoaded`事件之前完成。

```html
<script src="index.js" defer></script>
```

### 异步执行

一般情况下，脚本在加载的时候会阻塞页面其他行为，所以如果有大计算量的脚本，会白屏时间长，所以我们可以使用`async`属性来异步加载，不阻塞其他内容。

要注意的是，异步执行并不能保证执行完成顺序，所以有依赖的脚本要注意这个特点。

```html
<script src="index.js" async></script>
```

### 动态加载

我们可以使用 JavaScript 去创建`script`脚本。

```html
<script>
    function create() {
        var script = document.createElement('script');
        script.src = 'index.js';
        document.head.appendChild(script);
    }
</script>
```

我们可以在浏览器可交互的情况下进行动态的脚本加载，可以进行一系列的操作。

## noscript 元素

表示在脚本不被支持的时候显示的内容。

`noscript` 标签中的内容会在两种情况下显示：

-   浏览器不支持脚本
-   浏览器被禁用脚本

### 例子

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
    </head>
    <body>
        <noscript>
            <p>这段话只会在脚本不被支持或者脚本被禁用时看到。</p>
        </noscript>
    </body>
</html>
```
