---
title: 什么是堆排序
date: 2017-09-05 10:02:06
categories:
- 算法
tags:
- 排序
---

堆排序是一种常见的排序算法，时间复杂度是O(nlgn)，与归并排序一样，但它又与插入排序一样具有*空间原址性* ：任何时候都只需要常数个额外的元素空间存储临时数据。
<!--more-->
# 什么是堆？
  一般堆用数组存储，表现出近似完全二叉树形式，树上的每一个结点对应数组中的一个元素。除了最底层外，该树是完全充满的且从左至右填充。

# 最大堆和最小堆
  * 最大堆：除了根以外的所有节点i都要满足A[parent(i)]>=A[i]，即堆中最大元素是根节点。
  * 最小堆：除了根以外的所有节点i都要满足A[parent(i)]<=A[i]，即堆中最小元素是根节点。   

# 堆中节点的高度
  与二叉树的高度相同，定义为该节点到叶节点最长简单路径上边的数目。则包含n个元素的堆其高度为lgn。

# 维护堆的性质与方法（数组下标都从1开始）
  ## maxHeapify
  时间复杂度为O(lgn)。通过让A[i]的值在最大堆中“逐级下降”，从而使得以下标i为根节点的子树重新遵循最大堆性质。代码如下:
  ```javascript
  function maxHeapify(arr, i) {
    let largest;
    let left = i * 2; // leftChild
    let right = i * 2 + 1; // rightChild
    if( left <= arr.length && arr[i] < arr[left] ) {
      largest = left;  
    } else {
      largest = i;
    }
    if( right <= arr.length && arr[largest] < arr[right] ) {
      largest = right;
    }
    if ( largest !== i ) { // 把左右子节点中最大的元素与当前节点i交换
      arr[i] = arr[i] + arr[largest];
      arr[largest] = arr[i] - arr[largest];
      arr[i] = arr[i] - arr[largest];
      maxHeapify(arr, largest);
    }
  }
  ```
  ## buildMaxHeap
  时间复杂度为O(n)。用*自底向上*的方法利用maxHeapify把大小为n的数组转换为最大堆。因为最后一个叶节点序号为n，则其父节点序号为n/2,所以子数组[n/2+1,....,n]都是堆的叶节点，所以循环从n/2开始递减到1，每一次都保证节点i+1,i+2...,n都是一个最大堆的根节点的性质。
  ```javascript
  function buildMaxHeap(arr) {
    for (var i = arr.length / 2; i >= 1; i--) {
      maxHeapify(arr,i);
    }
  }
  ```
# heapSort:堆排序算法。
  有了上述两个函数方法，我们就可以实现堆排序。
  
  先将数组arr建为一个最大堆，因为最大堆的根节点总是最大的，通过把它与arr[n]互换可以得到正确位置，保证arr[n]总是当前堆中最大元素，然后将arr[n]存储到新的数组中。可以算出时间复杂度为O(nlgn)。
  ```javascript
  function heapSort(arr) {
    let arrSort = [];
    buildMaxHeap(arr);  // 先建一个最大堆
    let length = arr.length;
    for (var i = length; i >= 2; i--) {
      arr[1] = arr[i];
      arrSort.push(arr[i]);
      maxHeapify(arr, 1); // 每次交换后重新维护最大堆，复杂度为O(lgn)
    }
    arrSort.push(arr[1]);
    return arrSort;
  }
  ```
# 总结
  通过堆排序的实现，我们可以在时间复杂度为O(nlgn)的情况下对数组进行排序。而且当我们只需要找出数组中最大的几个元素，则可以用堆排序来实现，因为每次最大的元素总是当前最后一个，这样就不需要将数组全排序。