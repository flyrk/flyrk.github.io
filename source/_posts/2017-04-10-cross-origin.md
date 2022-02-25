---
title: 浏览器跨域问题
date: 2017-04-10 23:05:26
categories:
- Web相关
tags:
- 跨域
---

最近经常看到有关浏览器跨域的问题，查了很多资料，所以在这总结一下。
由于浏览器的[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)，当我们想要进行一个ajax请求或者获取其他域名的数据时就会出现跨域问题。避免跨域问题可以有以下几种方法：
<!--more-->
## document.domain
我们可以在要传输数据的两个页面设置相同的document.domain="www.example.com"，使其主域相同。但这只有当两个页面的主域相同时才能设置。比如"www.a.com"和"bt.a.com"，但"a.com"和"b.com"就不行。

## window.name
因为浏览器窗口共用一个window.name，利用这种hack设置window.name为传输的数据，可以在同一个窗口页面内进行数据传输。但缺陷是数据量不会太大，并且只能传输字符串。

## CORS
HTML5新增的CORS方案。简单来说就是在服务器端设置Access-Control-Allow-Origin：*则代表允许任何域连接，也可以设置为想要连接的域地址。

## JSONP
利用script标签的src获取url可以实现跨域，通过在url中设置callback参数，url返回的是json数据，然后在客户端设置回调函数处理数据，就可以实现跨域。但缺点是只能用于get方法。

## postMessage方法
HTML5新增window.postMessage(message, targetOrigin, [transfer)方法，message代表传输的数据，targetOrigin代表接受的目标域，transfer是可选变量，代表跟随message一起传输的数据，且传输过去后在发送端不再可用。
接受端通过监听message事件，调用event.data来获取数据。
