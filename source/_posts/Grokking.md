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

### [Maximum Sum Subarray of Size K (easy) -- GeeksforGeeks](https://www.geeksforgeeks.org/find-maximum-minimum-sum-subarray-size-k/)

k长的**连续子序列**，使该子序列和最大，这个和为output。
窗口长度都固定了，只需在遍历一遍时加后一个减前一个就行了，O(n)。

### [Smallest Subarray with a given sum (medium) -- LeetCode](https://leetcode.com/problems/minimum-size-subarray-sum/)

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

滑动窗口的算法，nums就算有负数也无所谓。（被烟雾弹迷惑🐶）这个条件大概是为“前缀和”这一方法准备的。

前缀和方法，简单来讲就是nums[0..i]之和组成一个sum数组，这个数组严格递增，都不存在相等的元素。

然后就可以用二分来找了（看到有序就要想到二分），当前sum[i]为0到i元素的和，`sum[j] - sum[i] >= target`转换为`sum[j]>=target+sum[i]`，那么就是在sum数组里找`target+sum[i]`的lower bound。

为什么不是以j为结尾，`sum[j] - sum[x] >= target`转换成`sum[x] <= sum[j]-target`，找sum[x]呢？因为是upper bound（第一个>某值的元素）的前一个（必定<=某值），不如直接找lower bound简洁，而且不用处理减出负数的情况。

### [Longest Substring with K Distinct Characters (medium, google) -- LintCode](https://www.lintcode.com/problem/longest-substring-with-at-most-k-distinct-characters/description)

Given a string, find the length of the longest substring in it with no more than K distinct characters.

也就是LeetCode340题 Longest Substring at Most K Distinct Characters。

这道题输出是最长子串的长度，所以显然可以套用滑动窗口的模板。和上一个题目一样，都是扩展会使得条件不符，要通过收缩来满足条件。
即，窗口扩展一旦字符种类超过k，就可以通过窗口收缩来削减字符种类。
唯一不同的点是：update longest string的时机（其实就是用end-start+1来尝试更新记录的longest）
因为while distinct_count(dict) > k时是去收缩，break while时说明distinct_count(dict) 已经<=k了，这个时候的[start, end]才是一个可能解，符合条件的可能解。
所以update longest string len是在while循环外，而且是之后。
至于如何实现dict和distinct_count，随便吧，简单也好，高效也好。

### [Fruits into Baskets (medium) -- LeetCode](https://leetcode.com/problems/fruit-into-baskets/)

跟上题一模一样。题目暗示着连续区间，就可以尝试滑动窗口方法。

但这一题case比上一题的规模大，最简单的dict实现（不及时删除value为0的项，每次都要遍历得到distinct count）这种方法就超时了。还是推荐及时删除value为0的项，其实比不删还简单，因为删除这个操作只会在收缩时出现（只有此处count--）。

### [No-repeat Substring (medium) -- LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

和前面的题目毫无差别。

### [Longest Substring with Same Letters after Replacement (medium, amazon) -- GeeksforGeeks](https://practice.geeksforgeeks.org/problems/maximum-sub-string-after-at-most-k-changes/0)

We have a string **s** of length n, which consist only UPPERCASE characters and we have a number k (always less than n and greater than 0). We can make at most k changes in our string such that we can get a sub-string of maximum length which have all same characters.
 

**Example 1:**

```
Input: s = "ABAB", k = 2
Output: 4
Explanation: Change 2 'B' into 'A'.
```

**Example:**

```
Input: s = "ABCD", k = 1
Output: 2
Explanation: Change one 'B' into 'A'.
```



这种要替换字符的题目，往往不用真的替换，只要数值上达到某个条件就行了。比如这个题，不用想着应该替换哪些字符，而应该想“总字符个数 - 不需要被替换的字符个数 >= k”就行了。很容易想到，“不需要被替换的字符个数”就是区间内个数最多的那个字符，这样，总字符数才能多一点（对某个区间而言，不需要理会那些无意义的可能解，“保留频率最高的字符，把其他的替换为该字符”肯定是操作数最少的）。于是，替不替换的问题就化简为简单的统计问题。

