---
title: 【Streams】基于 Streams API 的流式播放器
---

# 基于 Streams API 的流式播放器

在没有 Streams API 之前，我们必须在播放视频之前下载完整的视频文件。

但是在有 Streams API 之后， 我们可以边下载视频，边观看视频，就像我们现在在视频网站中观看视频一样。

## 基于 xhr 播放视频

> [这里有一个 XHR 下载播放视频的例子](https://nickdesaulniers.github.io/netfix/demo/bufferAll.html)

在 XHR 中，我们要观看视频，首先要等待整个视频文件都下载完成，然后把整个视频转换为 Blob URL 才能播放这个视频。

首先。我们先用 XHR 创建这个例子。

第一步先创建一个简单的 HTML 文件

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
    </head>
    <body></body>
</html>
```

我们利用的是 MediaSource 来进行视频的加载，所以，我们要先判断视频能不能加载当前视频。

```js
const mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';

if ('MediaSource' in window && MediaSource.isTypeSupported(mimeCodec)) {
    // 进行操作
} else {
    throw new Error(`Unsupported MIME type or codec: ${mimeCodec}`);
}
```

我们在 HTML 中再创建一个 video 标签

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
        <video controls id='video'>
    </body>
</html>
```

创建一个简单的 xhr 请求，用于下载视频。

```js
function xhr(url, cb) {
    const xhr = new XMLHttpRequest();
    xhr.open('get', url);
    xhr.responseType = 'arraybuffer';
    xhr.onload = function () {
        cb(xhr.response);
    };
    xhr.send();
}
```

接下来我们应该创建一个 MediaSource 来用于加载视频

```js
function sourceOpen(_) {
    //console.log(this.readyState); // open
    var mediaSource = this;
    var sourceBuffer = mediaSource.addSourceBuffer(mimeCodec);
    xhr(assetURL, function (buf) {
        sourceBuffer.addEventListener('updateend', function (_) {
            mediaSource.endOfStream();
            video.play();
            //console.log(mediaSource.readyState); // ended
        });
        sourceBuffer.appendBuffer(buf);
    });
}

const mediaSource = new MediaSource();
//console.log(mediaSource.readyState); // closed
video.src = URL.createObjectURL(mediaSource);
mediaSource.addEventListener('sourceopen', sourceOpen);
```

上面的例子是当 mediaSource 已经被 video 标签加载之后，进行视频资源的下载。

下载完成之后，调用 `mediaSource.endOfStream()` 方法进行视频的加载完成广播。

`sourceBuffer.appendBuffer(buf)` 是加载我们在 xhr 拿到视频的 arrayBuffer。

完整代码如下：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
    </head>
    <body>
        <video controls></video>
        <script>
            var video = document.querySelector('video');

            var assetURL = 'frag_bunny.mp4';
            // Need to be specific for Blink regarding codecs
            // ./mp4info frag_bunny.mp4 | grep Codec
            var mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';

            if ('MediaSource' in window && MediaSource.isTypeSupported(mimeCodec)) {
                var mediaSource = new MediaSource();
                //console.log(mediaSource.readyState); // closed
                video.src = URL.createObjectURL(mediaSource);
                mediaSource.addEventListener('sourceopen', sourceOpen);
            } else {
                console.error('Unsupported MIME type or codec: ', mimeCodec);
            }

            function sourceOpen(_) {
                //console.log(this.readyState); // open
                var mediaSource = this;
                var sourceBuffer = mediaSource.addSourceBuffer(mimeCodec);
                fetchAB(assetURL, function (buf) {
                    sourceBuffer.addEventListener('updateend', function (_) {
                        mediaSource.endOfStream();
                        video.play();
                        //console.log(mediaSource.readyState); // ended
                    });
                    sourceBuffer.appendBuffer(buf);
                });
            }

            function fetchAB(url, cb) {
                console.log(url);
                var xhr = new XMLHttpRequest();
                xhr.open('get', url);
                xhr.responseType = 'arraybuffer';
                xhr.onload = function () {
                    cb(xhr.response);
                };
                xhr.send();
            }
        </script>
    </body>
</html>
```

从例子的代码里可以看到，我们是有几个步骤

-   创建 MediaSource 实例
-   创建 ObjectURL 来加载 MediaSource
-   通过 xhr 把整个视频给下载完成，拿到 arrayBuffer
-   调用 mediaSource.endOfStream() 通知加载完成
-   播放视频

在这几个步骤中，最耗时间的是下载视频，我们要下载整个视频，意味着如果视频很大，或者网络不好，我们无法有很长一段时间无法看到这个视频，这样的体验相当不好。

## 基于 fetch 播放视频

> [这里有一个 Fetch 下载播放视频的例子](https://fanhaoyuan.github.io/examples/stream_video.html)

我们将使用 [fatcher](https://github.com/fanhaoyuan/fatcher)来进行 fetch 请求。

跟 xhr 不一样的地方，我们在页面加载之后，立即请求下载这个视频。

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
        <!--使用 fatcher 进行 fetch 请求  -->
        <script src="https://cdn.jsdelivr.net/npm/fatcher/dist/fatcher.min.js"></script>
        <!--查看下载进度  -->
        <script src="https://cdn.jsdelivr.net/npm/@fatcherjs/middleware-progress/dist/progress.min.js"></script>
    </head>
    <body>
        <video id="video" controls />
        <script>
            (async function () {
                let { data: stream } = await Fatcher.fatcher({
                    url: `https://nickdesaulniers.github.io/netfix/demo/frag_bunny.mp4?t=${Date.now()}`,
                    middlewares: [
                        // 用于查看下载进度
                        FatcherMiddlewareProgress.progress({
                            onDownloadProgress(loaded, total) {
                                console.log(
                                    '已经加载数据大小',
                                    loaded,
                                    '总共数据大小',
                                    total,
                                    '当前进度',
                                    `${((loaded / total) * 100).toFixed(2)}%`
                                );
                            },
                        }),
                    ],
                });
            })();
        </script>
    </body>
