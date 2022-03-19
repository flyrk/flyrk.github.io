---
title: clarity-js源码分析系列（二）之数据处理
date: 2022-02-26 20:20:47
toc: true
categories:
- JS相关
tags:
- 源码分析
- 前端监控
- 录制回放
---

上一篇我们大概介绍了clarity各个模块的作用，本篇我想着重来了解下`clarity`在数据存储、日志上报、性能优化方面做的事情，并试着理清clarity整个的架构。
<!--more-->

# 数据存储
对于前端监控，特别是需要录制回放功能的时候，数据就是生命。没有了监控数据，一切都免谈。

但是，为了尽可能收集用户的行为特征，和方便之后的行为分析和日志定位，肯定需要监控许多数据，光我一下子想到的就有这些方面：DOM元素信息、鼠标交互信息、页面状态、用户停留时长等等。

面对这么多的数据，首先想到的就是怎么去存储。我以为这么多数据，clarity肯定会用indexDB去存储在本地，但是通过源码发现，并没有。那么clarity是怎么做的呢？

## 队列缓存
clarity直接用js数组来缓存数据对象，每当有新的数据加入，通过`encode`函数统一分发处理，然后塞到数组队列里去：
```typescript
export function queue(tokens: Token[], transmit: boolean = true): void {
  if (active) {
    let now = time();
    let type = tokens.length > 1 ? tokens[1] : null;
    let event = JSON.stringify(tokens);

    switch (type) {
      case Event.Discover:
        discoverBytes += event.length;
      case Event.Box:
      case Event.Mutation:
        playbackBytes += event.length;
        playback.push(event);
        break;
      default:
        analysis.push(event);
        break;
    }

    metric.count(Metric.EventCount);

    let gap = delay();
    if (now - queuedTime > (gap * 2)) {
      clearTimeout(timeout);
      timeout = null;
    }

    if (transmit && timeout === null) {
      if (type !== Event.Ping) { ping.reset(); }
      timeout = setTimeout(upload, gap);
      queuedTime = now;
      limit.check(playbackBytes);
    }
  }
}
```

根据这段代码，我们可以知道`queue`主要做了两件事：

1、根据数据类型分类塞到不同的队列，供之后上传使用。

2、部分数据利用`setTimeout`延时上报。

注意的是，大部分数据在延时100～3000ms后都会立即上报，也是为了及时释放数据，不至于堆积得太多。除了某些场景，比如`metric`模块相关的性能指标，需要一直关注，会在最后结束的时候统一再上传。

## 数据检测
这里还进行了数据的检测：`limit.check(playbackBytes)`。主要是进行三种检测：

1、payload不能超过128个，也就是`envelope`对象的个数。

2、页面实例开启不能超过2个小时，也就是说最多能监测在页面停留两个小时的数据。

3、数据大小不能超过10MB。

如果不满足其中任何一种限制，则立即停止clarity实例。

经过这些限制，有效地保证了数据的大小，不至于内存过高，导致页面崩溃或者是上传过多不必要的数据。

# 日志上报
## 上报时机
日志上报的时机有两个：

1、在数据刚进入队列时，延时几百毫秒立即上报。

2、在实例`stop`的时候上报，比如`pagehide`。

## 数据压缩
上报的操作其实就是把当前收集到的所有数据集中起来，打包成一个`payload`。由于数据会很大，需要对`payload`进行压缩操作：
```typescript
export default async function(input: string): Promise<Uint8Array> {
  try {
    if (supported) {
      const stream = new ReadableStream({async start(controller) {
        controller.enqueue(input);
        controller.close();
      }}).pipeThrough(new TextEncoderStream()).pipeThrough(new window[Constant.CompressionStream]("gzip"));
      return new Uint8Array(await read(stream));
    }
  } catch { /* do nothing */ }
  return null;
}
```

通过源码得知这里的压缩原理：创建`ReadableStream`管道，将数据字符串塞进去，通过`TextEncoderStream`和`CompressionStream`对其进行`gzip`压缩。

## 上报方式
数据压缩完毕，下一步就是发起上报请求。常用的前端监控上报可以通过三种方式上报：

1、`(new Image()).src = ${uploadUrl}`，这里一般适合数据小，并且需要跨域的请求。

2、`ajax`请求，这个也是最常见的上报方式。

3、`sendBeacon`，一般用在`unload`或者页面关闭的时候，用于发送统计数据。因为在页面关闭时，正常的`ajax`异步请求经常会出现丢失情况，而同步请求又会阻塞页面的关闭跳转，有了`sendBeacon`，会使用户代理在有机会时异步地向服务器发送数据，同时不会延迟页面的卸载或影响下一导航的载入性能。不过要要注意的是，这里使用的是`POST`请求，而且无法设置`HTTP headers`，所以不能用压缩的数据，只能用字符串。

这里`clarity`使用了三种策略：

1、正常上报使用`ajax`请求，并且优先上报压缩过的数据（考虑到CompressionStream的浏览器兼容性，如果不支持则用原始字符串数据）。

2、如果是在页面关闭或者停止`clarity`实例时，优先用`sendBeacon`，如果不支持则用回`ajax`。

3、支持配置`upload`参数，利用用户自定义的`upload`函数来上传。

当然，这里的上报还做了保护机制，支持上报失败自动重试，并且在首次上报后还会更新`session`，保持会话状态。

# 性能优化
对于一个前端监控SDK来说，性能非常重要。因为我们的目的是监控页面，但是不能影响到用户的正常操作，而是以一个观察者的角色在旁边看着。如果监控代码影响到页面性能，甚至引发页面卡顿、或者出bug，阻塞用户正常页面操作，那就得不偿失了。

为了检测clarity的性能，我在我的博客加上了`claritySDK`，看看性能表现如何。

通过几天的观察，目前性能还算良好，每次交互完都会立即上报。不过因为博客页面比较简单，没有什么复杂的交互，主要是点击和滚动页面，每次上报的数据一般都是几十B，内存损耗比较小。

但同样的，通过Chrome本身的性能指标监控，发现每次交互完JS的堆内存都会暴涨一波，等上报完内存就会释放。虽然说内存及时释放了，但是如果是更复杂的页面，需要大量的交互操作，比如编辑器，那内存可能会居高不下，所以这方面的性能还有待提高。

从SDK代码本身来说，没有什么高耗时的函数运算，其风险主要还是在内存爆栈上。

# 总结
`clarity`总体上还是从以下几个方面去做监控：

页面信息采集 -> 事件监听 -> 数据分类 -> 批量上报。

其中我认为做的比较好的部分是页面信息采集和数据分类，尽可能多的去收集页面信息，并且记录元素对象和id，同时采用了一些数据压缩方式，来保证数据的完整性和可用性。

但同时，对于DOM结构复杂或者交互复杂的页面，`clarity`会占用大量的内存，由于是采用纯数组和对象存储，导致JS堆栈在某些时刻会飙升，非常有可能影响到页面性能，这方面的性能损耗还需进一步评测。

总的来说，在数据采集端`clarity`对数据的收集还是比较完备，不足的是性能优化方面，但对于大部分普通HTML页面，没有多少交互来说也能够支持。而`clarity`的模块分类和数据收集方式也是值得我们好好学习的。
