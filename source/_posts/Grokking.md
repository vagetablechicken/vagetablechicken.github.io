---
title: Grokking
date: 2021-05-06 10:58:13
tags: [Algo, LeetCode]
categories: Algo
---

# Grokking the Coding Interview: Patterns for Coding Questions

[课程原地址]( https://www.educative.io/courses/grokking-the-coding-interview?aff=K7qB)

[题目目录与答案 Python版](https://github.com/cl2333/Grokking-the-Coding-Interview-Patterns-for-Coding-Questions)

[题目目录与答案 C++版(附题目OJ的地址)](https://github.com/Huixxi/Algorithm-with-Cplusplus/tree/master/%E8%8E%B1%E7%89%B9%E6%89%A3%E7%9A%84-%E7%B3%BB%E5%88%97)——推荐OJ测试自己的算法

## Pattern 1 : Sliding Window

### Maximum Sum Subarray of Size K (easy)
[geeks oj](https://www.geeksforgeeks.org/find-maximum-minimum-sum-subarray-size-k/)
k长的**连续子序列**，使该子序列和最大，这个和为output。
窗口长度都固定了，只需在遍历一遍时加后一个减前一个就行了，O(n)。

### Smallest Subarray with a given sum (easy/medium)

[209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

注意读题，nums都是正整数，target也是正整数。这个条件大概率需要用到。

假设有一段连续子序列了，它已经>=target了，就不需要再继续加入元素了（序列外右边第一个元素），因为再加的话size就变大了。而这个size可以向右横移一格，可以立马算出新的sum（这里很节约时间，降低算法复杂度）。

这个sum如果 >= target，那么它就有机会再小一点，而这一次需要的是减去序列内的第一个元素，可以理解为“收缩窗口”。
这个sum如果 < target，那就可以再向右横移了，因为size扩大，对结果没有任何帮助。

总体看下来，只要横移和收缩，算法复杂度是O(n)。

#### 初步思路
初步思路是，先算个nums[0...x]之和 >= target的窗口，然后这个窗口开始向右移并尝试收缩。但其实不对，因为第一个窗口，不是非得从第0个元素开始，这个窗口自身就应该尝试收缩。这个逻辑补上后是能ac的。
不过，初步思路翻译为代码，还是有些小坑，肉眼很难查。建议背一个滑动窗口模板。

#### 滑动窗口模板思路
此思路最核心的思想就是，不强求窗口横移，反正先向右扩展1个，再左边收缩一个，就达到了横移。
因此滑动窗口的头尾都可以移动，而且是分别移动，用两个变量来表示，（start，end）。
因为收缩（start向右移）和扩展（end向右移）可以各做各的，所以没必要先找到一个总和 >= target的初始窗口了。

所以，步骤可以化简为，**每一次都扩展1下，然后尽力收缩**（while不定次收缩）。

```
while end < n:
	tmp_sum += nums[end]
	while tmp_sum >= target:
		win_size = min(win_size, end-start+1)
		tmp_sum -= nums[start]
		start += 1
	end += 1
```
至于几个变量的初始值，现场推理一下也可以得到，不做赘述。
P.S. win_size没必要用int的max，用len(nums)+1就可以了，反正都是不可能的值。

#### 补充

滑动窗口的算法，nums就算有负数也无所谓。这个条件大概是为“前缀和”这一方法准备的。

前缀和方法，简单来讲就是nums[0..i]之和组成一个sum数组，这个数组严格递增，都不存在相等的元素。

然后就可以用二分来找了（看到有序就要想到二分），当前sum[i]为0到i元素的和，`sum[j] - sum[i] >= target`转换为`sum[j]>=target+sum[i]`，那么就是在sum数组里找`target+sum[i]`的lower bound。

为什么不是以j为结尾，`sum[j] - sum[x] >= target`转换成`sum[x] <= sum[j]-target`，找sum[x]呢？因为是upper bound（第一个>某值的元素）的前一个（必定<=某值），不如直接找lower bound简洁，而且不用处理减出负数的情况。

### Longest Substring with K Distinct Characters (medium, google) -- LintCode

