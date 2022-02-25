---
title: JS如何设置Cookies
date: 2017-08-12 23:07:55
categories:
- 网络
tags:
- Cookies
---

Cookie一般是用来在浏览器端存储用户的一些登录、浏览数据，方便缓存。要设置Cookie，可以有两种方法。
<!--more-->
# 调用浏览器的CookiesAPI
为了使用浏览器的CookieAPI，我们首先要在[manifest.json](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/manifest.json)里设置Cookie`permission`，还要设置`host-permission`确定能获取Cookie的网址。

接下来就是调用CookieAPI方法。

## Cookie.set
使用`set()`方法添加新的Cookie，返回的是一个`Promise`对象。语法如下：
```javascript
var setting = browser.cookies.set(
  details               // object
)
```
`details`是一个对象，其中有可选的几个参数：
- `url`: 请求Cookie的url
- `name`: Cookie的名字
- `value`: Cookie的值
- `domin`: Cookie的作用域名
- `path`: Cookie的路径
- `secure`: Cookie是否安全（true／false）
- `httpOnly`: Cookie是否只能在http上（true／false）
- `expirationDate`: Cookie的截止时间，如果没有设置，则Cookie变为Session Cookie
- `storeId`: 代表Cookie的存储ID

## Cookie.get
使用`get()`方法获取已有的Cookie，返回的是一个`Promise`对象。语法如下：
```javascript
var getting = browser.cookies.get(
  details               // object
)
```
`details`可选的参数有：`url`、`name`、`storeId`。

## Cookie.getAll
使用`getAll()`方法获取Cookie集合中所有匹配details的Cookie，返回的是一个`Promise`对象。语法如下：
```javascript
var getting = browser.cookies.getAll(
  details               // object
)
```
`details`可选的参数有：`url`、`name`、`domain`、`path`、`secure`、`session`、`storeId`。其中`session`是bool值，代表是否要从cookies中过滤掉session cookie。

## Cookie.remove
使用`remove()`方法移除已有的Cookie，返回的是一个`Promise`对象。语法如下：
```javascript
var removing = browser.cookies.remove(
  details               // object
)
```
`details`可选的参数有：`url`、`name`、`storeId`。

## Cookie.getAllCookieStores
使用`getAllCookieStores()`方法获取所有的Cookie集合，返回的是一个`Promise`对象。语法如下：
```javascript
var gettingStores = browser.cookies.getAllCookieStores();
```
返回的`Promise`对象中包含的数据是包括所有cookiestore对象的数组。举个例子：
```javascript
function logStores(cookieStores) {
  for(store of cookieStores) {
    console.log(`Cookie store: ${store.id}\n Tab IDs: ${store.tabIds}`);
  }
}

var getting = browser.cookies.getAllCookieStores();
getting.then(logStores);
```

## Cookie.onChanged事件
当Cookie改变时，我们可以为它设置onChanged事件：
```javascript
browser.cookies.onChanged.addListener(callback)
browser.cookies.onChanged.removeListener(listener)
browser.cookies.onChanged.hasListener(listener)
```
其中 `addListener`接受一个callback，callback有一个参数`changeInfo`，`changeInfo`有三个属性：
  - removed：bool值，代表cookie是否移除
  - cookie：包含添加或移除信息的cookie对象
  - cause：Cookie改变的原因

---

# 使用document.cookie
Cookie的结构很简单，就是键-值对，一般是以`key-value;expiration_date;path;domain;`的顺序。

最简单的设置当前页面Cookie的方法就是直接给`document.cookie`赋值，例如：`document.cookie = "username=Simon, 12 Aug 2017 19:36:00 GMT; expires=Wed, 31 Oct 2017 11:00:00 GMT;";`，这样就设置了Cookie。但每次都这样设置很麻烦，我们可以用函数封装赋值方法。这里借鉴了网上的一些方法：

## setCookie()
```javascript
function setCookie(name, value, expires, path, domain) {
  var cookie = name + "=" + encodeURIComponent(value) + ";";

  if (expires) {
    // If it's a date
    if(expires instanceof Date) {
      // If it isn't a valid date
      if (isNaN(expires.getTime()))
       expires = new Date();
    } else {
      expires = new Date(new Date().getTime() + parseInt(expires) * 1000 * 60 * 60 * 24);
    }
    cookie += "expires=" + expires.toGMTString() + ";";
  }

  if (path) {
    cookie += "path=" + path + ";";
  }
  if (domain) {
    cookie += "domain=" + domain + ";";
  }
  document.cookie = cookie;
}
```
这里的`expires`可以是`Date`对象，也可以是代表天数的数字。

要创建新的Cookie，可以这样：`setCookie("website", "xmflyrk.com", new Date(new Date().getTime() + 10000));` 或者 `setCookie("author", "flyrk", 30);`

## getCookie()
```javascript
function getCookie(name) {
  var cok = document.cookie.split(';'),
        length = cok.length;
  for (var i = 0; i < length; i++) {
    var pairs = cok[i].trim();
    if (~pairs.indexOf(name)) {
      return pairs.substring(pairs.indexOf(name) + name.length + 1);
    }
  }
  return null;
}
```
调用`getCookie()`的例子：
```javascript
getCookie('author'); // "flyrk"
getCookie('Something'); // null
```

## removeCookie()
```javascript
function removeCookie(name, path, domin) {
  if (getCookie(name)) {
    setCookie(name, "", -1, path, domin);
  }
}
```
删除某个Cookie我们就可以这样：
```javascript
removeCookie("author");
console.log(getCookie("author")); //null
```

---

# 总结
有了这些API和自己封装的函数，我们就能对Cookie进行操作，方便地设置Cookie数据。

> 参考资料：
> [cookies](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/API/cookies)
> [How to deal with cookie](https://www.sitepoint.com/how-to-deal-with-cookies-in-javascript/)
