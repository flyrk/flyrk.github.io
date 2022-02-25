---
title: 有意思的leetcode算法题——巧用数学知识
date: 2019-02-27 12:42:43
categories:
- 算法
tags:
- leetcode
---

最近在leetcode上刷算法题，发现了一道比较有意思的题目，虽然不难，但要想尽可能的降低时间复杂度达到最优解，还是要有点技巧的，我们来看看。
<!-- more -->

# 题干
[sum-of-even-numbers-after-queries](https://leetcode.com/problems/sum-of-even-numbers-after-queries)。这道题的意思是说，给定一个包含一系列数字的数组A，和一个queries数组，对于`queries[i]`，设定`val=queries[i][0]`，`index=queries[i][1]`，使得`A[index]+=val`，求`ans`数组，使得`ans[i]`为经过`queries[i]`之后的A数组所有偶数数字之和。

# 初步模拟
初看完题目，我马上想到这不就是个简单的模拟题吗？照着题干的意思写不就行了，再一看数据范围不超过10000，嗯，O(n^2)先试试看，应该不会超时。于是很快的就把代码写出来了：
```javascript
/**
 * @param {number[]} A
 * @param {number[][]} queries
 * @return {number[]}
 */
var sumEvenAfterQueries = function(A, queries) {
    var ans = [], val = null, index = 0, evenSum = 0;
    for (var i = 0, l = queries.length; i < l; i++) {
        val = queries[i][0];
        index = queries[i][1];
        A[index] += val;
        evenSum = 0;
        for (var j = 0, l2 = A.length; j < l2; j++) {
            if (A[j] % 2 === 0) {
                evenSum += A[j];   
            }
        }
        ans.push(evenSum);
    }
    return ans;
};
```

写完提交，成功！emmmm，很简单嘛，下一题.....

等等，这时间好像有点多啊，8216ms，差一点就超时了。不行，肯定有什么更简便的方法。

# 数学妙用
在经过别人的答案启发后，我突然发现原来利用数学的知识这道题可以这么简单！话不多说，我们来看代码：
```javascript
var sumEvenAfterQueries = function(A, queries) {
  var ans = [], val = null, index = 0, tmpSum = 0, evenSum = 0;
  evenSum = A.reduce((p, n) => {
    return n % 2 === 0 ? p + n : p;
  }, 0);
  
  queries.forEach((query) => {
    val = query[0];
    index = query[1];
    tmpSum = A[index] + val;
    tmpSum % 2 === 0 ?
      (A[index] % 2 === 0 ? evenSum += val : evenSum += tmpSum)
      : (A[index] % 2 === 0 ? evenSum -= A[index] : null)
    A[index] = tmpSum;
    ans.push(evenSum);
  })
  return ans;
};
```

写完一提交居然只用了20ms左右！！！复杂度也只有O(n)，怎么做到的呢？

我们来分析分析：
```javascript
evenSum = A.reduce((p, n) => {
  return n % 2 === 0 ? p + n : p;
}, 0);
```
首先我们先把原始数组所有偶数数字加起来，为了方便之后直接在上面加减。

```javascript
tmpSum % 2 === 0 ?
      (A[index] % 2 === 0 ? evenSum += val : evenSum += tmpSum)
      : (A[index] % 2 === 0 ? evenSum -= A[index] : null)
    A[index] = tmpSum;
    ans.push(evenSum);
```
接着这段最关键的代码，我们对每次加完后的数字`tmpSum`进行判断，如果是偶数的话，分两种情况：`A[index]`是偶数，则代表之前的evenSum已经把`A[index]`加进去了，所以我们只用加上新的`val`，反之我们把`A[index]`和`val`都给加上；如果是奇数，我们则需要判断之前的`A[index]`是不是偶数，是的话需要把它给减去。

经过这样的处理最后得到的就是每一轮所有偶数的和，最后完美的解决了问题，时间复杂度也只有O(n)。

# 总结
虽然这道题很简单，但我还是想记录下来，因为它代表了一种思维方法，以后做算法题或者写业务代码时都应该多问问自己，还有没有最优解？还能不能继续优化？我们很多时候都是做完结果对了就万事大吉，等到问题出现的时候才去想怎么去解决。而大多时候的问题都是有更好的解决办法的，不怕你做不到，就怕你想不到。平时做事时多拓宽自己的思路，我们在真正遇到问题的时候才不会害怕。