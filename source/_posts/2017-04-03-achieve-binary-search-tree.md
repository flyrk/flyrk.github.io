---
title: 二叉搜索树JavaScript实现
date: 2017-04-03 14:24:32
tags: 
- Tree
- JavaScript
categories: 算法
---

### 什么是二叉搜索树
其形式就是二叉树，对于每个节点x，其左子树的值<=x.value，右子树的值>=x.value。
<!--more-->
### 二叉搜索树的遍历与查询

对于二叉搜索树，我们可以使用中序遍历，得到树上从小到大所有的元素。时间复杂度平均为O(n)。
```
function inorderTreeWalk(x) {
  if(x!== null) {
    inorderTreeWalk(x.left);
    print(x.key);
    inorderTreeWalk(x.right);
  }  
}
```

当我们想要查询二叉搜索树中某个关键字应该怎么做呢？由于二叉搜索树左子树和右子树的特点，我们很容易写出：
```
function treeSearch(x,k) {  //最开始x代表根节点
  if(x === null || x.key === k) {
    return x;
  }
  if(k < x.key) {
    return treeSearch(x.left, k);
  } else {
    return treeSearch(x.right, k);
  }
}
```
我们还可以写出循环版本（一般比递归更高效）:
```
function treeSearch(x, k) {
  while(x !== null || k !== x.key) {
    if(k < x.key) {
      x = x.left;
    } else {
      x = x.right;
    }
  }
  return x;
}
```
我们很容易可以找到二叉搜索树中的最小元素和最大元素，其查找时间复杂度为O(lgn)：
```
function treeMinimum(x) {
  while(x.left !== null) {
    x = x.left;
  }
  return x;
}

function treeMaximum(x) {
  while(x.right !== null) {
    x = x.right;
  }
  return x;
}
```
有时候我们需要按中序遍历查找二叉搜索树某个节点的后继和前驱。分析二叉搜索树性质可知，如果该节点右子树不为空，则它的后继应该是右子树中的最小元素；若右子树为空，则该节点的后继应该是其双亲有左子树的父节点。前驱则与其对称.时间复杂度O(lgn)：
```
function treeSuccessor(x) {  //后继查找
  if(x.right !== null) {
    return treeMinimum(x.right);
  } else {
    let y = x.parent;
    while(y !== null && x === y.right) {
      x = y;
      y = y.parent;
    }
    return y;
  }
}

function treeSuccessor(x) {  //前驱查找
  if(x.left !== null) {
    return treeMaximum(x.left);
  } else {
    let y = x.parent;
    while(y !== null && x === y.left) {
      x = y;
      y = y.parent;
    }
    return y;
  }
}
```
### 二叉搜索树实现插入和删除操作。
　　1. 插入相对比较简单，我们只要遍历二叉树把值插入到合适的位置就行了。
　　```
        function treeInsert(tree, newNode) {
          let y = null;
          while(tree !== null) {
            y = tree;
            if(newNode.key < tree.key) {
              tree = tree.left;
            } else {
              tree = tree.right;
            }
          }
          newNode.parent = y;
          if(y === null) {  // 树为空
            tree.root = newNode;
          } else {
            if(newNode.key < y.key) {
              y.left = newNode;
            } else {
              y.right = newNode;
            }
          }
        }
        ```
　　2. 删除操作相对较复杂。假如删除z节点，我们考虑三种情况：z没有子节点、z有一个子节点、z有两个子节点。
　　　　- 第一种情况：z没有子节点，则直接删除z，修改父节点属性，用null代替z。
　　　　- 第二种情况：z只有一个子节点，则用子节点代替z。
　　　　- 第三种情况：z有两个子节点，我们需要找到z的后继y，y肯定在z的右子树并且没有左孩子，当y恰好是z的右子节点，则直接用y替代z，并留下y的右孩子；否则先用y的右子节点替代y，再用y替代z。
        ```
        function transplant(tree, u, v) {  //子树替换父节点方法
          if(u.p === null) {
            tree.root = v;
          } else if(u === u.parent.left) {
            u.parent.left = v;
          } else {
            u.parent.right = v;
          }
          if(v !== null) {
            v.parent = u.parent;
          }
        }

        function treeDelete(tree, z) {
          if(z.left === null) {
            transplant(tree, z, z.right);
          } else if(z.right === null) {
            transplant(tree, z, z.left);
          } else {
            let y = treeMinimum(z.right);
            if(y.parent !== z) {
              transplant(tree, y, y.right);
              y.right = z.right;
              y.right.parent = y;
            }
            transplant(tree, z, y);
            y.left = z.left;
            y.left.parent = y;
          }
        }
        ```
**至此，我们完成了二叉搜索树的js实现。具体算法证明参考《算法导论》**
