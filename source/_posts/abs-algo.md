---
title: 抽象算法
date: 2022-10-07 19:48:30
tags: Algo
categories: Algo
---

很多题目都有基础的数据结构和算法，但也有它独特的corner case。所以，本文主要记录抽象的算法，避免重复记录。

## Quick Sort

快速排序

快排容易想的方法是3个while，一个while外循环，加每次左标移动和右标移动两个while。但这个翻译成代码很容易出错，主要是感觉while的保护和退出条件不容易记忆。出错了，你没法跑单测就很难补好洞。虽然面试的自己可能都写不出来，但他复制你的代码去跑，那就还是你gg。所以我倾向于记忆单循环的写法，主要是不需要什么保护和退出，简单。不过，3 while确实更加贴近抽象算法的原理，可以尝试记忆一下。

我们记锚点为pi，它位置上的值为piv。

### 单循环思路

快排建议记忆单循环的写法，该写法也不用死记，需要记住几个点：

1. 单次循环，又每个元素都得参与比较，所以for循环肯定是[l, r-1] 或者[l+1, r]的。
2. pivot选最右的那一个，这样可以直接利用range左闭右开的性质。
3. 遍历用i，那么还得有个标号j来表示[left, j]区间都是<=pivot的，初始可以j = left - 1，表示此时还没有能保证<=pivot的数，每次都先++再交换（不然就动了left-1的数了，也不可能）。划重点，[left, j]闭区间都是<=pivot的。
4. 什么时候交换？可以看到<=pivot的空间要增大，那么[i]<=pivot时，就应该挪到前面去。为什么不是 < pivot才交换？其实都可以😂，例如[这个题解就是用的小于做判断](https://segmentfault.com/a/1190000004410119)。j先++再i、j交换，j++后的值可能是已经被访问过的>pivot的，也可能是i自身，但都ok，>pivot的放后面不会错，自己跟自己交换也不会错。极端情况就是pivot就是最大的值，一路访问过来，到r-1的位置都不会实际交换。
5. 最后结果是i访问完了，没有什么指示性。j则肯定代表点什么。j位置必然是<= pivot的，那j本身肯定不能跟pivot交换，不能把小于等于pivot的换到尾部吧。所以应该是j+1。

注意判断条件用 <= 或 < 应该都可以，[oj](https://leetcode.cn/problems/kth-largest-element-in-an-array)上也没测出问题来。


单mark重点在于保证mark左边都是<=piv的，所以初始时mark可以是-1，意味着还没有保证任何元素<=piv。保证左边，那pi当然没理由在左边找，游标到pi位置时怎么整，还得折磨一番。所以pi选最右。自然游标从left到right-1就好了，不用看最右这个right（也是pi）。循环结束后，pi这个位子的piv是==piv的，完全可以又交换到左边（被交换到最右的那个值必然是>piv的，合理），成为一个分界点。

### 3 while思路

3 while思路，推荐在[陈斌老师的写法](https://www.bilibili.com/video/BV1Ya4y1x771?p=7)上改进一点点，一点点就可以。重点就是双mark由于写法过长容易产生的坑。

首先看基础思路：

我们先暂时放下while的保护条件，先讨论理想情况。两个游标怎么游？算法中不是要< piv, piv, > piv，毕竟可能piv有多个，而是<=piv, piv, >=piv，**注意，两边都可能==piv，不是必须在哪一边**。因此内层两个while判定条件是<=piv ++,  >=piv --，遇到等于的可以继续游，不需要停下。

而游标停下来有几种情况？因为我们涉及交换，所以可以想想离得近的情况，比如lmark+1=rmark，或lmark==rmark。第一个相邻好说，你都停下来了，必然是lmark的数> piv，rmark的数< piv，它们需要交换，正常进行。而lmark==rmark这是不可能的，因为一个数它跟piv比只有三种情况，而三种情况，两个游标都不可能同时停下，必然有人要跨过去，所以安全。内层双while结束时，很可能有lmark>rmark的(不会有==，当然，如果有保护机制，可能影响)。那我们就应该在lmark>rmark时终止，==不应该存在（甚至可以assert看看），也别去记它，干扰思路。反正lmark>rmark时不能交换，交换前也要判断这一情况，所以可以外层True，内层break（陈斌老师用的done变量，没必要）。**重点记忆，“交错”(lmark>rmark)就停止**。

外层while也跳出后，是个什么样子？lmark>rmark了，piv该放哪儿，和左还是右交换？由于piv是拿的最左边，交换到最左边的当然应该是<=piv的，所以拿rmark是ok的。rmark已经在lmark左边了，它必然指向了<=piv的值，没必要用lmark做更多的处理。那么**交换一下rmark和pi（最左）就ok了**。

所以partition**初步**的伪代码为：
```
while True:
	while … and [lmark]<=piv: lmark++
	while … and [rmark]>=piv: rmark--
	if lmark > rmark:
		break
	exchange
exhange rmark and left
```

现在还需要加保护。比如piv刚好是最小的值，lmark能一直走穿，partition函数如果只对局部数组做，还可能踩到错误地方。lmark可以<=right（当前partition输入数组的最右），也可以<=rmark，因为lmark>rmark就会退出循环了，不需要让它继续发展。所以推荐rmark。那rmark也一样，只要交错就可以退了，不需要用最左边当边界，所以rmark>=lmark就行了。需要有==，不然就保护太过了，不可能出现游标交错。

可以理解为，**核心是“交错”，交错就停止，只要不交错，3 while都可以继续**。
```
while True:
	while lmark<=rmark and [lmark]<=piv: lmark++
	while lmark<=rmark and [rmark]>=piv: rmark--
	if lmark > rmark:
		break
	exchange
exhange rmark and left
```

实际oj测试，还是双mark快点。

另一种写法见[leetcode题解](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/solution/jian-zhi-offer-40-zui-xiao-de-k-ge-shu-j-9yze/)，3 while用的i < j，注意写法是右标先移动，否则有问题。这种大概率记不住，先左后右比较顺。
