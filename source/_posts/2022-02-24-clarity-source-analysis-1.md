---
title: clarity-js源码分析系列（一）
date: 2022-02-24 15:20:47
toc: true
categories:
- JS相关
tags:
- 源码分析
- 前端监控
- 录制回放
---

前端监控一直是前端不可或缺的一部分，这里我调研了微软的[clarity](https://github.com/microsoft/clarity)，它们主要是针对用户的行为监控进行录制回放，并且能生成热力图分析。为了彻底搞清楚其中的原理，对`clarity-js`进行了源码分析。

话不多说，直接开始！
<!--more-->
# 整体代码结构
```
clarity-js
    -- src
        -- core
        -- data
        -- diagnostic
        -- interaction
        -- layout
        -- performance
        clarity.ts
        global.ts
        index.ts
```
下面围绕入口文件``index.ts``开始逐步分析。

# 代码分析
## index.ts
入口文件，主要导出三个对象：``export { clarity, version, helper };``。

这里我们主要关注`clarity`对象。

`clarity`关键导出四个方法：``start、pause、resume、stop``，从字面上也能猜出他们分别代表的功能：开始、暂停、继续、停止。

先来看源码：

```typescript
export function start(config: Config = null): void {
  // 先检查浏览器是否支持相关api
  // 保证不会多次执行start
  if (core.check()) {
    core.config(config);
    core.start();
    data.start();
    modules.forEach(x => measure(x.start)());
  }
}

export function pause(): void {
  if (core.active()) {
    data.event(Constant.Clarity, Constant.Pause);
    task.pause();
  }
}


export function resume(): void {
  if (core.active()) {
    task.resume();
    data.event(Constant.Clarity, Constant.Resume);
  }
}

export function stop(): void {
  if (core.active()) {
    // 以与modules初始化相反的顺序去停止
    modules.slice().reverse().forEach(x => measure(x.stop)());
    data.stop();
    core.stop();
  }
}
```
从源码很容易看出，这里主要就是针对整个监控流程的一个生命周期操作。主要是对``core、data、modules``这几个对象进行操作，其实最关键的部分就是初始化，我们来分模块看下。

## core
### core.config
说白了就是支持自定义配置项，具体配置内容先不深入讲，之后用到的时候再讨论。这里给出``Config``实例：
```typescript
export interface Config {
  projectId?: string;
  delay?: number;
  lean?: boolean;
  track?: boolean;
  content?: boolean;
  mask?: string[];
  unmask?: string[];
  regions?: Region[];
  metrics?: Metric[];
  dimensions?: Dimension[];
  cookies?: string[];
  report?: string;
  upload?: string | UploadCallback;
  fallback?: string;
  upgrade?: (key: string) => void;
}
```

### core.start
初始化操作：
```typescript
export function start(): void {
  status = true;
  time.start();    // 时间打点开始
  task.reset();    // 重置任务队列
  event.reset();   // 移除所有事件绑定
  report.reset();  // 清除缓存的上报数据
  history.start(); // 开始记录url的history state
}
```
这里其他的方法都好理解，关键来看看最后的``history.start``。

总共做了两件事：

1、绑定``window.popstate``事件
```flow
st=>start: 绑定window.popstate事件
cond=>condition: url是否发生变化？
sub1=>subroutine: 停止当前clarity实例
sub2=>subroutine: 250ms后重新开启clarity实例
e=>end: 结束
st->cond
cond(yes)->sub1->sub2->e
cond(no)->e
```

2、代理``history.pushState``和``history.replaceState``事件
```flow
st=>start: 代理pushState、replaceState事件
op1=>operation: 正常执行pushState、replaceState事件
cond1=>condition: 调用堆栈是否小于20？
cond2=>condition: url是否发生变化？
sub1=>subroutine: 停止当前clarity实例
sub2=>subroutine: 250ms后重新开启clarity实例
e=>end: 结束
st->cond1
cond1(yes)->op1->cond2
cond1(no)->e
cond2(yes)->sub1->sub2->e
cond2(no)->e
```
这样就能确保当url地址发生变化时，能及时重启clarity实例，保证跟踪到每个页面的状态。

## data

```typescript
export function start(): void {
  metric.start(); // 初始化所有与性能相关的信息
  modules.forEach(x => measure(x.start)()); // 初始化数据，并且测量耗时
}
```

这里的主要目的就是初始化所有数据，包括了一系列需要记录的信息：如页面浏览器、页面来源、userid、页面长宽、鼠标指针等等。数据非常繁多，同时也支持自定义，总之是尽可能地去收集页面数据，方便之后的日志分析。

但看到这里同时引入一个问题：这么庞大的数据是怎么保存和上传分析的呢？别急，之后会拿来专门分析。

## modules
这里加载了一些模块，然后进行初始化。模块包括：

`diagnostic, layout, interaction, performance`

那么这些模块在初始化时又做了些什么呢，来看看他们的操作。

### diagnostic
通过代码发现，主要做了两件事：

1、绑定`window.error`事件，记录一些错误堆栈和相关信息。

2、初始化历史缓存，用来之后打log

### layout
这个模块跟页面元素的变化息息相关，又细分了很多模块。

首先看源码：
```typescript
export function start(): void {
  // 这里的执行顺序非常重要
  doc.start();
  region.start();
  dom.start();
  mutation.start();
  discover.start();
  box.start();
}
```

通过源码分析，可以得到各个模块的大概作用：

**doc**：记录整个页面的最大宽度和高度。

**region**：利用了`IntersectionObserver`，来观察元素的变化，记录元素的交互状态，方便之后的数据重放与还原。

**dom**：遍历所有元素，记录需要遮罩的元素和监听记录所有元素的属性、状态、性能变化。

**mutation**：利用`MutationObserver`，监听DOM树和CSS的变化。

**discover**：记录dom和region变化函数的耗时。

**box**：利用`ResizeObserver`监听元素size的变化。

### interaction
这个模块主要是做一些跟交互有关的操作，先看代码：
```typescript
export function start(): void {
  timeline.start();
  click.start();
  clipboard.start();
  pointer.start();
  input.start();
  resize.start();
  visibility.start();
  scroll.start();
  selection.start();
  submit.start();
  unload.start();
}
```

分别做了以下事情：

**timeline**：记录跟踪click事件的时间线。

**click**：监听点击事件，记录点击元素相关信息。这里要着重看下记录了哪些信息，来看这段关键代码。
```typescript
if (x !== null && y !== null) {
  state.push({
    time: time(), event, data: {
      target: t,  // 当前元素
      x,          // pageX
      y,          // pageY
      eX,         // 点击时相对元素坐标X
      eY,         // 点击时相对元素坐标 Y
      button: evt.button,      // 点击按钮元素
      reaction: reaction(t),   // 是否是点击无交互元素，比如纯文本，或者非"input", "textarea", "radio", "button", "canvas"元素
      context: context(a),     // link标签a元素的target类型，比如：blank、parent、top
      text: text(t),           // 点击文本，截取前25个非空字符
      link: a ? a.href : null, // 跳转链接
      hash: null
    }
  });
  schedule(encode.bind(this, event));
}
```

这样一来，就能相对完整地记录点击的元素信息，方便之后还原。

**clipboard**：监听`cut、copy、paste`事件，并记录相应的event对象。

**pointer**：监听所有跟鼠标指针交互相关的事件：`mousedown、mouseup、mousemove、mousewheel、dblclick、touchstart、touchend、touchmove、touchcancel`，并记录指针位置。

**input**：监听`input`事件，包括`value、attr、placeholder`等方面的隐私处理，主要记录value。

**resize**：监听`window.resize`事件，记录window视窗变化。

**visibility**：监听`visibilitychange`事件，记录`document.visibilityState`。

**scroll**：监听元素的`scroll`事件，记录当前滚动元素和滚动位置。

**selection**：监听元素`selectstart、selectionchange`事件，记录选区起始和结束锚点和元素。

**submit**：监听元素`submit`事件，记录当前元素。

**unload**：监听`window.pagehide`事件，记录事件，停止clarity实例。



### performance
这里的模块很容易理解，就是记录页面的各种性能，主要包括以下两部分：

**navigation**：利用`PerformanceNavigationTiming`记录页面首屏性能指标，包括：DNS解析时间、请求时间、DOM解析时间、重定向时间等等。

**observer**：利用`PerformanceObserver`观测页面性能指标，包括：浏览器、资源、长任务、首次输入延迟、累积布局偏移、最大内容绘制。

# 总结
到此，我们分析了clarity的代码结构，和初始化时各个模块的分工。

下一篇，我将着重分析关键的数据存储和上报方式，并且回顾整个系统架构，整体分析clarity的设计理念。