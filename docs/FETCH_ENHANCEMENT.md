---
title: 【Fetch】增强你的 fetch 请求
---

# 增强你的 fetch 请求

## 前言

在 Node 17.5.0 开始，fetch 被 nodejs 加入支持，几个相关的 api 在 18.x 相继被加入到支持列表中。

-   fetch >= 17.5.0
-   FormData >= 17.6.0
-   Headers>= 17.5.0
-   ReadableStream >= 18.0.0
-   Response >= 17.5.0
-   Request >= 17.5.0

虽然上面这几个 API 都是在 Stability 1 的实验性状态，但是在使用上已经和在浏览器中使用 fetch 很相同（不能说很像，只能说一摸一样）

```ts
fetch('/url').then(res => res.json());
```

在 fetch 没有被支持的话，我们一般会在 node 和浏览器中使用 axios（基于 xhr 或者是 http）来进行请求发送，或者想使用 fetch 的话就会使用 node-fetch。

现在，我们可以在 node 和浏览器中使用相同的请求方式，而且并不需要安装什么依赖，相同的语法，相同的响应，而且天然支持 Promise, 提高了开发效率。

但是 fetch 还是一个比较底层的 api，对响应的处理还需要自己进一步的封装。

所以，我们今天介绍如何增强 fetch 的一些功能。

## 取消请求

我们在一些上传或者下载的操作的时候，或者一些耗时比较长的请求任务时，我们可能会需要取消当前请求。

这个时候，我们需要用到一个叫 AbortController 的内置对象。

> AbortController 接口表示一个控制器对象，允许你根据需要中止一个或多个 Web 请求。 [AbortController](https://developer.mozilla.org/zh-CN/docs/Web/API/AbortController)

```ts
const abortController = new AbortController();

fetch('/get', {
    signal: abortController.signal, // 连接 abortController
})
    .then(res => {
        // 成功响应
        console.log(res);
    })
    .catch(err => {
        if (err instanceof DOMException && err.name === 'AbortError') {
            // 取消请求会返回一个 DOMException
            return;
        }

        // 其他错误
    });

abortController.abort(); // 取消请求
```

通过 AbortController 我们可以简单的取消掉 fetch 请求。

## 请求超时

我们在如果在长时间没有得到响应的请求时，可能会在请求超时的时候进行一些提示或者是取消请求。

在上文已经实现了如何取消一个 fetch 请求。

所以，我们可以很简单的利用计时器完成一个请求超时的功能实现。

```ts
const abortController = new AbortController();

let timer: number | null = null;

fetch('/get', {
    signal: abortController.signal, // 连接 abortController
})
    .then(res => {
        // 成功响应
        console.log(res);

        if (timer) {
            clearTimeout(timer); //请求成功 去掉计时器
        }
    })
    .catch(err => {
        if (err instanceof DOMException && err.name === 'AbortError') {
            // 取消请求会返回一个 DOMException
            return;
        }

        // 其他错误
    });

timer = setTimeout(() => {
    abortController.abort();
}, 1000 * 10); // 10 秒后取消请求
```

## 并发限制

在一些场景下，我们上一次的请求还没有返回，而再次发出请求时，如果不取消掉前面的请求，第二个请求先于第一个请求响应的话，则会出现数据不一致的情况发生。

所以我们在发送相同请求的时候，我们可以对前一个请求进行取消操作。

取消请求已经在上文有介绍。

```ts
const concurrencyMap: Record<string, { signal: AbortSignal; abort: () => void }[]> = {};

function concurrencyFetch(url: string, options?: RequestInit = {}) {
    const group = `${url}_${options.method ?? 'GET'}`;

    const { signal, abort } = new AbortController();

    concurrencyMap[group] ??= [];

    if (concurrencyMap[group].length) {
        concurrencyMap[group].forEach(item => item.abort());
    }

    concurrencyMap[group].push({ abort, signal });

    signal.addEventListener('abort', () => {
        concurrencyMap[group] = concurrencyMap[group].filter(item => item.signal !== signal);

        if (!concurrencyMap[group].length) {
            delete concurrencyMap[group];
        }
    });

    return fetch(url, {
        ...options,
        signal, // 连接 abortController
    });
}

concurrencyFetch('/get')
    .then(res => {
        console.log(res);
    })
    .catch(err => {
        if (err instanceof DOMException && err.name === 'AbortError') {
            // 取消请求会返回一个 DOMException
            return;
        }

        // 其他错误
    });
```

## 下载进度

在下载的场景中，我们很可能需要展示给用户知道当前的下载进度。但是 fetch 没有提供这样的一个 api 给我们进行获取。

这个功能的核心为在响应头中返回一个`Content-Length`字段。通过总大小与已获取量的大小则可以模拟出当前的下载进度。

```ts
async function readStreamByChunk<T = Uint8Array>(readableStream: ReadableStream<T>, callback: (chunk: T) => void) {
    async function read(reader: ReadableStreamReader<T>) {
        const { value, done } = await reader.read();

        if (done || !value) {
            return;
        }

        callback(value);

        await read(reader);
    }

    return read(readableStream.getReader());
}

fetch('/get').then(res => {
    const total = ~~(result.headers.get('Content-Length') || 0);

    if (total > 0) {
        let current = 0;

        const clonedResponse = res.clone(); //克隆一份新的响应来处理

        readStreamByChunk(clonedResponse.body, (current, total) => {
            console.log(current); // 当前已经获取了多少
            console.log(total); // 总共有多少
            console.log(current / total); // 当前进度是多少
        });
    }

    // 继续处理 res
});
```

## 响应缓存

在一些不高频改动的数据中，我们可以在客户端进行对结果的缓存，在有效时间内，每次命中缓存 key 的请求会把响应数据缓存起来，下一次请求的时候，优先返回该数据。

会有效的提高响应效率，在一些场景下特别有用。

一般是`GET`等无副作用请求才会进行缓存。

```ts
const cacheMap = new Map<string, Response>();

const timer: Record<string, number> = {};

async function sendFetch(url: string, options?: RequestInit = {}) {
    if (options.method === 'GET') {
        if (cacheMap.has(url)) {
            return cacheMap.get(url).clone();
        }

        const response = await fetch(url, options);

        cacheMap.set(url, response.clone());

        timer[url] = setTimeout(() => {
            cacheMap.delete(url);
            delete timer[url];
        }, 1000 * 60 * 60); //过期时间

        return response;
    }

    return fetch(url, options);
}

sendFetch('/get', { method: 'GET' }).then(res => {
    // 这里是缓存之后的结果
    console.log(res);
});
```

## 组合使用

在上面单独封装功能不会很复杂，但是在请求的时候，可能需要多个功能一起使用。

而且 fetch 本身也需要对 400、500 等响应进行封装。

所以我们需要封装成一个方法才可以正常使用。

[fatcher](https://github.com/fanhaoyuan/fatcher)这个库利用中间件把这个功能都封装起来，类 koa 的中间件设计，可以在中间件中实现自定义拦截器，声明式中间件，可以按需使用中间件，使用方式与 fetch 基础请求一致，增强了 fetch 的功能，与日常的使用行为一致。

## 总结

虽然 fetch 没有给我们提供一些已经封装好的方法，我们通过一些简单的方式就可以对 fetch 进行一些常用功能的封装。

fetch 天然的异步和简单的 api 是它的优势之处。虽然 fetch 现在仍有不足之处，但是随着浏览器和 nodejs 的支持度越来越高，相信 fetch 会越来越完善的。
