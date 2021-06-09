---
title: Grokking
date: 2021-05-06 10:58:13
tags: [Algo, LeetCode]
categories: Algo
---

# Grokking the Coding Interview: Patterns for Coding Questions

[课程原地址]( https://www.educative.io/courses/grokking-the-coding-interview?aff=K7qB)

[题目目录与答案 Python版](https://github.com/cl2333/Grokking-the-Coding-Interview-Patterns-for-Coding-Questions)——完整题目目录，包含challenge。 

[题目目录与答案 C++版(附题目OJ的地址)](https://github.com/Huixxi/Algorithm-with-Cplusplus/tree/master/%E8%8E%B1%E7%89%B9%E6%89%A3%E7%9A%84-%E7%B3%BB%E5%88%97)——推荐用OJ测试自己的算法，但是这个repo里基本没有challenge题目。

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
```

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

### [Longest Subarray with Ones after Replacement (medium) -- GeeksforGeeks](https://www.geeksforgeeks.org/longest-subsegment-1s-formed-changing-k-0s/)

比上题更简单，只需要一个int变量就能描述一个窗口内的0的个数。

### [Problem Challenge 1 - Permutation in a String (hard)](https://leetcode-cn.com/problems/permutation-in-string/)

排列，不允许多一个字符，所以窗口大小是固定的，滑动用来节省“更新窗口属性的代价”。题目如果对字符多加限制，比如此题限制只有小写字母，也就只有26个可能，其实描述窗口用长26的数组都可以，不用非要用counter。用counter有个麻烦点在于当某个字符的count为0时，你需要删掉它，不然就没法和s1的counter比大小。如果两个字符串的counter都先把26个字母的空间开出来，那还不如数组节省空间。

### [Problem Challenge 2 - String Anagrams (hard)](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/submissions/)

和challenge1没有区别

### [Problem Challenge 3 - Smallest Window containing Substring (hard) *](https://leetcode-cn.com/problems/minimum-window-substring/)

没啥特别，就是用collections.Counter()减法，比自己写的比较函数要慢不少，1200ms vs 500ms。

### [Problem Challenge 4 - Words Concatenation (hard)](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)

注意读题！words长度相同！匆忙读题后，我还以为要处理不同的切割方式，以为会出现“一个substring里有几种正确的word排列”。那这题可能不止hard了。

可以简单想到的办法就是窗口从0一直滑到尾，step为1，每一次窗口都得重新计算下word-count，然后和words参数比较。

显然这个没有什么巧妙点，没有节约计算量，所以想要利用滑动窗口，当然得想点骚方法。其实就是，每一次都设置一个起点，从这个起点开始，只会按词长来拓展，这样就能像字符型的题目一样，充分利用滑动窗口的扩展收缩，减少计算。举例说明，就是一个单词长度为len，s串总长为n的话，第一趟是从0开始一个len切一刀，这样切割后的串扩展和收缩都是一个词的，第二趟就是从1开始切，以此类推，最后一次是len-1开始切。可以想到，这样也是把所有可能性都考虑到了。

改进后的滑动窗口方法效果是明显的，1000ms到100ms。

### Additional

#### [Sliding Window Maximum (hard) -- LeetCode](https://leetcode.com/problems/sliding-window-maximum/)

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

单调队列它为什么做到了呢？

可以这么想，首先简单起见，只考虑窗口内元素的单调队列情况，比如3,1,2序列，k=3，很明显1会在2进入时被扔掉，因为2进入了之后，怎么也是2比1晚出窗口，就算可能是最大值，也是2可能，1是完全没有可能竞争最大值的。

再考虑队列前部还有窗口外元素的情况，窗口外的元素还留在队列里（可以称为过期元素），也就是说窗口内的单调队列最大值（也就是队列列首）比过期元素小。但这并不影响窗口元素那部分的选择，无论有没有过期元素，窗口那部分的单调队列都长一个样。所以过期元素可以简单地pop出来扔掉。

总的来说，单调队列就是剔除了“必然不能争最大值”的那部分无用值。单调队列实现上没有什么难度，不再赘述。

P.S. Python实现遇到超时问题，因为我没有用pop，而是选择到一个点，取出切片[:x]，然后又append。具体细节待调查。从耗时来看，很明显这一连串操作应该搞出了deep copy之类的耗时操作。都叫单调队列了，就好好用python collections里的deque。

#### [Shortest Subarray with Sum at Least K (hard) -- LeetCode](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

仔细读题，这个题的数组里有负数。滑动窗口的算法模板是没办法直接套用的，因为窗口收缩的停止条件是sum < k，由于有负数，你不能在窗口sum < k时停止，你必须继续收缩，否则就可能错过解。

停留在滑动窗口算法模板这个框架里，是没办法找到解法的。

这时候~~只能躺平~~。

只能说先回答个O(n^2)的解法吧，聊胜于无。很明显，所有区间和都是可能的最佳答案，所以前缀和加**二级遍历**所有区间，能得到答案。

当然可以优化，但是很难凭空想出来参考答案“单调队列”。接下来的说明是以知道“前缀和+单调队列是较好解法”为前提来看这个题目，所以没有什么顺理成章推导，全凭参考答案提示。

单调队列维护什么？

题目核心是求最短的合法子串，所以短是最核心的，和只要>=k什么都行。那么假设当前看i，以i开头的最佳子串，当然是从i开始寻找第一个j，能使sum[i..j]>k（前缀和数组里两个数减一下就好了）。j之后的都不用找了，没有比i到j更短的了。那么反过来思考，从i到j，中间可能有多个值，一一遍历就是暴力解法，优化当然是更快找到j，那么如何迅速定位到j呢？

或者换个说法，i到j中间的多个可能性，有哪些是必然不可能的，可以直接过滤？答案是没得😛。因为如果从[i, n)区间提取出单调递增的部分，可以使用二分，但这个题应该用不着。那这个思路还有什么优势呢？我们一一遍历到j，然后得到这个i的最优解，然后i++，然后你还是得遍历过去，因为单调递增队列本身不能有更多的改动，最多把队列头比i小的pop出来。那换个角度，递增队列是不是能从后往前看，当某个靠后的j满足了条件，你能把它怎么样？你不能动它，因为接下来的i（更大的i）和j一起也能满足条件，这个窗口长度肯定比现在的短，你删了这个j，就丢掉了解。

那么就应该考虑换个顺序，以当前j为标杆，去找前面的i，它有个什么好处呢？当你从前往后找i，某个i满足条件后，你可以大胆删了它，因为j会++，当前的i充分发挥价值（用来更新window len）后，就不需要了，后面的j和这个i组合是无意义的。

P.S. 我想到了以j找i，但又想的是从j-1往0这个方向找，想想看，跟以i找j没什么差别，它们跟暴力解法比，完全不是稳定降低复杂度，甚至可以劣到没区别。（于是Python实现也完美超时了）

总的看“前缀和+单调队列”，大概的最好状态是每次O(1)，然后n个j，所以O(n)，比如，每次看队列头d[0]的preSum都很大，不能让preSum[j]-preSum[d[0]]>=k，单调递增的d后面的元素也不能满足要求了，所以每次就立马完成。最差的状态可能是某次j，从0到j遍历，每次都符合条件，都要pop并更新window len。但很明显，队列最多跟n一样长，而且pop出去了又不会回来，所以次数最多n次，并不会膨胀。所以总的复杂度还是O(n)的。



这道题还是很难的，难在拼出正确的方法，知道方法后，代码实现不难。

#### [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)

突如其来的一道复习题。while-if即可，虽然和while-while实测没什么区别，大概是case的原因。

#### [Replace the Substring for Balanced String (medium) -- LeetCode](https://leetcode.com/problems/replace-the-substring-for-balanced-string/)

题目是说要替换的字符包含在一个子串里，要求子串最短，子串符合某个条件即可，那就很适合套用滑动窗口模板了。

#### [Count Number of Nice Subarrays (medium) -- LeetCode](https://leetcode.com/problems/count-number-of-nice-subarrays/)

这题一看就不适合立马套用模板，扩展和收缩求最长最短很有效，但这里没有用处。举例说明，当我们找到一个窗口恰好有k个奇数，此时可以滑动窗口么？当然不能。所以放弃吧。然后考虑到奇数是核心，先找到k个奇数的最小可能，它的左右两边如果分别有a个偶数和b个偶数，那么这里就有很多个子串可能，1+a+b+a*b。而找奇数，可以直接抽出奇数，这样奇数数组里每k个就是一个base，延展下两边的偶数。既没有重复也不会漏算。



滑动窗口完结撒花🎉

## Pattern 2 : Two Pointers 

### [Pair with Target Sum (easy) -- LeetCode](https://leetcode.com/problems/two-sum/)

这题第一感觉是map或者排序后二分查找，双指针是个什么操作？

双指针自然得用在排序后的数组上，头尾各一个，[left]+[right] == target返回，>target right--，<target left++。确实也比二分好写，如果不能用库函数二分，双指针编码不容易错，也是个很好的选择。

就是答案要求返回索引，排序后的索引怎么查也是个问题。代价还是不小的。

### [Remove Duplicates (easy) -- LeetCode](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

原地当然是要原地替换了，那么应该被替换的地方需要一个指针，理应替换到前面去的需要一个指针。

比如，00111，第二位的0会被第三位1覆盖，那么第三位理论上是个空缺位，但编码上，不用管一位，因为一视同仁的话，序列等于是01111，第三位是1，也不并妨碍接着删除1。所以可以不用特别处理，代码会写的很简单。

### [Squaring a Sorted Array (easy) -- LeetCode](https://leetcode.com/problems/squares-of-a-sorted-array/)

非递增序列，类似概念还有：非递减，单调递增，单调递减。。。

单调就是相邻两个数不会相等，非xx自然是可能相等的。

用高数教材的定义，当x1<x2时，都有f(x1)<f(x2)，f(x)就是递增函数，increasing function。其实“单调”这个词有些干扰。

increasing就是上升，不存在横着走，non-decreasing就是不下降，那自然可能横着走走（也就是，可能相邻几个数相等）。剩下两个同理。



题目本身很简单，结果是从小到大，但是绝对值最小的数不好找，所以反其道而行之，找大的然后放在尾部就行了。

### [Triplet Sum to Zero (medium) -- LeetCode](https://leetcode.com/problems/3sum)

在此题必然用双指针的提示下，想到了固定1个数，然后变成两数之和问题，用双指针处理这一子问题。

但这个题的输出很多限制，需要排除的东西很多，示例

```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]
```

基本包含所有可能，完成这个test就能AC。

编码上，可以避开很多不必要计算，始终记住，假设当前固定的数是nu ms[i]=a，双指针肯定是i<left<right。不要把left始终设为从0开始，没必要。

### [Triplet Sum Close to Target (medium) -- LeetCode](https://leetcode.com/problems/3sum-closest/)

其实比上一题简单，因为不用担心解重复的问题。注意的是result一开始设置一个极大值以便更新，别写做10^4了，这个python里会解释为14。表示次方使用两个星号。

### [Triplets with Smaller Sum (medium, google) -- LintCode](https://www.lintcode.com/problem/3sum-smaller/description)

不是直接套用模板能搞定的了，好好理解下面这个example。

```
Input:  nums = [-2,0,1,3], target = 2
Output: 2
Explanation:
Because there are two triplets which sums are less than 2:
[-2, 0, 1]
[-2, 0, 3]
```

应该是能O(n^2)解决的。

稍微暴力点，但容易想到的解法是for i, for j in [i+1, n)，这一层，可能有多个k可以是三数之和<target。这里可以考虑二分找，lower bound还是upper bound，不用细想，但可以知道二分方法能做到。但是复杂度就是n^2 * logn，肯定有优化点。忘记二分，如果我们顺序找k，从左或者从右都行，但考虑到“如果这一轮j找到了，j++后k是不需要跳回最右边的，k可以就从当前位置开始”，这个应该很好懂。所以如果k是从右往左找，代码就可以写的很简洁。而且明显降低了复杂性。

再来思考下复杂度是多少，由于固定i后，j和k又是最多只会遍历[i+1,n)的部分，所以总的是O(n^2)。

### [Subarrays with Product Less than a Target (medium) *](https://leetcode-cn.com/problems/subarray-product-less-than-k/)

要求输出所有subarrays，也有要求输出subarrays个数。输出要求都不太难。

套用双指针或者说是滑动窗口模板时，需要考虑到乘积的写法。尤其是当某一个数本身就>=k时，窗口应该收缩成0。这个用双while写起来就很怪，所以我是在最开头就判断下，如果是就重新设置那一堆变量，进入下一次循环。官方代码使用for-while把这一情况包含了，不用单独判断。这里可以再思考下，官方的是不是正确，当nums内有数字1或者k==1时又会不会有奇怪的问题出现？

for-while的写法，就是可能会出现while结束后，start == end + 1，此时product == 1，因为没有取到任何一个数。接着会马上进入下一次外层for，end++后就会是start == end，即想选择[end]这一个数作为subarrary，然后接着while判断。这一整段没毛病。

至于nums内有数字1，完全不影响计算。而k==1，也不会有问题，走一遍逻辑就知道了，k==1时无论数组长什么样，结果都是0个，不存在<1的正整数，更不会有乘积<1了。

官方还有二分查找法，乘积[0, i]是非递减的，确实有道理。而且由于乘积可能很大，还用对数，很强势，看一看。

- [ ] 取对数后很难是正整数，那么必然有精度损失，不影响正确性？

### [Problem Challenge 1 - Quadruple Sum to Target (medium) *](https://leetcode-cn.com/problems/4sum/)

四数之和，数组是整数但可零可负。

在已知此题可用双指针的情况下，自然去思考如何利用双指针，但双指针只能用于两数和问题（指定这两个数和为某定值，找出两数的所有可能解），那四数还有两个数怎么办，最简单的想法就是把这两个数所有可能列出来。于是平方复杂度列举出所有a和b的可能，对每一对ab，用线性复杂度的双指针找出c和d的所有可能解。大约就是三次方的复杂度了（可能真实复杂度可以算的更精确点，但反正小于等于三次方复杂度，暂不纠结这个问题，简单看待）。

想到这里，三次方的复杂度让我有点虚。不过，转头想这个题如果是暴力解法就是四次方，如果是定三个数再利用有序二分找第四个数，就是n^3*logn，n^3的复杂度好像瞬间就容易接受了。

题目有个麻烦点是不能有重复的四元组，如果不通过某种方式设置set（对四元组去重）的话，就需要跳过那些重复的组合。也能写出来，就是代码有点丑，也需要case来调试。不能一气呵成。

这个比较顺畅的思路是：

1. 首先思考第一个和第二个数，简写为a和b，它们都是for循环，比较类似。

   假设a固定，看b的移动。b如果右移还是同样的值，“右移之后的解集”只会小于等于“右移前”的（cd因为b的右移，可能性变少了），可以想到，解集不会增加任何可能性，反而只可能减少，b的取值却又是一模一样，那么右移后的abcd组合，在右移前遍历中都考虑完了，自然完全没必要重复遍历，所以b要一直跳到和前一个值不一样的地方，才需要进行遍历求解。

   而a的情况也一样，a的右移只可能使得bcd组合减少，没有新花样，a右移前后的解，值都一样，右移后的完全可以被跳过。所以a也是要跳到和前一个值不一样的地方。

2. 再思考cd两个值，如何过滤重复的解。

   c+d不等于想要的值时，方向很明确，只会left或者right某一个移动。如果c移动到下一个值还是不变，那还是c+d不等于期望值。所以不等于的情况不用过滤。而c+d==期望值时，如果只有left右移，那么肯定要跳过重复的值。但如果是right那边重复，只left跳过重复值，能否解决问题？答案是可以。举例说明，当left这边序列是11111xxx时，1+d==期望值，那么left一直移动到最后一个1，下一个数就不是1的时候，abcd这个组合可以成为一个合法解，然后left++，那么下一次while时就是大于1的数+没有动的d，那肯定是大于期望值，right那边就会左移，遇到重复的right还是左移，并不需要考虑过滤。

   当然，只做right的重复过滤也可以。没必要做两边的。当然两边都跳过也行，但少写代码少错😄。



再思考下几数和问题，两数和就是双指针，线性复杂度，三数和就是指定一个数再解决两数和，平方复杂度，四数和三次方复杂度。

### [Problem Challenge 2 - Comparing Strings containing Backspaces (medium)](https://leetcode-cn.com/problems/backspace-string-compare/)



- Problem Challenge 3 - Minimum Window Sort (medium) *