</html>
```

我们通过请求拿到一段 ReadableStream， 这段 ReadableStream 就是 这个视频的 arrayBuffer。 但是这段 arrayBuffer 不是一次性能拿到，而是会一段一段的推送过来。

我们使用了一个 FatcherMiddleware 来查看我们 ReadableStream 的读取进度。

```js
const video = document.querySelector('#video');

const mediaSource = new MediaSource();

video.src = URL.createObjectURL(mediaSource);
```

当获取视频成功之后，我们就像上面 xhr 的例子一样，创建一个 ObjectURL 供 video 标签读取。

上面的操作看上去和 xhr 的流程一样，他有什么特别之处呢？

```js
// 转换流，可以用于转码之类的转换操作
const transformStream = new TransformStream({
    transform(chunk, controller) {
        // 可以用于转码
        controller.enqueue(chunk);
    },
});

stream = stream.pipeThrough(transformStream);
```

这两行代码是把我们拿到的 ReadableStream 交给一个 TransformStream 来进行流式转换,我们可以在转换流中进行转码等操作。

相比于 xhr 的完整下载后操作（串行），stream 可以边读取边操作（并行），大大省下处理的时间。

我们也需要监听 MediaSource 被开始读取的事件，在被读取的时候，我们要对流进行读取并加载。

```js
mediaSource.onsourceopen = async () => {
    const sourceBuffer = mediaSource.addSourceBuffer('video/mp4; codecs="avc1.42E01E, mp4a.40.2"');

    let buffers = []; // 缓冲区

    await Fatcher.readStreamByChunk(stream, chunk => {
        if (!sourceBuffer.updating) {
            // 加上缓冲区的数据长度
            const total = buffers.reduce((t, c) => c.length + t, 0) + chunk.length;

            // 新建一个 ArrayBuffer 基座
            const arrayBuffer = new ArrayBuffer(total);

            // ArrayBuffer 数据操作台
            const dataView = new DataView(arrayBuffer);

            // 合并缓存区加当前块的数据
            for (let i = 0, offset = 0, chunks = [...buffers, chunk]; i < chunks.length; i++) {
                for (let j = 0; j < chunks[i].length; j++) {
                    dataView.setUint8(offset, chunks[i][j]);
                    offset++;
                }
            }

            // 加载视频流
            sourceBuffer.appendBuffer(dataView.buffer);

            // 清空缓冲区
            buffers = [];

            return;
        }

        // 正在加载别的流，加入缓冲区，等下一次一起加载
        buffers.push(chunk);
    });
};
```

上面的例子，通过不断的获取 ReadableStream 中的数据，一块一块的供 MediaSource 读取。

因为 SourceBuffer 在 appendBuffer 的时候，并不是立即完成，如果我们对一个进行加载中的 SourceBuffer 调用 appendBuffer，会抛出错误。所以我们要设立缓冲区，只有在空闲的时候才进行加载。

```js
// 加载完成 告诉 mediaSource 已经加载完成了。
sourceBuffer.onupdateend = () => {
    console.log('视频加载完成');

    mediaSource.endOfStream();
};
```

视频加载完成后，调用 mediaSource.endOfStream 广播视频完成加载的事件。

完整例子如下：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
        <script src="https://cdn.jsdelivr.net/npm/fatcher/dist/fatcher.min.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/@fatcherjs/middleware-progress/dist/progress.min.js"></script>
    </head>
    <body>
        <video id="video" controls></video>
        <script>
            (async function () {
                let { data: stream } = await Fatcher.fatcher({
                    url: `https://nickdesaulniers.github.io/netfix/demo/frag_bunny.mp4?t=${Date.now()}`,
                    middlewares: [
                        // 用于查看下载进度
                        FatcherMiddlewareProgress.progress({
                            onDownloadProgress(loaded, total) {
                                console.log(
                                    '已经加载数据大小',
                                    loaded,
                                    '总共数据大小',
                                    total,
                                    '当前进度',
                                    `${((loaded / total) * 100).toFixed(2)}%`
                                );
                            },
                        }),
                    ],
                });

                const video = document.querySelector('#video');

                const mediaSource = new MediaSource();

                video.src = URL.createObjectURL(mediaSource);

                // 转换流，可以用于转码之类的转换操作
                const transformStream = new TransformStream({
                    transform(chunk, controller) {
                        // 可以用于转码
                        controller.enqueue(chunk);
                    },
                });

                stream = stream.pipeThrough(transformStream);

                mediaSource.onsourceopen = async () => {
                    const sourceBuffer = mediaSource.addSourceBuffer('video/mp4; codecs="avc1.42E01E, mp4a.40.2"');

                    let buffers = [];

                    await Fatcher.readStreamByChunk(stream, chunk => {
                        if (!sourceBuffer.updating) {
                            // 加上缓冲区的数据长度
                            const total = buffers.reduce((t, c) => c.length + t, 0) + chunk.length;

                            // 新建一个 ArrayBuffer 基座
                            const arrayBuffer = new ArrayBuffer(total);

                            // ArrayBuffer 数据操作台
                            const dataView = new DataView(arrayBuffer);

                            // 合并缓存区加当前块的数据
                            for (let i = 0, offset = 0, chunks = [...buffers, chunk]; i < chunks.length; i++) {
                                for (let j = 0; j < chunks[i].length; j++) {
                                    dataView.setUint8(offset, chunks[i][j]);
                                    offset++;
                                }
                            }

                            // 加载视频流
                            sourceBuffer.appendBuffer(dataView.buffer);

                            // 清空缓冲区
                            buffers = [];

                            return;
                        }

                        // 正在加载别的流，加入缓冲区，等下一次一起加载
                        buffers.push(chunk);
                    });

                    console.log('视频下载完成');

                    // 加载完成 告诉 mediaSource 已经加载完成了。
                    sourceBuffer.onupdateend = () => {
                        console.log('视频加载完成');

                        mediaSource.endOfStream();
                    };
                };
            })();
        </script>
    </body>
</html>
```

从例子的代码里可以看到，我们是有几个步骤

-   获取视频，得到 ReadableStream
-   创建 MediaSource 实例
-   创建 ObjectURL 来加载 MediaSource
-   并行加载 arrayBuffer
-   播放视频

因为我们是按块进行加载 arrayBuffer， 所以我们可以边观看视频边下载视频，相对比 xhr 的完整视频下载，体检大大提升。

## 总结

基于 xhr 的方式观看视频，需要完整下载视频文件后进行操作后才可以观看，属于串行操作。

基于 Streams 的方式观看视频，只需要获取到 ReadableStream 后就可以开始观看视频，属于并行操作。

从体验上来说，Streams 方式会优于 xhr 的方式，而且 Streams 可以边下载边操作，减少了等待的时间。

虽然 Streams 支持的比较晚，但是相信未来是 Streams 的天下。
