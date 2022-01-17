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

## 1. Pattern: Sliding Window

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

## 2. Pattern: Two Pointers

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

顺序找规律怎么也找不到时，一定要记得反向试试。数组题一定要记得反向可能有奇效！！！

此题没有什么巧妙写法，写出来的代码有点长，不过容易调试成功。

### Problem Challenge 3 - Minimum Window Sort (medium) *

```
Given an array, find the length of the smallest subarray in it which when sorted will sort the whole array.

Example 1:

Input: [1, 2, 5, 3, 7, 10, 9, 12]
Output: 5
Explanation: We need to sort only the subarray [5, 3, 7, 10, 9] to make the whole array sorted

Example 2:

Input: [1, 3, 2, 0, -1, 7, 10]
Output: 5
Explanation: We need to sort only the subarray [1, 3, 2, 0, -1] to make the whole array sorted

Example 3:

Input: [1, 2, 3]
Output: 0
Explanation: The array is already sorted

Example 4:

Input: [3, 2, 1]
Output: 3
Explanation: The whole array needs to be sorted.
```

这个题没找到OJ地址。思考的时候想到了拐点，不过后续思考没跟上。

首先，找到了subarray之后，这个subarray一旦能排好序，整个array就有序了。那么，非subarray的值就应该是它们应该在的位置。

Example1中1和2在应该在的位置，就很容易想到指针找到第一个比右边还大的值（也就是拐点），也就是5。但Example2直接打脸，1本就不在该在的位置。

到这儿就没继续沿着这个思路继续下去了。然后就没找到思路了。。。

找到左右两边的拐点后，拐点之间的这个区间必然不是都在正确位置上，所以这个区间肯定在subarray里面。但Example2也提示了，找到了拐点区间[3,2,0,-1]，可以看到前面的1也是错的，为什么呢？因为1排序后也不应该站在它现在的位置，也就是说subarray应该还需要扩张。

扩张的依据是什么？这个元素是否在当前区间的最大最小值之间。因为如果在之间，全局排序后，这个元素和当前区间的位置都是要变的。画一下折线图，纵坐标表示值的大小，横坐标是元素位置，就可以很直观的看到，区间的扩张是简单的，因为它只需要往外1个1个check就行了，没有什么复杂规则。

- [ ] code

## 3. Pattern: Fast & Slow pointers

### [LinkedList Cycle (easy)](https://leetcode-cn.com/problems/linked-list-cycle/)

题目描述很反人类，但其实就是给你一个链表（只有head指针），让你判断是否有环。进阶是只使用O(1)的空间，也就是常量空间。

有环的链表肯定会访问到重复的节点，环内有1个或以上的节点。搞一个空间存已经访问过的节点，查到之前访问过就能判断了，但空间最坏能到O(n)。常量空间，自然要用快慢指针。

推理一下，快慢指针什么时候能证明链表有环？

猜想肯定是快慢指针指向同一个节点时，但怀疑是否存在“有环链表下快慢指针也不会同时指向同一个节点”。画画图，列一下方程式，假设链表入环前有s个节点，环上有c个节点，可以知道，当走整数倍c（2倍起，同时还得>s）的时候快慢指针是能遇到的，这个值不会不存在，还会多次相遇。所以不用担心快慢节点会永远遇不到。

编码上很简单了，记住init时fast，slow都指向head，然后while内先走再check。

### [Middle of the LinkedList (easy)](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

很简单的题，把两个case都手动推理一遍，就知道了。hint：fast先走，提前退出，slow就不用走了。

### [Start of LinkedList Cycle (medium)](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

这个题猜得到是需要快慢指针相遇后再加点什么操作的，但是推理容易卡壳。主要还是对快慢指针相遇的情况理解的不够。

快慢指针相遇很好写，node地址相等就是了。但相遇时，快慢指针并不是可能走了无数圈。这里有一个点必须记住，就是“慢指针入环后，和快指针相遇时，慢指针在环上是不会走超过一圈的”。也就是慢指针入环后继续走，一圈以内必定碰到快指针。

这个知识点，我以前应该用方程式推过，也不记得推理有没有漏洞了，这次做题还没推出来。所以这里最直观也正确的解法是数学归纳法。我们讲slow入环后和fast相遇，那么最快相的情况，当然是slow和fast直接在入环点直接相遇，0步。再麻烦点，就是fast在slow的前面（此刻的位置是fast.next==slow），这样两个相邻的节点，只需1步就能相遇。假设fast更远一点，即fast.next.next==slow，就需要2步，以此类推，入环时刻，fast和slow的距离（fast的n个next==slow）假设为n，就需要n步相遇。但fast和slow之间最大距离就是fast在slow的前一个，假设环里有k个节点，fast和slow的最大距离就是k-1，所以最多k-1步，fast和slow必定相遇。所以slow不可能在相遇前走几圈。

