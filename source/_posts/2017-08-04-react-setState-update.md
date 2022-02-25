---
title: React中如何更新state
date: 2017-08-04 20:49:46
categories:
- JS相关
tags:
- React
- 前端框架
- state
---

我们知道，React中最重要的概念就是state和props，我们在写每一个组件时，都可以为该组件创建一个state，用以保存当前组件的数据。那么如何去更新state就是一个重要的问题了。

<!--more-->
# setState
首先我们来看一个React Component初始化的例子：
```javascript
import React, { Component } from 'react';

class newComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      name: 'flyrk',
      sex: 'male'
    };
  }

  render() {
    return (
      //....
    );
  }
}
```

而当我们要更新state，不能直接赋值修改this.state，而是应该使用this.setState()方法。

但在最近写项目的过程中，我遇到了一个问题，我想实现从服务器异步获取数据，然后在React对收到的数据进行更新state，更新的state再进行处理，同步显示在某个div中。当我这样写的时候，发现视图层迟迟没有更新：

```javascript
class newComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      name: ''
    };
  }

  componentWillMount() {
    this.props.getName().then(res => {  // 从服务器获取数据
      const name = res.data.name;
      this.setState({ name });
      //...
    });
  }
  render() {
    return (
      <div>
        {this.state.name}
      </div>
    );
  }
}
```

后来上网查阅官方文档，才发现React的setState()可能是异步的。也就是说，当我们用this.setState()更新state时，state可能不会立即更新，React会将多个setState()合并在一起最后调用来提高性能，如果我们想立即使用更新后的state，有两种方法：

setState()接受函数参数，我们可以这样写：

```javascript
this.setState((prevState, props) => ({
  name: prevState.name + props.name
}));
```

这样我们就可以获取之前的state。

还有一种方法，setState()第二个参数接受一个callback：

```javascript
this.setState({ name }, callback);
```

在callback里我们可以对修改后的state进行操作。

这两种方法，都可以保证操作的state是最新值，并且能在render()里展现出来。

