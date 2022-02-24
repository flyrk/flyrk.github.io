---
title: leetcode刷题ing(week-one)
date: 2017-04-09 20:58:30
categories: 
- 算法
tags: 
- leetcode
---

前段时间面试时算法题做的一塌糊涂，深感自己算法还有很大的不足，所以这周开始在leetcode刷题了。
其实早就知道leetcode这个网站，以前大一时都是在OJ刷题，当时都是用C++，后来发现leetcode的题也很全，而且支持Javascript，这点让我很开心，于是就转战leetcode吧！以后基本每天都会刷刷题，练练算法，每周总结一些有意思的题目的思路，也当给自己复习～代码就不贴了。。。
<!--more-->
### [Longest Uncommon Subsequence I](https://leetcode.com/problems/longest-uncommon-subsequence-i/#/description)
> 题目描述大概是这样的:给定两个字符串，让你找出两个字符串中的最大不同子字符串的长度。最大不同子字符串的定义是：属于某一个字符串的最大子字符串并且不属于另一个字符串的子字符串。比如说"abcad", "abcda"这两个字符串的最大不同子字符串就是它们自己任何一个。

开始想着是找最大子字符串用动态规划或者什么的，后来在草稿纸上研究了一下，发现原来就是个简单的找规律：当两个字符串的长度不一样时，最大不同子字符串肯定是二者中长度更长的那个;如果两个字符串长度相同，则判断一下二者是否相等，不等则就是它们的长度，相等则没有。

### [Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/#/description)
> 题目大意是：给两个字符串，判断二者是否同构。同构的意思就是二者的形式是否相同。比如"abba"和"cddc"就是同构，"abcde"和"abcdf"就不是同构。

一开始没完全理解题意，以为光形式相同就行了，只要用一个数组统计字符串1出现的连续字母长度然后与字符串2比较看字符长度是否相同且顺序一样就行了。但后来发现没有考虑前后字母是否一样，即使长度一样但字母不一样也算匹配失败。后来想了半天才找到一个好的方法：首先二者长度肯定要相同，然后直接O(n)循环一次，用两个类似hashmap数组统计字符串1和字符串2每个字母的最新坐标，如果字符串1和字符串2的第i个字母的坐标不一样则代表匹配失败。

### [Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/#/description)
> 题目大意：给定一个数组，数组中每一个元素都有h,k两个值。h代表该元素的大小，k代表大于等于该元素的h值的元素有多少个。让你重新构造队列使其符合逻辑。

其实这就是一个贪心策略：我们先对整个数组people排序，按h值从高到低来排，如果h值相同，则k值从低到高来排。这样我们向结果数组依次从数组people取剩余最大的元素插入到与该元素的k值相同的位置。这样我们保证了结果数组里面总是剩下的里面最大的，剩下的只要依次按照k值插入合适的位置就行了，因为结果数组里面保证了大于等于每次插入的元素的h值。

### [Assign Cookies](https://leetcode.com/problems/assign-cookies/#/description)
> 题目大意：给两个数组g和s，g[i]代表每个孩子想要吃的蛋糕大小，s[i]代表每个蛋糕的大小，问你怎么分才能使尽可能多的孩子吃到蛋糕。

又是贪心策略：我们先对两个数组排序，然后从g中最小的元素开始，找到s里满足大于等于该元素的最小值，然后从该值的位置接着循环g中下一个元素，直到找不到满足条件或者遍历整个s数组为止，统计满足条件的次数。

### [Arithmetic Slices](https://leetcode.com/problems/arithmetic-slices/#/description)
> 题目大意：给一个数组让你找到里面最多能有几个连续的等差数列。

简单的数学题：我们扫一遍数组，统计每个连续出现的等差数列元素个数，然后该等差数列可以划分成(n-1)*(n-2)/2个等差数列，然后把结果累加就行了。

### [Counting Bits](https://leetcode.com/problems/counting-bits/#/description)
> 给定一个数字n统计0～n里每个数字的二进制中1的个数。

主要用到的就是位运算，关键是统计数字的二进制中1的个数，剑指Offer中看到过一个方法：
```javascript
var count = 0;
while(n) {
  count++;
  n = n & (n-1);
}
```
因为n & (n-1)是把n最右边的1去掉，循环多少次，则代表n有多少个1。

### [Add Digits](https://leetcode.com/problems/add-digits/#/description)
> 给一个数字，返回它的每一位相加循环直到结果为个位数的值。例如：num = 38, 3 + 8 = 11, 1 + 1 = 2. 返回2。

这里有一个数学公式，wiki上有[digital_root](https://en.wikipedia.org/wiki/Digital_root#Congruence_formula)，具体我就没推了，直接用结论：dr(n) = 1 + (n - 1) % 9