知道slow在一圈内就会和fast相遇，其中几个关键长度如下图所示。那么相遇时slow的公式应该写为，slow=a+b，fast=a+b+n(b+c)。

![示意图](https://assets.leetcode-cn.com/solution-static/142/142_fig1.png)

(leetcode图源，可能会挂，自行戳网址)

再回到题目本身，当两指针在紫色点上相遇时，代入化简一下已有式子，得a+b=n(b+c)，即a=c+(n-1)(b+c)，为什么化简成这样？不是因为这样a可以通过等式右边得到，右边这式子也没法继续求。而是因为，右边写成这样，b+c是一整圈，也就是说，如果一个指针从链表头开始走起，另一个指针从紫色点走了c长度又走了几个完整圈，它们会在入环口相遇，另一个指针走几圈完全不用在意。

但注意，这个式子可能有坑，思考下有没有可能a很小，c很大？

n必然>=1，因为fast肯定走的多点，不然fast=a+b+n(b+c)就不对了。那么，a最小就等于c，不可能比c小。这个事情挺有趣的。没有想到一个很通俗易懂的表达，但数学证明了也就证明了吧。

### [Happy Number (medium)](https://leetcode-cn.com/problems/happy-number/)

这题现在是简单😂

题目没有读仔细，所以第一时间没能察觉到和快慢指针有什么联系。题目表述为“也可能是 **无限循环** 但始终变不到 1”，我简单地想到了不能一直计算，但忘记了无限循环不是每一次结果都不一样，它只是每次结果都是不是1，不是说每次结果都是从未出现的数字。

接着，就排除了用定义一直计算这种思路，变成了想利用数学之类的其他方法来解决。虽然也推理出了平方和计算中，不能爆出非常大的值，最大32bit的数，十进制也就是13位，就算13位都是9，一次平方和计算下来，也不过1053。而1053的平方和也挺小的，1053以内能算出较大平方和的也就是999。而999再算一步也就是243。

可以明显感觉到，不可能出现一个小的数字经过多次计算膨胀的很大，只可能很大的初始数一下子变得很小。当然像case1中19经过计算会变大，但这都是一定范围，最大不可能超过1053。

然而，从这里开始，我就期待用动态规划一类的方式，类似斐波那契数列，提前算好所有快乐数。所以我认为，可以从1，10，100，1000反推平方和等于它们的数，但是反推的链路有点长，说不定反推也会不能停止，毕竟这个思路不太对劲，可能有不少漏洞。

总结还是应该利用平方和结果范围有限这一点，也就是抽象成链表的有环判定。当然，因为平方和都是即时算的，不像链表问题提前准备好链表。简单的方案就是直接保存之前计算出的结果，也就是不用快慢指针，而是用是否已存在来判断。而如果还是想用快慢指针，可以def一个next函数，考虑到节约空间，next函数只管计算不用缓存。快指针两次next，慢指针一次next就好了。

纯数学的角度其实也可以继续推下去。但需要更仔细的范围研究。所以，回顾之前的推理，最大的数字13个9，也会瞬间收缩到1053，再归纳总结，12个9收缩到。。。4个9收缩到324，这里特别的来了，999到243。为什么说999到243特别，因为这里位数不降了，之前的那些大数都会收缩位数，变得更小，999虽然会经过一次计算变小，但位数不会收。所以一旦计算结果收缩到3位数，接下来的所有平方和结果都是三位数以内，不可能膨胀，且跌入三位数后（三位数后再计算一次），必然困在243以下。而为什么就243了，有没有可能实际数字更小？简单举例99就知道，它能变大到162，也就是涨到三位数，但不可能从三位数继续变大。所以，计算进入243以内后，可能会振荡，比如2位数蹦到3位数，但不可能比243更大，也就是出不去了。

那么，假设计算结果小于等于243时，我们能直接给出结论，是不是快乐数。就不用进入无限的循环了。所以问题变成243以内的数字，哪些是快乐数？

而这样的问题缩减，有什么好处呢？好处在于，243个数字，最大值也就是243，大可以暴力算加是否已存在判定，最大空间复杂度也就到243，比起给一个数就缓存，空间占用要小一些。（其实应该也小不了多少，毕竟收缩很快）但把<=243的数字直接打成表，对于频繁查快乐数的情况，就会节约时间的多。oj时间上节省的会比较明显。如果只是一次快乐数判定，当然打表反而还浪费时间。

### [Problem Challenge 1 - Palindrome LinkedList (medium)](https://leetcode-cn.com/problems/palindrome-linked-list/)

难度简单。

很明显快慢指针能够找到链表的中部。而中部开始向左向右比对，就能判定是否回文。右半边很顺畅，不用改动，关键在于左半部分。但左半部分再快慢指针途中是被遍历访问过的。这其实也就提示了，如果修改链表，就能降低空间复杂度。题目进阶就是希望用O(1)空间，那大概率就是原地修改链表的左半部分。

很直接的就能想到是反向指。写写画画，不难得到简单的代码。得到了链表中部的指针后，当然是往两边走，逐个对比。但注意，奇数回文串和偶数回文串比较的起点是不一样的。我粗心的只想到了偶数回文，跑测试才发现奇数回文跑不过。

P.S.

这个题还有个递归解法，我不太擅长递归，经常起手就是迭代，所以这里值得再学习下。首先，递归能给我们什么？假设我们只是简单的print val，那么递归就能从尾到头，逆序读出每个节点的值。假设我们递归到底了，现在能读到尾部节点，这个时候我们应该拿头部节点和它进行比较，接着递归会读倒数第二个，而这时又应该拿顺数第二个节点比较。可以看到，从头部开始读也可以用一个指针来解决，但这个指针得是外部的变量，放在递归函数里面太困难了。再思考奇偶情况，奇数节点两指针地址会指向同一个节点，容易判断，偶数情况本来以为会麻烦点，但还好，因为两个指针指向的节点是相邻的，所以只要next判断一下就好了。leetcode官方解更飘逸，全遍历，不用提前返回，那对栈空间要求更高了。不大实用，但可以多熟悉下递归。论简单，还是递归这个写法代码少。

### Problem Challenge 2 - Rearrange a LinkedList (medium)

```
Given the head of a Singly LinkedList, write a method to modify the LinkedList such that the nodes from the second half of the LinkedList are inserted alternately to the nodes from the first half in reverse order. So if the LinkedList has nodes 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> null, your method should return 1 -> 6 -> 2 -> 5 -> 3 -> 4 -> null.

Your algorithm should not use any extra space and the input LinkedList should be modified in-place.

Example 1:

Input: 2 -> 4 -> 6 -> 8 -> 10 -> 12 -> null
Output: 2 -> 12 -> 4 -> 10 -> 6 -> 8 -> null

Example 2:

Input: 2 -> 4 -> 6 -> 8 -> 10 -> null
Output: 2 -> 10 -> 4 -> 8 -> 6 -> null
```

跟challenge1类似，尝试了一下递归的写法，比较容易写，就是先把偶数链表搞定，再补一下奇数情况的判断就行了。但是注意，debug时别打印链表，修改中途的临时链表很可能是有环的，打印也打不出来。当然可以限制下打印的node个数，调试时还是可以有的。

如果不用递归写法，这个题还是和challenge1一样，可以把后半部分链表原地反转。不多赘述。

### [Problem Challenge 3 - Cycle in a Circular Array (hard)](https://leetcode-cn.com/problems/circular-array-loop/)

题目有点难读，但解析下来，题目的意思是，首先有个环有n个节点，节点里的值表示下一步会向前or向后跳几个节点，可以理解为**在做链表的next链接**。这样下来很可能会出现环状的链表，而且题目定义的链表更狭窄点，k=1（一个节点自环）的不算，不算一会儿前进一会儿后退的。

根据“链表next”和“判断是否循环”，就知道可以用快慢指针。但快慢指针会测到自环，所以即使判断有环，还得再一次确认是否是k=1自环的情况。

而“不算一会儿前进一会儿后退”这一点很重要，题目的说法是循环的下标序列不是全正就是全负。那么指针跑的时候去要看自己现在所处的位置i，这个位置指定的步数，也就是nums[i]如果和之前的nums[...]符号不同，就可以直接判断为False。

那么，还剩两个问题。

1. 快慢指针没有null作为退出条件了，不知道能不能结束。这里可以考虑防御性编程，加个一定限制。但其实不需要，原因是，如果一个nums值全为正，选一个元素来看，它必然得指向某个元素，这个元素如果是它自己，就会发现一个自环；如果不是，它势必要指向一个新元素（我们还未访问的）。但访问得一直继续，如果每次都不是循环（快慢指针查不到的），不会停止，一直访问下去，那么元素迟早被访问完，那下一步会去哪儿呢？所以必然会有快慢指针能够查到的环，虽然自环不能算本题定义的“循环”。
   1. 总结一下，就是，除了不是全正or全负会提前退出，快慢指针是必然会相遇的。自信点，不用防御编程。
2. 此题不是简单的从数组头开始。拿个示例画一下，也能发现，从某个下标开始跑快慢指针，结果都是不一样的。所以理论上，每个数组元素都是可能的循环的开始。

总结到这里，突然想到一个问题，既然只有全正全负会陷入while loop不能提前退出，而全正全负又一定能有广义的环。那么完全不用快慢指针，只需要把每个点当作可能的入环口，也就是从这个点开始我能转回这个点，就找到了一个广义环。并不是非得快慢指针。只是这里有个坑，那就是这个点只是可能的入环口，它也有可能是类似有环链表的直线部分，走了几步才入环。所以得有个限制退出while步进，不然会死循环出不去。又由前面整理的结果，想要全正or全负没有环是不可能的，最大的环也就到整个数组的长度（当然，可能链表箭头走的很骚，不是说只能沿着数组+1/-1步这么走，但长度是不能再长了）。

这个思路比快慢指针代码上简单点，时间复杂度上却不是变少，因为这个思路走满环，最坏时能达到数组长度，O(n)，快慢指针中fast指针和slow指针第一次相遇时slow也没有走满环，fast多一倍步数，也没差多少。所以这个思路也不会带来质变，聊胜于无。

## 4. Pattern: Merge Intervals

### [Merge Intervals (medium)](https://leetcode-cn.com/problems/merge-intervals/)

挺简单的题，尤其是我对这种merge interval的题有一个印象，就是“反着比正着简单”。所以很容易就想到了，这题应该反向来看intervals，所以遍历顺序是end大的到end小的，合并interval或者即时push interval到result里，都很简单，不用多注意什么。

### [Insert Interval (medium) *](https://leetcode-cn.com/problems/insert-interval/)

这个题目纯粹是恶心人，没啥别的作用了。由于每个interval都有两个数表示，又得两个interval之间比较，绕的很晕。

大概逻辑很好想，由于加入new interval（后面简称new），可能new和intervals内的多个interval（后面简称i）有重叠，所以可能会消除多个i，为了代码简洁，肯定是新建一个list，符合条件的才插入，这个逻辑比较好。

又回到new和多个i的合并上，重叠就需要合并，合并后的new'可能还是不应该插入，毕竟可能多个i都需要跟new合并，所以需要一个tmp interval。很容易发现，直接用new来做这个tmp interval正好。

本来思路到这儿还是很清晰的，但是由于对“重叠”的定义没先弄清楚，所以写出了漏洞百出的算法。这一点需要吸取教训。算法如果一开始用数学很难表示，就应该先用形容，能把算法定义清晰了，再翻译为数学。不要总想一步到位。

而“重叠”定义，最简单的办法就是画图，画两个interval的相对位置关系，可以看到，分四种情况，a的右边跟b重叠，a的左边跟b重叠，a完全在b内，a完全包容b。当然数学上，由于与或非关系，可以把前三个写为一个判断式，但第四个无法合并，很容易忘记这一种情况，要细心。

当然，可以反向来看，那就是“不重叠”的非集就是“重叠”。而“不重叠”的判定更简单（我一开始是这么想的，但当时对“重叠”情况的处理很混乱，所以换了思路）。如果再次做此题，正向反向都容易想到，没有特别的坑。

当扩展后的new和当前i不再重合时，需要把new和i都加入result里。这里需要一个布尔量表示new有没有已经被加入，这个步骤没办法写的更优雅。

coding时，还可以注意，我在妄想一步到位时，写满了[0]，[1]。。。把自己都给绕进去了。python是可以`for left, right in intervals`这么写的，所以别折磨自己，python is beautiful!

### [Intervals Intersection (medium)](https://leetcode-cn.com/problems/interval-list-intersections/)

拟定一下算法流程，照着流程过一遍example，就没什么坑了。没什么巧思，一个一个if-else保证正确就行了。

### Conflicting Appointments (medium)

```
Problem Statement
Given an array of intervals representing ‘N’ appointments, find out if a person can attend all the appointments.

Example 1:

Appointments: [[1,4], [2,5], [7,9]]
Output: false
Explanation: Since [1,4] and [2,5] overlap, a person cannot attend both of these appointments.

Example 2:

Appointments: [[6,7], [2,4], [8,12]]
Output: true
Explanation: None of the appointments overlap, therefore a person can attend all of them.

Example 3:

Appointments: [[4,5], [2,3], [3,6]]
Output: false
Explanation: Since [4,5] and [3,6] overlap, a person cannot attend both of these appointments.
```

最简单的区间问题，排序完了，遍历就行了。排序是正着还是反着都行。反正拍完序，只需要看相邻两个区间有没有相交。

### Problem Challenge 1 - Minimum Meeting Rooms (hard)

https://leetcode-cn.com/problems/meeting-rooms-ii/ plus

```
Given a list of intervals representing the start and end time of ‘N’ meetings,
find the minimum number of rooms required to hold all the meetings.

Example 1:

Meetings: [[1,4], [2,5], [7,9]]
Output: 2
Explanation: Since [1,4] and [2,5] overlap, we need two rooms to hold these two meetings. [7,9] can
occur in any of the two rooms later.

Example 2:

Meetings: [[6,7], [2,4], [8,12]]
Output: 1
Explanation: None of the meetings overlap, therefore we only need one room to hold all meetings.

Example 3:

Meetings: [[1,4], [2,3], [3,6]]
Output:2
Explanation: Since [1,4] overlaps with the other two meetings [2,3] and [3,6], we need two rooms to
hold all the meetings.

Example 4:

Meetings: [[4,5], [2,3], [2,4], [3,5]]
Output: 2
Explanation: We will need one room for [2,3] and [3,5], and another room for [2,4] and [4,5].
```

因为所有intervals都得被满足，所以贪心策略可能可以得到最优解？

简单设想一下，已经排序好的intervals，一个一个尽量排，重叠了就新增一个room。

假设，此时已经用贪心策略得到了2个room，r1和r2，假设r1.end <= r2.end，这时又想加入一个interval，它的start如果比两个end都小，那必然得出第三个room，这个不可能缩减。而如果start比一个小比一个大，那肯定是放入，但放1还是2？我们先比较，肯定是选r1（end最小的room）来比，如果end最小都放不下新的itv，就新增room。如果r1能放下新的itv，为了代码简洁，也应该直接放下了。（假设r2也放得下新itv，那么下一个itv也能放到r2，不会新建room；假设r2放不下新的itv，那就更不可能放r2了，至于下一个itv，就看更新后的r1r2能否满足了。）

——这一部分用数学推理下。

但显然，room不可能只限2个，所以可以考虑一个排序的容器，保存room的end。容器需要支持重复key，因为多个room的end可能数字一样。容器只需要取的出最小的end，之后end可能更新变大再插回容器，其他位置不用管。所以用优先队列最符合要求，而且python没有multiset，想不用优先队列都不行。

代码写起来还是很简单，rooms只会“堆顶被pop再更新push”和“pop新的room”两种情况。不过题解里有一个很骚的操作，就是会把rooms里end<=itv.start的元素都pop掉。怎么理解这个东西？

首先按我原本的设计，rooms的len只会不变和变大，不仅如此，当前itv的end是必然会进优先队列的，区别只在于堆顶会不会pop（也可以理解为itv是会进入优先队列的，因为这里需要考虑itv.start了）。原本设计里优先队列“时刻对应”rooms的已有排列。但前面的思考里，也体现出了，rooms内可能多个room的end都<=itv.start，这部分rooms不会对当前itv和之后的itv产生影响。无法产生影响，就可以直接当做“不存在”。“不存在”，所以可以从heapq里剔除掉，当前itv必然加入heapq（itv会不会影响，要看下一个itv的比较），当然，这时的heapq可能比“当前实际rooms len”小，所以用max来追踪heapq的len最长的时候。

（题解思路还不够清晰，有空再思考下）

### Problem Challenge 2 - Maximum CPU Load (hard)

```
We are given a list of Jobs. Each job has a Start time, an End time, and a CPU load when it is running.
Our goal is to find the maximum CPU load at any time if all the jobs are running on the same machine.

Example 1:

Jobs: [[1,4,3], [2,5,4], [7,9,6]]
Output: 7
Explanation: Since [1,4,3] and [2,5,4] overlap, their maximum CPU load (3+4=7) will be when both the
jobs are running at the same time i.e., during the time interval (2,4).

Example 2:

Jobs: [[6,7,10], [2,4,11], [8,12,15]]
Output: 15
Explanation: None of the jobs overlap, therefore we will take the maximum load of any job which is 15.

Example 3:

Jobs: [[1,4,2], [2,4,1], [3,6,5]]
Output: 8
Explanation: Maximum CPU load will be 8 as all jobs overlap during the time interval [3,4].
```

很容易发现这个题和上一题安排会议，抽象下来是一样的。最容易想到的push/poppush的方法，但由于题目是求max load，所以要尝试改造，然后发现不行。因为这种方法没办法跟踪当前load sum。举例说明，假设itv已经排好序，start小的排前面，start相等时end小的排前面，如果itv1和itv2重合一部分，itv3和itv2重合，但是它并不与itv1重合，在看itv2和3重合的那一段时，load应该是2和3的load之和；而如果itv3和itv1也重合了，这时就应该是三个itv的load之和。但push/poppush是区分不了的，因为它只和heap top比较了一下，摸不到别的itv。

再想到while pop这个方法，虽然在上一题里它很不容易理解，但这道题反而该使用这个方法。核心在于while pop出和当前itv无关的itv，并相应减掉它们的load，就能够实时追踪到某一个时刻的load之和了。

- [ ] 再理解下上一题和while pop方法。

### Problem Challenge 3 - Employee Free Time (hard)

https://leetcode-cn.com/problems/employee-free-time/ plus

```
For ‘K’ employees, we are given a list of intervals representing the working hours of each employee.
Our goal is to find out if there is a free interval that is common to all employees.
You can assume that each list of employee working hours is sorted on the start time.

Example 1:

Input: Employee Working Hours=[[[1,3], [5,6]], [[2,3], [6,8]]]
Output: [3,5]
Explanation: Both the employess are free between [3,5].

Example 2:

Input: Employee Working Hours=[[[1,3], [9,12]], [[2,4]], [[6,8]]]
Output: [4,6], [8,9]
Explanation: All employess are free between [4,6] and [8,9].

Example 3:

Input: Employee Working Hours=[[[1,3]], [[2,4]], [[3,5], [7,9]]]
Output: [5,7]
Explanation: All employess are free between [5,7].
```

这题本质是一种反向，题目提过工作时间，让你求非工作的某种时间，所以可以用交集并集差集来理解这个题，方便快速的分辨多个解法的复杂度。当然，最佳算法可能比这种逻辑运算的计算量更小，但不熟悉这类题目的情况下，越抽象的思考越容易做。

可以看到k个list，每个list都多个interval，所有的interval的并集，就是所有员工的工作时间，这个集合的反，就是所有的空闲区间（空闲区间显然也可以是多个）。

题目说明了，k个list，list内的区间都是有序的。这很像k路归并排序。按理可以比所有inteval混在一起排序更高效。那我们先mark这个归并思路，放在一边。（有序基本就是在提醒我们存在“利用有序”的优化算法）

#### 简单思路

假设经过某种算法，已经计算出了所有working hours的并集，这个并集当然可能是多个区间，不一定是一个连续区间。求反只需要遍历这一堆区间就行了。

再来考虑，这个“某种算法”求并集，怎么做。最简单的肯定是全排序。每个区间先比较start，小的排在前面，end随意。这里很可能会出现连续几个区间是有重合部分的，所以，求并集，还得遍历一边，重组一下区间，得到并集（不重合的区间组成的数组）。然后还得再遍历求反。

简单一想，也知道，最后一次遍历多余了。

所以，可以立马优化为，遍历已排序的区间时就求空闲区间。

可以再思考一下，我先每个list先求反，得到每个员工的空闲时间，所有的空闲时间的交集也是最终答案。但空闲时间区间个数不见得比工作区间个数少多少（应该是，对每个list，空闲区间=工作区间-1）。并没有把复杂降低。而且求交集比求并集复杂，因为空闲区间没办法全部一起求交集。想象一下，假设5个员工，有1个在[0,t]之间都不空闲，但是另外4个却可以在[0,t]这个时间内存在交集，但是这个交集是不能要的，因为那一个人不空闲。区间全混在一起时，根本无法判断。所以还是求并集吧。

#### 归并思路

每个list“有序“，自然是提醒我们要充分利用这个特性。全排序肯定不够用。而list内interval有序，很显然可以做类似归并排序的操作。

归并思路，k个list也就是k路，每次都从k个list的头上选出最小的。这k个区间比还呆在list里的区间的start都要小，在k个中选出来的start最小的区间，也就是全局start最小的区间。所以，当我们将k个中选start最小看作一个模块时，我们从这个模块中拿出“剩余区间”中start最小那个区间。一个一个拿，就是总能按顺序拿出区间了。

核心是这个模块如何实现，即考虑“k个中选最小的”怎么高效，归并算法如果只有两路，当然if-else都能行，但k路肯定是不可能一个一个比的，很明显可以利用最小堆，每次的top就是最小的，top取出来后，top所在的list就应该补充一个区间进入最小堆。python中也就是使用heapq，可以极快的写完代码。

##### 堆

首先是二叉堆，巩固下：

我们希望有一个数据结构，它可以迅速pop出最小的，push一个新的数也希望是对数时间里完成，这样的时间复杂度才有价值，不然简单的有序数组pop/push都是O(n)就足够了，还简单。

这个理想的数据结构，应该考虑树形，毕竟对数时间。

再说什么树，因为没有什么限定条件，二叉树就是最简单的。然后再想，n个数去构建二叉树，如果n不是2^k-1，那这个二叉树就不满，不满也不能乱搞，因为对数时间，你当然希望这个树足够矮小，不然最坏情况又是线性了。所以平衡二叉树是理想形态。

而平衡二叉树并不必须用链表作为节点，它可以用连续数组表示。这也是让代码更简单的一个优点。

再来说说二叉树的基本操作逻辑。

一个是pop，pop出堆顶后，树的根节点就空了，这个“树”就不是有效的树了，那得想个办法修好，办法就是把最尾部的元素移动到堆顶，这个时候不满足堆有序，那么就去让它有序，画画图也可以发现，只有堆顶也就是根节点不符合堆有序，那么调整根节点和它的两个子节点，比如它和左子节点交换，它们三个就堆有序了。但因为左子节点变了，它和它的两个子节点可能又乱了，就继续调整。一层一层往下，也叫做“下沉”。只会沿着某一条路径下沉到叶子，所以复杂度是logn，不会影响别的路径。

二是push，新增一个元素到已形成的堆，将新元素塞入树中间显然不合理，放在最后一个位置，就像是多给某个节点加了个子节点。然后，这个节点和它的一个或两个子节点，可能不满足堆有序，就需要调整。调整后，它作为子节点，可能又不能有序，于是就一直“上浮”，上浮到根节点就结束了。同样的，只会影响一条路径，也是O(n)。（如果当前堆已经满了呢？新加的节点，在逻辑上看，就是树多加了一层，这一层只有一个节点。说得通。）

再提一嘴python的heapq，优先队列一般底层结构就是二叉堆，具体二叉堆怎么实现都行，python这里也是使用数组，但是它的内部方法实现不是前面提到的算法，它更高效，可以仔细看下heapq的实现和注释，实名diss了前面的常规算法。但这个不影响时间复杂度，不可能比logn更小，只是时间优化。（heapq算法里的siftup,siftdown和上浮下沉就对不上了。有空可以学习下。）

而且heapq提供heapreplace，也就是把堆的pop和push两步放在一步完成，因为size恒定。你如果pop[0]，就把新的元素放在[0]就行，让它下沉，就pop了堆化一次，又push，加一个尾部元素，又堆化一次。

注意heapq.heapreplace是一定replace的，它不会管你想push的元素会不会比heap top更小。它一定输出heap的top，再把push值塞进heap里。不过我们这个题目，是可以保证push的大于等于heap top的，不会有问题。

如果没有这一点保证，你想做的就是push后pop，因为你想从heap和想push的新值一起看的集合中最小的值，可以使用headpq.heappushpop。不过，这个自己写也行，逻辑不复杂，就是先peek一下堆顶，如果push的新值更小，就直接返回，如果不是，就把堆顶值取出，把新值放在堆顶，然后堆化。

##### 返回题目

具体到这个题目所需要的操作，首先要思考这个堆该如何运行，放多少数据，pop出来后又push多少进去，初步设计应该尽量分割步骤，目标是迅速写出正确的算法，后面再做优化。

所以初步设想是，k路数组，每个数组自己内部有序，但k个数组的头，我们是不知道谁最小的，所以把k个数组的头放进堆，我们就可以立马拿到最小的数，此时堆里剩k-1个数，这时候应该将pop出去的那一路的新头补充进堆，因为我们知道数组内的都比堆里的大，只有堆里的有资格比拼一下，所以堆大小在运行期间应该保持为k，不需要多放，多了增加复杂度，少了就比不出全局最小了。

##### 败者树

堆解法不是终点， 如果提到有没有别的思路，就肯定要说到败者树算法了。本质都是树，不会有数量级提升。但总有些区别。

###### 外排序todo

winner/loser tree很像B+树，非叶子节点是不存值的，所以，和B+树蕾丝，它们可以用在“外排序external sorting”上。

https://www.cise.ufl.edu/~sahni/cop5536/ 这个网站有很详细的exteral sorting的ppt，需要好好看看。（但是很难懂。。可以当作提纲）

针对winner/loser tree（统一可以叫做Tournament Tree）来讲，你需要知道，Tournament Tree可以利用于improve run generation, 也可以improve run merging。

解释下run generation。外排序不可能把所有的扔进内存里，所以只能先切分成小部分排序，然后做merge，这两步叫做，run generation和run merging。A run is a sorted squence of records。

improve run generation最简单的就是reduce the number of runs(也就是increase average run length)。但这也是有极限的，而且由于外排序在IO上很耗时，所以overlap IO也是一个优化方法。都在课件里，之后再慢慢看。

###### 返回tournament tree

败者树和胜者树是可以一起看的类似结构。胜/败者树都是类似B+树，只有叶子节点是真实值，非叶子节点是胜者/败者的标号，而且它是完全二叉树，不会有歧义。但胜者树和败者树**不是**单纯的一个非叶子节点记录胜者，一个非叶子节点记录败者，否则这两种就应该是一种树，只要compare函数求个反就行了。

胜者树

胜者树更简单，我们先看胜者树，比如小的是胜者，那么每个非叶节点都是它的子节点中更小的那一个。原理很简单，实现上要捋一捋。

可以把整个建树过程理解为打比赛，那比赛，肯定是从下往上打。最下层的非叶子结点先被填上值。按最简单的想法，胜者树用数组表示，每个元素是值（比较大小的值，不是索引）。这样子会有什么问题？当我们把胜者根节点拿出去，我们都不知道应该补上哪个list的元素。所以这个胜者树的节点，应该有索引，标记第i个list，这样就可以补充list[i].top进来。节点可以又保存值，又保存list标号，不过被拿来建树用的k个区间，本来也会放在一个地方，可以叫做ext数组，这样，胜者树节点只需要保存list标号就行了。ext[i]可以取值，list[i].top可以拿到补充用的区间。

而且注意，胜者树是所有参赛者都作为叶子节点的，比如，有3个叶子节点时怎么建树？有6个叶结点时，上一层3个节点，那它的上一层又怎么办？

注意，胜者树定义为complete binary tree，完全二叉树，它只有最底层可能少尾部一些节点，其他节点都是满的。所以，不会存在6个叶节点，上一层只有3个节点的情况，这棵树必须是最底层6个节点（理论上是8个），上一层4个节点（即使这一层最右的这个节点是没意义的，可以用max值，这样不影响比赛结果）。

解决胜者树的节点结构和树形，这两个问题，就可以写算法了。Ref 

https://gist.github.com/vagetablechicken/8a2fc94da27f127921c0b6dd08d7863f

败者树

而败者树呢，非叶节点是记录败者，但是往上送的参赛者是“胜者”（关键点）。

为什么这么做呢？假设胜者树的重新比赛，一个叶子节点被换了，那么它要和它的兄弟节点比。而败者树里，更新的节点只需要找它的parent，就能决定parent是否要更新，不用去访问兄弟节点。

这里粗略一想，觉得很莫名，也没有节约多少东西？

假设两个兄弟节点n1、n2，n1之前不是胜者，但现在比较一下发现n1是胜者，n1的parent就要被我更新为n1？而在败者树里面，我虽然只需要访问n1和n1的parent，不用访问n2，但这能节约多少？

这个问题，如果只看算法题目这种规模当然节约不了什么。但败者树通常用于外排序，外排序的数据规模就很可观了。就访问而言，以前要访问两个，现在要访问一个，访问总次数一多，耗时差距就很大了。（应该不用觉得兄弟节点会需要单独一次I/O从磁盘里读出来，毕竟外部排序的归并部分也是应该保证在内存里进行的，归并部分都要存硬盘了，这个归并k选的也不太对了。）

逻辑上走通了，再来说说code的问题。两个叶子节点比较，得到败者作为父节点，这里还很常规，但是还得往上送胜者，这里就微妙了。难道代表败者树的数组，每个元素都得存两个东西，胜者和败者？还是说，得用node指针来构建树？

complete binary tree不用数组怪浪费的。而且node指针构建树，一开始只有k个单节点的败者树，还要把它们merge起来。merge得先两两merge，注意，要一直保持每棵树的complete binary tree性质。所以，还总是需要构建无意义的节点，为了保持complete binary tree。有点离谱。

还是尝试用数组来表示败者树。其实败者树从构建成功起，就不会再变更树结构，所以建树期间的胜者可以用单独变量来保存，建好树之后，它们就没有意义了，只有全局的那个胜者有意义。

还是在 https://gist.github.com/vagetablechicken/8a2fc94da27f127921c0b6dd08d7863f

注意，胜者树可以随便更新参赛者，并且需要从叶子一路更新到根节点，不然可能存在问题。因为，新参赛者即使曾经是某一层的winner，现在值变了还是winner，但它的值变了，往上走它可能就不是了。只有它之前是loser，现在跟之前的winner比还是loser时，才可以停下来。这个优化可以，但没必要，容易写错。建议先写出简单不易错的算法，再谈要不要优化。

败者树则是必须更新winner所在的值，因为败者树要保证只更改胜者走的那条路径。有了这个保证，才能肯定parent存的败者一定是兄弟节点，才可以避免访问兄弟节点。败者树也得一路到根节点，不能因为以前是胜者，现在还是，就不往上比较了，值变了！！！



题解里的答案。。。
