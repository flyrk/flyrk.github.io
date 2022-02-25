---
title: 如何实现文字输入效果
date: 2019-01-23 19:01:58
categories:
- CSS相关
tags:
- CSS技巧
---

我们看到有些网站的文字会有类似于输入的效果，就好像自动打字一样，虽然这个效果用的地方可能不多，但如果用的好的话可能会有奇效，比如在博客、演讲展示等方面应用。那么，怎么实现呢？

<!--more-->

## 使用setTimeout实现

第一种方法是使用setTimeout进行文字控制，每一个文字都是一个`span`，利用DOM操作以一定的间隔将每个文字添加到文档中，同时设置`opacity:1`使其显示出来。光标闪烁的效果则利用`border-left`进行animation动画展示。

### HTML结构

先看代码：

```html
<div class="container">
  <article class="input-container">
      <!--这里是需要展示的原始文字，设为display:none;-->
    <p class="originWords">Hello world!</p>
  </article>
  <article class="typewriter-container">
      <!--这里是最终有文字输入效果的文字-->
    <p class="typewriter-output"></p>
  </article>
</div>
```

解释一下，我们首先把原始文字设为`display: none;`，这样不会占用文档位置，然后我们给之后添加的文字`span`块指定class为`word`。

### CSS设置

先看代码：

```css
.container {
  width: 100%;
  display: flex;
  justify-content: center;
}

.input-container {
  display: none;
}
.typewriter-container {
  width: 50%;
}
.word {
  opacity: 0;
  transition: opacity 2s step-start;
}
.typewriter-output:after {
  display: inline-block;
  content: '';
  vertical-align: -1px;
  height: 14px;
  width: 1px;
  border-left: 2px solid black;
  margin-left: 2px;
  animation: shine 1s infinite;
}
@keyframes shine {
  50% {
    border-left: 0;
  }
}
```

我们对`word`设置`transition: opacity 2s step-start;`。这里之所以通过`opacity`来进行transition，是因为transition不支持`display`，而`opacity`只需要设置`0`或`1`就可以使元素进行显示而不需要重排。

这里`transition-timing-function`使用`step-start`，使其有跳跃的效果，`step-start`相当于`steps(1, jump-start)`，`steps(n, <jumpterm>)`代表transition会停顿n次，每一次的效果为`<jumpterm>`，`jump-start`代表第一个跳跃发生在`transition`刚开始的时候，更多效果见[文档](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function)。

为什么要使用`:after`呢？因为我是逐步向`typewriter-output`添加`span`块，所以一开始宽度是不确定的，而且光标高度也不好设置，所以我想了个笨方法，使用`after`伪元素生成一个光标，并用`animation`控制其`border-left`的显示。但这里有个bug就是文字显示的间隔不能超过300ms，所以说这样处理还是有些问题，以后可能会有更好的方法。

### JS部分

其实就是利用`setTimout`设置时间间隔，通过DOM操作进行添加文字，先看代码：

```javascript
var words = document.querySelector('.originWords').innerText;
var output = document.querySelector('.typewriter-output');
var word = null;
var lastWord = null;

for (var i = 0, l = words.length; i< l; i++) {
  setTimeout(writeWord(i), i * 200);	// 每隔0.2s输出一个文字
}
// 这里使用闭包保存i
function writeWord(index) {
  return function () {
    word = document.createElement('span');
    word.classList.add('word');
    output.appendChild(word);
    if (!lastWord) {
      lastWord = word;
    }

    if (lastWord !== word) {
     lastWord.style.opacity = '1'; 
    }
    word.innerText = words[index];
    lastWord = word;
    if (index === words.length - 1) {
      lastWord.style.opacity = '1';
    }
  }
}
```

代码部分很简单，就是先获取需要展示的文字内容，然后每个字都通过dom操作塞进去，设置`opacity:1`，再进行一下边界判断，就实现了文字的展示。

---

## 纯animation实现

这次我们不用`setTimeout`，所有展示的动画全部用`animation`来实现。废话不多说，我们来看一下。

### HTML结构

```html
<h1 id="typewriter">Hello, My friend.</h1>
```

这种方法的好处就是不用有一个隐藏的原始文字段落，直接显示想显示的内容。

### CSS设置

接下来我们设置CSS：

```css
body {
    background: black;
    color: #fff;
}

h1 {
    font: bold 100% monospace;
    border-right: 0.1em solid;
    margin: 2em 1em;
    white-space: nowrap;
    overflow: hidden;
}

@keyframes typing {
    from {
        width: 0;
    }
}

@keyframes cursor-blink {
    50% {
        border-color: transparent;
    }
}
```

看起来很简单对不对？挨？说好的动画呢？别急，我们通过JS来设置`animation`。

### JS部分

先看代码：

```javascript
var typewriter = document.getElementById('typewriter');
var words = typewriter.innerText.length;
if (words) {
    typewriter.style.width = words + 'ch';
    typewriter.style.animation = 'typing 3s steps(' + words + ', end), cursor-blink 0.5s step-end infinite alternate';
}
```

这里我们所做的事就是获取文字段落字符串的长度，然后设置其宽度和动画效果。非常简单！

那么，为什么这样做可以呢？别急，我们来分析分析。

首先，在获取了文字字符串长度后，我们设置段落的宽度为`width: Xch;`，这里X代表字符串长度，`ch`是长度单位，代表数字“0”的宽度，通常也就是一个字符的宽度。

然后，我们直接设置动画：
`animation: typing 3s steps(X, end), cursor-blink 0.5s step-end infinite alternate;`
一切就搞定了！为啥？通过CSS我们设置了`@keyframe`typing，表示宽度从0开始，一直到设置的`Xch`，持续3s，用`steps`动画函数，分为X步，也就是有多少个字符，就停顿多少下，达到输入的效果。至于光标的效果，我们直接通过动画设定`50%`的时候`border-color`为`transparent`，则每一秒闪烁一次，因为只有`border-right`设置了宽度，所以完美达到光标的效果。

## 总结

两种方法比较起来，第二种方法更加实用简便，推荐大家使用。这只是简单的CSS3 animation动画应用，今后还可以开动脑筋，实现更多炫酷有趣的效果。