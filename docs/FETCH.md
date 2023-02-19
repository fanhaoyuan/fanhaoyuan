---
title: 【Fetch】深入 Fetch API
---

# 深入 Fetch API

从第一个实现 Fetch API 的浏览器开始计算，现在已经是第 7 个年头了。现在各大浏览器对 Fetch API 的支持已经相当不错了。ES6 的不断普及，我们基本上在编码上都会使用 Promise 和 Async/Await，异步编程的今天，可能已经忘记了如何去编写一个 XMLHttpRequest。Fetch API 的易用性让不少小伙伴转入了它的怀抱。

## 简单的 Fetch 请求

```ts
fetch('https://bar.com/foo', {
    credentials: 'same-origin',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ foo: 'bar' }),
}).then(console.log);
```

这是一个简单的 Fetch 请求， 只要传入一个配置项，就能创建出一个简单直观的 fetch 请求。 而且相对于 XMLHttpRequest 来说，提供了更多的控制参数，例如 credentials 和 redirect 等。可以让使用者去配置是否携带 cookies 或者是跳转等。而且 Fetch API 天然的异步，避免了回调地狱。很符合今天异步编程的大环境。

所以我们可以通过 Async/Await 去简化我们的代码。

```ts
const res = await fetch('https://bar.com/foo', {
    credentials: 'same-origin',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ foo: 'bar' }),
});

console.log(res);
```

如果现在使用 XMLHttpRequest，我们需要调用 open() 方法开启一个请求, 然后调用其他的方法或者设置参数来定义请求，最后调用 send() 方法发起请求，再在 onload 或者 onreadystatechange 事件里处理数据。

一套组合拳下来，只能说 Fetch API 真香。

## 进阶功能

从上文来看，Fetch API 相对比与 XMLHttpRequest 具有不少的优势，但是 XMLHttpRequest 给我们提供了 3 个简单的方法来进行取消请求、超时取消、下载进度功能。我们只需要简单的配置就可以是心啊这三个功能。但是 Fetch API 却没有提供这 3 个功能，我们只能手动实现这 3 个功能。

### 中断请求

在 XMLHttpRequest 中， 我们调用一个 abort() 方法，就可以中断请求，XMLHttpRequest 还有 onabort 事件，可以监听请求的中断并做出响应。

在 Fetch API 中，我们只能使用 AbortController 与 AbortSignal 来进行请求的中断

```ts
const abortController = new AbortController();

fetch('https://bar.com/foo', {
    credentials: 'same-origin',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ foo: 'bar' }),
    signal: abortController.signal, // 连接 abortController
}).then(console.log);

abortController.abort(); // 取消请求

abortController.signal.onAbort = console.log; // 监听取消事件
```

通过 AbortSignal 来连接 AbortController 与 fetch 实例，AbortSignal 中的 onAbort 事件会在请求中断之后触发。

### 超时中断

既然我们已经实现了中断请求，我们只需要使用 setTimeout 就可以模拟一个超时时间，实现超时中断.

```ts
const abortController = new AbortController();

fetch('https://bar.com/foo', {
    credentials: 'same-origin',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ foo: 'bar' }),
    signal: abortController.signal, // 连接 abortController
}).then(console.log);

setTimeout(() => abortController.abort(), 10 * 1000); // 10s 后取消请求

abortController.signal.onAbort = console.log; // 监听取消事件
```

### 获取下载进度

在所有配置项中，我们都不能看到有关于下载进度的配置。那我们应该如何去实现这个功能呢？

在下一节中，会介绍和它相关的 Streams API，了解了之后，我们再来揭晓如果获取 fetch 下载进度。

## 延伸阅读

-   [fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch)
-   [fatcher](https://github.com/fanhaoyuan/fatcher)
-   [caniuse](https://caniuse.com/fetch)
-   [mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API)
