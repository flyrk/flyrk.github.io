---
title: 实现基于codemirror的markdown编辑器(一)
date: 2018-12-15 22:23:57
toc: true
categories:
- JS相关
tags:
- 编辑器
- markdown
---

我们知道，要实现一个Markdown编辑器，一般会想要实现这样的功能：实时预览、同步滚动、支持标签按钮添加、markdown语法高亮等等。那么如何去实现一个功能比较完整的markdown编辑器呢？

首先这里我们只讨论web端，其实现在市面上的Markdown编辑器已经很优秀了，有道笔记、印象笔记、Cmd等等，很多都可以拿来直接用，满足基本需求足够了。但是作为前端程序员来说，当然想要实现一个自己定制化的markdown编辑器比较有意思，不过通过查阅资料文档和各种框架比较，发现也不是那么容易的事。
<!--more-->
# 框架选择
要实现一个markdown编辑器，首先得找到一个合适的框架，目前市面上优秀的markdown框架五花八门。由于现在自己写的项目一般都使用React，所以理所当然能找到一个比较契合React的Markdown框架是更好的。

经过前期调研，发现有几款基于React的Markdown框架还不错，[react-quill](https://zenoamaro.github.io/react-quill/)、facebook的[draft](https://draftjs.org/docs/getting-started.html)、[react-codemirror2](https://github.com/scniro/react-codemirror2)，但它们其实都是富文本编辑器，定制的东西已经很完善了，我想要的框架是可支持自定义扩展，定制化自己想要的功能，找来找去，最后决定基于[CodeMirror](https://codemirror.net/doc/manual.html#api)来自己实现一个定制化的markdown编辑器。因为CodeMirror其实相当于一个基础的框架，它有很多可配置项，可以自己添加工具栏，添加各种实用功能，这正好满足了我的需求，于是就决定用它了！

# CodeMirror
现代很多编辑器其实都基于`codemirror`，为什么要选择它呢？这都得益于它强大的API和配置项，在不失基本功能的同时给予了开发者极大的定制空间，开发者可以基于它开发出各种功能的markdown编辑器。这里我先简单介绍一下codemirror的一些基本操作。想要了解更多有关codemirror，可以去看[官方文档](https://codemirror.net/doc/manual.html)。

## 如何加载codemirror
一般我们在项目里npm下载完codemirror包后，在代码里引入`codemirror.js`，而且还会根据需要引入想要的语言模式包：
```js
import * as CM from 'codemirror';

import 'codemirror/mode/xml/xml';
import 'codemirror/mode/markdown/markdown';
import 'codemirror/addon/edit/continuelist';
```

当然，我们还得在html里引入`codemirror.css`样式，也可以自己修改成想要的样式。

然后，在需要加载的地方写html标签，id为`codemirror`，这里我们用的是JSX：
```jsx
<div className="editor-root" ref={(elem) => { this.editRoot = elem; }}>
  <textarea id="codemirror" name={this.props.path} autoComplete="off" />
</div>
```
我在`textarea`外面用div包了一层，是为了方便之后的滚动效果。这样，我们就已经可以使用codemirror写一个markdown编辑器了。

## options 可以使用的参数
  CodeMirror函数和它的fromTextArea方法都可以使用一个配置对象作为第二个参数。这个options极为重要，基本就决定了你的编辑器主要的输入语言、样式、内容。这里列一下它几个常用的可选的参数：
- value: string | CodeMirror.Doc
  编辑器的初始值（文本），可以是字符串或者CodeMirror文档对象(不同于HTML文档对象)。
- mode: string | object
  通用的或者在CodeMirror中使用的与mode相关联的mime，当不设置这个值的时候，会默认使用第一个载入的mode定义文件。一般地，会使用关联的mime类型来设置这个值；除此之外，也可以使用一个带有name属性的对象来作为值（如：{name: “javascript”, json: true}）。可以通过访问CodeMirror.modes和CodeMirror.mimeModes获取定义的mode和MIME。
- lineSeparator: string|null
  明确指定编辑器使用的行分割符（换行符）。默认（值为null）情况下，文档会被 CRLF(以及单独的CR, LF)分割，单独的LF会在所有的输出中用作换行符（如：getValue）。当指定了换行字符串，行就只会被指定的串分割。
- theme: string
  配置编辑器的主题样式。要使用主题，必须保证名称为 .cm-s-[name] (name是设置的theme的值)的样式是加载上了的。当然，你也可以一次加载多个主题样式，使用方法和html和使用类一样，如： theme: foo bar，那么此时需要cm-s-foo cm-s-bar这两个样式都已经被加载上了。
- indentUnit: integer
  缩进单位，值为空格数，默认为2 。
- smartIndent: boolean
  自动缩进，设置是否根据上下文自动缩进（和上一行相同的缩进量）。默认为true。
- tabSize: integer
  tab字符的宽度，默认为4 。
- indentWithTabs: boolean
  在缩进时，是否需要把 n*tab宽度个空格替换成n个tab字符，默认为false 。
- electricChars: boolean
  在输入可能改变当前的缩进时，是否重新缩进，默认为true （仅在mode支持缩进时有效）。
- keyMap: string
  配置快捷键。默认值为default，即 codemorrir.js 内部定义。其它在key map目录下。
- extraKeys: object
  给编辑器绑定与前面keyMap配置不同的快捷键。
- lineWrapping: boolean
  在长行时文字是换行(wrap)还是滚动(scroll)，默认为滚动(scroll)。
- lineNumbers: boolean
  是否在编辑器左侧显示行号。
- firstLineNumber: integer
  行号从哪个数开始计数，默认为1 。
- lineNumberFormatter: function(line: integer) → string
  使用一个函数设置行号。
- scrollbarStyle: string
  设置滚动条。默认为”native”，显示原生的滚动条。核心库还提供了”null”样式，此样式会完全隐藏滚动条。Addons可以设置更多的滚动条模式。
- inputStyle: string
  选择CodeMirror处理输入和焦点的方式。核心库定义了textarea和contenteditable输入模式。在移动浏览器上，默认是contenteditable，在桌面浏览器上，默认是textarea。在contenteditable模式下对IME和屏幕阅读器支持更好。
- readOnly: boolean|string
  编辑器是否只读。如果设置为预设的值 “nocursor”，那么除了设置只读外，编辑区域还不能获得焦点。
- showCursorWhenSelecting: boolean
  在选择时是否显示光标，默认为false。
- autofocus: boolean
  是否在初始化时自动获取焦点。默认情况是关闭的。但是，在使用textarea并且没有明确指定值的时候会被自动设置为true。

更多的配置请查看[相关文档](https://codemirror.net/doc/manual.html#config)。

# 实时预览
这个需求可以说是现代markdown编辑器的基本需求，因为markdown写起来本身是没有任何样式的，纯文本的话对于有些人来说不能看到最终效果就很不习惯，自己写的效果怎么样都不知道，写的心里就会很没底（当然再花哨的样式最后还是要看内容好不好）。说白了，实时预览更多是写作时一个心理上的满足，这对于写作体验的提升来说是必不可少的。

这里的markdown渲染引擎我直接用了比较受欢迎的[`marked`](https://marked.js.org/#/README.md#README.md)库，其实markdown渲染是很考验技术的一门活，绝大部分都是用正则匹配来进行标签的替换，但由于markdown的语法相对还是较多的，要自己写一个markdown渲染引擎还是有点难度的，考虑到时间因素所以一般都会用现成的库，当然以后有时间的话我也会考虑写一个markdown渲染引擎玩玩。

这里我因为是用`React`写的markdown组件，下面我就简单的介绍下如何实现实时预览。

首先在组件渲染完后，我们在`componentDidMount`函数里对codemirror进行事件监听：
```js
componentDidMount () {
  // 因为需要使用fromTextArea获取options，所以采用document.getElementById
  this.codeMirror = CM.fromTextArea(ReactDOM.findDOMNode(document.getElementById('codemirror')), this.getOptions());
  this.props.init(this.editRoot, this.codeMirror);
  this.codeMirror.setValue(this.props.defaultValue);
  this.codeMirror.on('change', this.codemirrorValueChanged);
  this._currentCodemirrorValue = this.props.defaultValue;
}
```
这里我们使用`fromTextArea`获取codemirror的dom节点，它支持两个参数，第一个是codemirror所挂载的dom节点，第二个是配置的options。

之后调用init函数，这里之后会讲。然后就是初始值设置`codemirror.setValue(defaultValue)`，因为是编辑器，如果想对之前的文档进行修改，肯定需要传入初始值。

然后就是关键的监听事件`change`，当输入变化时调用`this.codemirrorValueChanged`事件：
```js
codemirrorValueChanged = (cm) => {
  const newValue = cm.getValue();
  this._currentCodemirrorValue = cm.getValue();
  this.props.onChange && this.props.onChange(newValue);
}
```
这里说下，我们通过`cm.getValue()`获取当前输入的值，然后通过调用`props.onChange(newValue)`将值抛给父组件处理。因为这里我想把编辑部分和预览部分分开来做成两个公共组件，这样组件复用性更强，有的可能只需要编辑功能，有的只需要预览展示功能。
那么在父组件里，我们需要对`onChange`事件传入的值进行处理，将其渲染成html然后展示出来，其实很简单：
```jsx
class MarkdownEditor extends Component {
  //...

  handleEditChange = (newCode) => {
    this.props.renderMarkdown && this.props.renderMarkdown(newCode);
    this.setState({
      code: newCode,
      hasChanged: true
    });
  }
  debounceEditChange = debounce(this.handleEditChange, 500);
  // ...
  render() {
    const preview = marked(this.state.code);
    return (
      <div>
        <MyEditor
          defaultValue={this.state.code}
          onChange={this.debounceEditChange}
        />
        <div className="md-preview-container">
          <div className="preview-content" dangerouslySetInnerHTML={{__html: preview}}/>
        </div>
      </div>
    );
  }
}
```
这里要注意的是，我用到了`debounce`函数进行延迟处理：
```js
function debounce (fn, delay = 20) {
  let timeout;

  return function (...args) {
    const self = this;
    if (timeout) {
      clearTimeout(timeout);
    }
    timeout = setTimeout(() => {
      fn.call(self, ...args);
    }, delay);
  }
}
```
因为考虑到性能原因，如果输入过快的话，`change`事件频繁触发，预览页面不断重绘，性能损耗十分严重，所以我用到了`debounce`，并设置延迟时间500毫秒，当然可以更短，这里我觉得设置500毫秒已经足够了。这样就达到了实时预览的效果。

# 总结
基于codemirror的markdown编辑器实现起来其实还有很多坑，本篇先介绍了`codemirror`的一些基本配置和如何实现markdown实时预览，接下来我将进一步介绍如何实现同屏滚动和自定义标签按钮等其他功能。
