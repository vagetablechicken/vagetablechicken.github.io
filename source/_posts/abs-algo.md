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
2. pivot选最右的那一个，这样可以直接利用range左闭右开的性质。（这里很容易想成最左作为pivot，然后就会陷入pivot的空洞怎么swap的疑惑里。注意只要想**不要交换空洞**就好了。也就是如果最左为pivot，那也是从l+1开始才会有交换。）
3. 遍历用i，那么还得有个标号j来表示[left, j]区间都是<=pivot的，初始可以j = left - 1，表示此时还没有能保证<=pivot的数，每次都先++再交换（不然就动了left-1的数了，也不可能）。划重点，[left, j]闭区间都是<=pivot的。
4. 什么时候交换？可以看到<=pivot的空间要增大，那么[i]<=pivot时，就应该挪到前面去。为什么不是 < pivot才交换？其实都可以😂，例如[这个题解就是用的小于做判断](https://segmentfault.com/a/1190000004410119)。j先++再i、j交换，j++后的值可能是已经被访问过的>pivot的，也可能是i自身，但都ok，>pivot的放后面不会错，自己跟自己交换也不会错。极端情况就是pivot就是最大的值，一路访问过来，到r-1的位置都不会实际交换。
5. 最后结果是i访问完了，没有什么指示性。j则肯定代表点什么。j位置必然是<= pivot的，那j本身肯定不能跟pivot交换，不能把小于等于pivot的换到尾部吧。所以应该是j+1。

判断条件用 <= 或 < 应该都可以，[oj](https://leetcode.cn/problems/kth-largest-element-in-an-array)上也没测出问题来。


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

## Binary Tree Traversal

二叉树遍历

二叉树的三种遍历递归写法都没难度，不过稍微加点东西就容易搞不定了。还是不够熟悉递归，不够熟悉将稍复杂的逻辑想清楚，所以转换为代码会卡住。

### 中序遍历 

https://leetcode.cn/problems/binary-tree-inorder-traversal/

中序递归遍历，一般就用result数组来存结果就行了，因为result在递归中也是一个一个append结果的。拼接的写法`inorder(root.left)+[root]+inorder(root.right)`大可不必，空间消耗有点过分了，也没简单多少。

而中序迭代遍历，该怎么写？首先得加个辅助结构，或者改当前的数据结构，否则不够用。

我们先说“加辅助结构”的算法。中序是左-中-右，递归不用额外结构因为它可以回到中，即使已经在左子树游了一圈了。那迭代就需要记住中，因二叉树单向的，往下走回不来，不缓存起来就回不去。不管是栈还是队列，一维数组肯定够了。当然一般“递归->迭代”都是栈，递归本质也是先进后出。

再看怎么使用栈，其实就是模仿递归，递归往左下走到底后，可以逐步回来，所以迭代里就应该往左下走，并且记录每个“中”节点。到底后，就可以向result加结果了。拿最左的叶子节点举例，它左子节点None了，就类似递归回到了它，它作为中节点，被加入result，那么此时还应该考虑它的右子节点。而以这个右子节点作为起始，做的实际还是中序遍历，不过是子树的中序遍历，所以这里代码可以复用，只需要以右子节点作为起始节点。可以看出，中先进栈，左子再进，把左子和中都弹出栈后，右子才会进栈，不用担心右子节点遍历完了后就断了，因为中的上一层还在栈里面。

伪代码写作：
```
cur = root
stack = []
while ?:
	while cur:
		cur -> stack
		cur = cur.left
	# no more left child, cur == None now
	top = stack.pop
	top -> result
	cur = top.right # next loop will traversal the right child
```

最后看终止条件怎么写，stack为空，没法pop，肯定得保护，但是因为内层while也在填充stack，所以并不是stack为空就得终止，内层while可能会补充。不过如果cur和stack都为空，那就没必要了。

所以终止条件为`while cur or stack`。

### 前序遍历/先序遍历

https://leetcode.cn/problems/binary-tree-preorder-traversal/

递归写法是“中-左子-右子”，因为中就是当前，左子就`.left`往左走就行了，所以只需存右子，且是先进后出。用简单例子演示一遍也可以知道，伪代码可以写作：
```
stack, cur = [], root
while ?:
	if not cur:
		cur = stack.pop
	cur -> result
	if cur.right: -> stack
	cur = cur.left # be the new traversal root
```

最后看终止条件，stack为空，如果有cur，也可以走下去（比如root节点）；如果cur为空，stack还有，也能走（比如走到树的最底层，但树右边还有节点没遍历到）。所以终止条件还是`while cur or stack`。

### 后序遍历

https://leetcode.cn/problems/binary-tree-postorder-traversal/
