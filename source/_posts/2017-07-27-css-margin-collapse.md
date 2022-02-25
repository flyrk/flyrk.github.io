---
title: CSS外边距合并效应
date: 2017-07-27 22:44:05
categories:
- CSS相关
tags:
- CSS技巧
---

大部分人都知道CSS布局时有外边距合并，也就是说如果给两个块元素设置margin-top或者margin-bottom值，它们上下之间的margin值会发生合并，但具体是根据什么来合并呢？我查了一下，大部分的解释都是**叠加后的margin值取两者之间的最大值**，也就是说，如果给上面的元素设置margin-bottom: 40px, 下面的元素设置margin-top: 50px,则最终两者之间的外边距为50px。但如果设置的值是负值呢？
<!--more-->

# 一个为负值，一个为正值

```CSS
.tops {
  width: 20px;
  height: 20px;
  background: rgb(94, 195, 27);
  margin-bottom: -50px;
}

.bottoms {
  width: 20px;
  height: 20px;
  background: rgb(29, 141, 213);
  margin-top: 60px;
}
```

![margin](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/margin-10.jpg)

可以看到，结果两者之间的外边距值为正负值相加，为10px。

# 两个都为负值

```CSS
.tops {
  width: 20px;
  height: 30px;
  background: rgb(94, 195, 27);
  margin-bottom: -10px;
}

.bottoms {
  width: 20px;
  height: 20px;
  background: rgb(29, 141, 213);
  margin-top: -20px;
}
```

![margin](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/margin-20.jpg)

最后两者之间外边距为-20px，也就是两个负值绝对值更大的那个。

# 总结

所以说，外边距合并一般有三种情况：
- 两者都是正值，取最大的；
- 两者一正一负，取二者相加的结果；
- 两者都是负值，取二者绝对值更大的；

> 但要注意的是，外边距合并只发生在块元素之间。一般有三种情况：
> - 相邻的兄弟姐妹元素：毗邻的两个兄弟元素之间的外边距会塌陷（除非后者兄弟姐妹需要清除过去的浮动）。
> - 块级父元素与其第一个/最后一个子元素：如果块级父元素中，不存在上边框、上内边距、内联元素、 清除浮动 这四条属性（也可以说，当上边框宽度及上内边距距离为0时），那么这个块级元素和其第一个子元素的上边距就可以说”挨到了一起“。此时这个块级父元素和其第一个子元素就会发生上外边距合并现象，换句话说，此时这个父元素对外展现出来的外边距将直接变成这个父元素和其第一个子元素的margin-top的较大者。
> - 空块元素：如果存在一个空的块级元素，其 border、padding、inline content、height、min-height 都不存在。那么此时它的上下边距中间将没有任何阻隔，此时它的上下外边距将会合并。

**参考文档**：[MDN-margin-collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)
