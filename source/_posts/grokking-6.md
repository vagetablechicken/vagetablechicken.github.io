---
title: Grokking Pattern 6
date: 2021-05-06 10:58:13
tags: [Algo, LeetCode]
categories: Algo
---

# Grokking the Coding Interview: Patterns for Coding Questions

See {% post_link grokking-1 %}.

## 6. Pattern In-place Reversal of a LinkedList

### Reverse a LinkedList (easy)

```
Given the head of a Singly LinkedList, reverse the LinkedList.
 Write a function to return the new head of the reversed LinkedList.
```

反转的基本操作：两个指针，一前一后，改变后一节点的next指针，同时就会失去后一节点的原next，所以需要第三个指针，保住后一节点的原next。

再优化：由于每一次新的比较，都只需要两个指针，指向“后一节点的next节点”的指针是可以在此基础上再找到的，不需要一直追着。

遍历结束后，最后一个节点就是头节点，所以不需要别的额外的变量。

### Reverse a Sub-list (medium)

```
Given the head of a LinkedList and two positions ‘p’ and ‘q’, reverse the LinkedList from position ‘p’ to ‘q’.
```

p,q是positions，也就是第几个元素。不是value，更不是直接的node地址。

### Reverse every K-element Sub-list (medium) *

```
Given the head of a LinkedList and a number ‘k’, reverse every ‘k’ sized sub-list starting from the head.

If, in the end, you are left with a sub-list with less than ‘k’ elements, reverse it too.
```

首先看边界，因为末尾不足k的也翻转，所以可以无脑翻转。如果题目末尾变成不足k的**不可以翻转**（[leetcode 原题](https://leetcode.cn/problems/reverse-nodes-in-k-group/)），那么就得先数k，不到k个就退出，有k个再翻转，多了一次遍历，不过也是O(n)。

题目有些细节，但本质还是两个指针的事情，不难做对。不过我第一反应是不会改动初始链表的，但这一题如果加哨兵（一个dummy node接上链表head），会更简单点，不用在遍历while里写if判断是不是首次翻转，如果是首次，需要存下这个new head。（效率还是更高的，虽然不算太重要）

### Problem Challenge 1 - Reverse alternating K-element Sub-list (medium)

```
Given the head of a LinkedList and a number ‘k’, reverse every alternating ‘k’ sized sub-list starting from the head.

If, in the end, you are left with a sub-list with less than ‘k’ elements, reverse it too.
```

### Problem Challenge 2 - Rotate a LinkedList (medium)
