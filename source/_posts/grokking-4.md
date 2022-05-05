---
title: Grokking Pattern 4
date: 2021-05-06 10:58:13
tags: [Algo, LeetCode]
categories: Algo
---

# Grokking the Coding Interview: Patterns for Coding Questions

See {% post_link grokking-1 %}.

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

具体到这个题目，首先要思考这个堆该如何运行，放多少数据，pop出来后又push多少进去，初步设计应该尽量分割步骤，目标是迅速写出正确的算法，后面再做优化。

所以初步设想是，k路数组，每个数组自己内部有序，但k个数组的头，我们是不知道谁最小的，所以把k个数组的头放进堆，我们就可以立马拿到最小的数，此时堆里剩k-1个数，这时候应该将pop出去的那一路的新头补充进堆，因为我们知道数组内的都比堆里的大，只有堆里的有资格比拼一下，所以堆大小在运行期间应该保持为k，不需要多放，多了增加复杂度，少了就比不出全局最小了。

Grokking标准答案也是使用python heapq来做，这个确实写得最快。

##### 败者树

堆解法不是终点， 如果提到有没有别的思路，就肯定要说到败者树算法了。本质都是树，不会有数量级提升。但总有些区别。

###### 外排序todo

winner/loser tree很像B+树，非叶子节点是不存值的，所以，和B+树类似，它们可以用在“外排序external sorting”上。

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
