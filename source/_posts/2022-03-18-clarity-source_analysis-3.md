---
title: clarity-js源码分析系列（三）之元素变化
date: 2022-03-18 15:47:06
toc: true
categories:
- JS相关
tags:
- 源码分析
- 前端监控
- 录制回放
---

页面监控需要尽可能多地收集数据，这样才有可能还原出当时用户操作的真实场景。

还原用户场景常见的有几种方法：逐帧截屏、记录DOM元素及其变化、录制视频。

目前主流的用户行为监控方案都是用的记录DOM元素变化，主要优点是对用户基本上无感知，并且生成的数据量相比另外几种偏小，对性能的影响也不大。本篇我们就来深入分析下clarity在这块做了什么。

<!--more-->

# 遍历元素
在初始化的时候，会对所有元素进行遍历，绑定监听事件。浏览器支持一种方法：`MutationObserver`，相当于以观察者的角色对元素的属性、字符、子元素变化进行监听。文档可以参考[这里](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)。

首先，在遍历之前，会先对css相关的元素进行一个hook，比如`<style>`和`<link>`。因为当使用`CSSStyleSheet.insertRule`来添加css时不会触发DOM变化，所以需要在调用`insertRule`API的时候把node元素添加到延时队列里面去，以便之后统一监听。

接下来，从document开始遍历，这里会把任务都塞到一个task队列里。这个队列的作用是异步执行任务，并且按照优先级高低来排序，这里mutation的任务是最高级。

遍历DOM树，对每个元素进行处理。这里要注意的是，只有`document`、`shadowRoot`、`iframe`才用MutationObserver去观察，其他元素只更新DOM树信息。

因为`document`、`shadowRoot`、`iframe`这三种类型的元素每个都相当于一个独立的document隔离环境，需要分别去监听，iframe只支持同源。

# 观察元素
那么对于每个元素是怎么观察的呢？这里同样也是把观察元素的函数加到任务队列里执行。

首先我们知道，`observe`执行函数会返回一个`MutationRecord[]`数组，对于每个`mutation`，会有三种变化类型：`Atrributes`、`CharacterData`、`ChildList`，分别代表着属性、字符数据、子元素的变化。对于这三种变化，分别再去递归遍历更新DOM树信息。

这里还有一个特殊处理，对于每个`mutation`，用`parent.selector`+`target.selector`+`target.attrributeName`+`target.addedNodes`+`target.removeNodes`来作为唯一的`Key`。clarity对于每个正在交互操作的元素会更新一个激活态，表示当前是否有交互操作。对于当前mutation如果没有激活态，就先不进行处理，除非积累到一定程度（同一个mutation执行10次）后再去移除这个mutation需要干掉的元素。

我想这样做的目的是因为一般我们更关注在进行交互的时候使DOM发生的变化，如果没有交互的话，我们没必要重复去更新同一个mutation，而是等它积攒到一定次数后统一进行一次的变化处理，节省一些函数操作。

# 更新元素
对于每个元素，我们会去进行一个id查找，只会在原有的DOM树基础上进行更新或者添加删除。等到所有元素的变化都更新记录完后，会把该次更新所有的相关信息push到数据队列里，然后在upload里统一上报。

# 总结
利用了`MutationObserver`，我们可以很方便的对DOM元素的变化进行更新，为了减少冗余更新信息的记录，我们会对所有交互行为记录一个激活态，非激活态的更新元素我们不会更新的那么频繁，每次`observe`观察到的mutation全部更新完后，我们再把数据存入队列里，统一上报，这样就达到对DOM元素变化更新记录的效果。
