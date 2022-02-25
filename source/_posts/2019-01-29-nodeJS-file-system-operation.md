---
title: NodeJS常用的一些文件操作技巧
date: 2019-01-29 18:20:04
categories:
- JS相关
tags:
- nodejs
---

在使用NodeJS的时候，我们经常会遇到对文件进行查找、删除、增加内容等操作，还好NodeJS内置了`fs`模块，`fs`的API提供了一系列方法，通过它我们可以实现基本的文件操作，接下来我们就来见识见识。
<!--more-->

> 说明:
> 本文全部使用ES6语法
> 提到的每个方法都有异步和同步的区别，同步则在方法后面加上`Sync`

# 文件夹操作
## 读取文件夹内容
很多时候我们得到一个文件夹路径，如果需要获取文件夹里有哪些文件，我们可以利用`fs.readdir()`，例如：
```javascript
const fs = require('fs');
const dest = '.../work/myFiles';

// 异步读取文件夹内容
fs.readdir(dest, (err, files) => {  // callback函数
  if (!err) {
    files.forEach(file => console.log(file));  // 返回的files是一个包含目录中所有文件名的数组
  } else {
    throw err;
  }
});

// 同步读取文件夹内容
try {
  const files = fs.readdirSync(dest);
  files.forEach(file => console.log(file));
} catch (err) {
  throw err;
}
```

## 创建文件夹
创建文件夹利用的API是`fs.mkdir()`，这个操作我们其实很熟悉了，在bash里我们经常用`mkdir filedirectionName`这一操作来新建文件夹，这里也是一样的，但有几个方面要注意：
```javascript
// 异步创建文件夹
fs.mkdir(dest, { recursive: true }, (err) => {
  if (err) {
    throw err;
  }
});

// 同步创建文件夹
fs.mkdirSync(dest, { recursive: true });
```

这里我们看到`{ recursive: true }`这个参数，设置为true代表应该创建父文件夹。比如我们设置`recursive`为true后，要创建`/tmp/a/app`目录，则无论是否存在`/tmp/a`目录都会新建，如果不设置`recursive`，则没有父文件夹的情况下新建会不成功。

`fs.mkdirSync()`返回`undefined`。

## 确认某个文件夹是否存在
之前我们在创建文件夹的时候可以通过设置`recursive`为true来创建父文件夹不存在的情况下的文件夹，但是有的时候我们得到一个文件夹路径，我们并不知道它的父文件夹是否存在，甚至不知道它父文件夹的父文件夹是否存在...所以，我们不想关心那么多，我们就想知道这个文件夹是否存在，怎么办呢？

这里我用到了`fs-extra`模块，它是在`fs`的基础上进行了一些扩展，有兴趣的可以去[Github](https://github.com/jprichardson/node-fs-extra)查看。其中有一个API：`ensurDir()`，它可以用来查找文件夹是否存在，如果不存在，则新建该文件夹，且会自动把父文件夹也新建了（如果父文件夹不存在）。说白了，就是能确保你想要查找的文件夹存在，因为我们一般发现某个文件夹不存在的话肯定会想要新建这个文件夹，这个API则帮我们把操作都实现了：
```javascript
const fs = require('fs-extra');

// 异步
fs.ensureDir(dest, err => console.log(err));

// 同步
try {
  fs.ensureDirSync(dest);
} catch (err) {
  throw err;
}
```

# 文件操作
## 读取文件内容
之前介绍了如何读取文件夹内容，但如果我们想要读取文件内容又如何操作呢？这里有两种方法。

### fs.readFile
通过这个方法我们可以获取到文件的内容：
```javascript
// 异步
fs.readFile(dest, {encoding: 'utf8'}, (err, data) => {
  if (err) {
    throw err;
  } else {
    console.log(data);
  }
});

// 同步
try {
  const data = fs.readFileSync(dest, {encoding: 'utf8'});
} catch (err) {
  throw err;
}
```

这里我们options可以设置参数：`encoding`，代表返回的data内容字符串编码方式，如果不指定的话，则返回的data格式为buffer。

### fs.readJson
除了上面那个方法，我们还可以使用`readJson`方法，当然这个是`fs-extra`里的扩展方法。因为我们现在存储数据很多时候都用JSON文件来存储，利用这个方法可以很方便的读取JSON文件，返回的是将JSON内容转化为Object的JSON对象，方便我们对数据进行操作。
```javascript
const fs = require('fs-extra');

// 异步
fs.readJson('./package.json', (err, packageObj) => {
  if (err) {
    throw err;
  } else {
    console.log(packageObj.version);  // 可以直接对对象的属性进行访问
  }
});

// 同步
try {
  const packageObj = fs.readJsonSync('./package.json');
} catch (err) {
  throw err;
}
```

## 写入文件内容
有了读取文件内容，我们还需要写入文件内容，同样有两种方法，一种是`writeFile`，另一种是`writeJson`。前者和`readFile`差不多，只是第二个参数变成了要写入的内容，具体可参考[文档](http://nodejs.cn/api/fs.html#fs_fs_writefile_file_data_options_callback)。

后者则也是对JSON文件进行的写入，要注意的是，每次写入都会覆盖之前所有的内容，所以我们如果想在原有的基础上新增内容，则需要先读取修改完内容再写入，这里我就只演示异步操作：
```javascript
const fs = require('fs-extra');

// 读取
fs.readJson('./package.json', (err, packageObj) => {
  if (err) {
    throw err;
  } else {
    packageObj.newProperty = 'new Property';
    fs.writeJson('./package.json', packageObj, err => {
      if (err) {
        throw err;
      } else {
        console.log('success');
      }
    })
  }
})
```

# 拷贝文件或者文件夹
在`fs-extra`模块，提供了`copy(src, dest, [options, callback])`操作，其中`src`可以为文件夹也可以为文件。当`src`为文件夹，则会拷贝文件夹里的所有文件和文件夹，要注意`src`和`dest`必须同时为文件夹或者同时为文件，这样才能正确的把`src`的内容拷贝到`dest`里。
```javascript
const fs = require('fs-extra');
const src = '/tmp/srcfile';
const dest = '/tmp/destfile';

fs.copy(src, dest, err => {
  if (err) console.log(err);
})
```

# 获取文件状态
当我们想要获取某个文件的状态，我们可以用`fs.stat(path)`、`fs.lstat(path)`、`fs.fstat(path)`，它们都返回一个`fs.stats`类对象。区别在于，`fs.lstat`的path可以是符号链接，`fs.fstat`的path是文件描述符`fd`。

返回的`Stats`类包含许多代表文件状态的属性和方法，常用的有：
- stats.isDirectory()。判断是否是文件夹
- stats.isFile()。判断是否是文件
- stats.dev。包含改文件的设备的数字标识符
- stats.size。文件的大小（字节为单位）
- stats.atime。上次访问此文件的时间戳。（要注意是系统本地时间，因此如果复制到别的电脑系统时间不一样可能会造成不可预知的后果，所以慎用）
- stats.mtime。上次修改此文件的时间戳。

# 总结
这里我只粗略地介绍了几个常用的方法，`fs`还有很多文件操作，官方文档有更加详细的解释，感兴趣的小伙伴可以戳[这里](http://nodejs.cn/api/fs.html)了解更多，帮助自己在使用NodeJS操作文件时更加得心应手。