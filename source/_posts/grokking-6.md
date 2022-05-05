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

### Problem Challenge 1 - Reverse alternating K-element Sub-list (medium)

```
Given the head of a LinkedList and a number ‘k’, reverse every alternating ‘k’ sized sub-list starting from the head.

If, in the end, you are left with a sub-list with less than ‘k’ elements, reverse it too.
```

### Problem Challenge 2 - Rotate a LinkedList (medium)
