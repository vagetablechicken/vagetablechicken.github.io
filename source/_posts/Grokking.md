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



P.S.注意到了吗，这个题目和前面一题Smallest Subarray的区别？

看不出来也正常，我做了几道题才突然回头发现的😂而且明明之前做过笔记，重蹈覆撤🙄️

~~果然人类的本质都是复读机~~

这个题已经开始了longest之路，也就是说，只是想求一个最长的长度，根本不在意“重复无意义的更新”。

按之前的模板，此题的伪代码应写为：

```
while end < n:
	dic[s[end]]+=1
	while len(dic)>k:
		dic[s[start]]-=1
		if dic[s[start]]==0:
			del dic[s[start]]
		start+=1
	longest = max(longest, end-start+1)
	end+=1
```

这样的写法，保证了longest变量更新时，当前窗口都是len(dic)<=k的，也就是合法的情况。但确实可能会出现窗口被收缩的很小的时候（为了合法），此时max更新也是白干的（longest还是原值）。

而如果代码将内层while变为if：

```
while end < n:
	dic[s[end]]+=1
	if len(dic)>k:
		dic[s[start]]-=1
		if dic[s[start]]==0:
			del dic[s[start]]
		start+=1
	longest = max(longest, end-start+1)
	end+=1

自然，在longest更新时，当前窗口可能都还满足条件，不合法，但仍旧去做了一次longest的更新。但从数值上来讲，由于不满足条件会被收缩一次，加上前面的end扩展一次，窗口等于做了一次平移。那么end=start+1的值就不会变大，longest的更新自然也是不会有实际作用的。

这一个改动，它到底好在了哪里呢？光看经过，start，end两个游标都是单向前进的，2-while和while-if都不会使两个游标左右飘。但由于2-while会保证每次窗口都是合法的，和while-if相比，start这个游标可能会更靠右。举个极端例子，如果longest是[0, n-2]这个窗口，下一次扩展end就会到末尾n-1，while-if此时发现窗口不合法，收缩一次，就溜了。而2-while，会愣是要求[start, n-1]窗口合法，可能start从0不断右移，直到n-1才停止。无用操作在这个例子就占了一半。具体例子就是AAAAB, k =1。

这个优化并不容易读，我也不建议平时代码搞这么tricky。在这个题目里，仅仅在节省len查询和dict更新，都是较高效的操作，而且有次数上限（由于start标最多都走到末尾，操作次数最多n次）。优化效果仅仅锦上添花。但这个将while改为if，在其他题目中可能有奇效。因为while的判断条件如果复杂度较高，这里的收益就很大了，见Longest Substring with Same Letters after Replacement一题。

### [Fruits into Baskets (medium) -- LeetCode](https://leetcode.com/problems/fruit-into-baskets/)

跟上题一模一样。题目暗示着连续区间，就可以尝试滑动窗口方法。

但这一题case比上一题的规模大，最简单的dict实现（不及时删除value为0的项，每次都要遍历得到distinct count）这种方法就超时了。还是推荐及时删除value为0的项，其实比不删还简单，因为删除这个操作只会在收缩时出现（只有此处count--）。

可优化为while-if。

### [No-repeat Substring (medium) -- LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

和前面的题目毫无差别。

但注意第二层while的判断条件是什么，如果判断条件是“窗口dict的value全为1”，那么你可以改出while-if。

如果是“当前字符c的count>1”，就改不了了。因为现在只看当前字符，那就意味着窗口必须是合法的，然后扩展，加入当前字符。while-if便不可用。

“只看当前字符”这种方法的优化思路是，如果start要跳，直接让start去“当前字符上一次出现的位置的右边”。 因为前面的字符都可以跳过了。还可以再化简代码，但会很难读，所以还是适合而止吧。

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

统计问题虽然简单，但是复杂度略高。想要快速，可能需要两个map，ch->count, count->ch。所以，while-if就很适合了，由于只收缩一次，max_count就从dict里统计一次就好了，不用强求更快速。

### [Sliding Window Maximum (hard) -- LeetCode](https://leetcode.com/problems/sliding-window-maximum/)

这个题跟滑动窗口模板毫无关系，窗口大小都固定牢了。唯一的问题点在于用什么结构体来提炼窗口信息，既能很快查到最大值，又能在窗口滑动时很快更新好（滑动本质就是加入一个数，去掉一个数）。快速查到最大值，可以想到利用堆。但是堆有一个明显问题，就是它不适合去删除内部的某个元素（不是堆首）。而“去掉一个数”这个操作很可能就是去删除某个中间的值。堆的删除操作是个不太ok的操作，因为这个堆是简单的堆（并非压平了看，完全有序的，不是堆排序之后的结果），也就是删除操作不能快速定位到要删除的元素。

但是别直接放弃堆（我就放弃了，想别的方法，走远了）。这时候，尝试多推导一下，就会发现，不用着急删除元素。因为元素可以跟上自己的位置idx，如果堆顶拿到的idx已经不在窗口内，再删除也不迟。也就是延迟了删除操作，正好还让删除操作符合堆的操作习惯，只删堆顶。

到这里，起码算是题目做出来了，而且这个方法也不差，能交差。

但这道题更想考的点是别的，所以还需要再优化。说是优化，不如说是换了解法。从堆想到单调队列，反正我是做不到。。。

单调队列是什么东西？先记住它能够动态地维护定长序列中的最值。所以只要定长、最值，就可以想到尝试单调队列。具体来讲，单调队列是，push元素前会把前面的元素都从后往前访问一遍（遇到比自己大的停止），比当前元素小的都删除，再将元素放在队尾。其实，全访问也没关系，最后结果没有差别，不过，遇到比自己大的就停止可以节省点时间。最后的结果就是队列里头到尾是从大到小有序的。

举例说明(为了多点情况，和leetcode原例子有差别)：

```
nums = [1,3,-1,-3,-2,3,6,7], k = 3
```

单调队列应该长什么样？

[1,3,-1]初始窗口入队列后，q=[3,-1]，那么这个窗口的最值就是3；

[3,-1,-3], q=[3,-1,-3]，最值还是3；

[-1,-3,-2], q=[3,-1,-2]，最值3，但3已经不在窗口内，所以pop，q=[-1,-2]，最值-1；

[-3,-2,3], q=[3]，最值3；

[5,3,6], q=[6]，最值6；

[3,6,7], q=[7]，最值7。

可以看到，只用进行简单比较，就能维持着最值，比堆的时间复杂度低。

单调队列它为什么做到了呢？可以这么想，3,-1,-3,-2序列，前三个最值是3，次值是-1，当窗口离开了3，次值是很有可能当选最大值的，维护信息里是不可能放弃这个值的。而窗口滑到-1,-3,-2这个序列，为什么-3无用了，因为-2进入，-3小于-2，直到-3离开窗口 ，它都没有-2可能成为最大值候选者。因为-3被扔掉是合理的。也就是说？

