---
title: 点击按钮粘贴所选内容到剪贴板
date: 2018-12-04 21:13:09
categories:
- JS相关
tags:
- JS技巧
---

我们经常可以看到这样的功能，点击某个按钮可以自动复制想要的内容到剪贴板。比如说某个图片、视频的地址，某个输入的内容，我们有的时候懒得去用键盘鼠标选中某个区域，希望只用点击一下，就能复制想要的内容。
<!--more -->
# 代码实现
其实这个用原生JS实现难度不大，主要用到的是`document.execCommand()`。代码如下：

```javascript
function copy(e) {
    let transfer = document.createElement('input');
    document.body.appendChild(transfer);
    transfer.value = target.value;  // 这里表示想要复制的内容
    transfer.focus();
    transfer.select();
    if (document.execCommand('copy')) {
        document.execCommand('copy');
    }
    transfer.blur();
    console.log('复制成功');
    document.body.removeChild(transfer);
    
}

$('#copyBtn').addEventListener('click', copy);
```

这里我们其实就是新创建了一个`input`DOM元素，然后选中该元素，把要复制的内容赋给`input.value`，这个时候执行`document.execCommand('copy')`，会把当前页面所有选中的内容复制到剪贴板，也就实现了复制的操作，最后再把新建的DOM元素移除，在不影响DOM树的情况下达到复制的目的。

# 总结
```document.execCommand()``` 方法可以使当前选中的可编辑内容实现一些常用的操作，如copy、cut、paste、delete、contentReadOnly等等，具体请看[document.execCommand](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand)，但个人感觉一般最好不要去用里面的一些操作，因为这样可能会和用户的交互操作产生冲突，发生意想不到的结果。但是`document.execCommand`在富文本编辑器里又是个神器，值得之后慢慢研究。