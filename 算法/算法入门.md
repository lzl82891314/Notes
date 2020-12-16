# 学习总结

大家好，我是DD3的李正龙，很荣幸能给大家总结一下近期我学习的内容，希望能帮助到大家共同学习，共同进步。

近期我在极客上学习[算法训练营](https://u.geekbang.org/subject/algorithm/1000343?utm_source=time_web&utm_medium=menu&utm_term=timewebmenu)的相关课程。目前课程刚刚过半，正好可以将之前所学习到的东西总结一下，也是一种温故知新。如果对算法感兴趣想要重新回顾或者学习的朋友，可以看看这篇来降低入门的难度。如果是算法高手，也希望能够帮我指出我所说的不足之处，毕竟我也还未出师，希望能和大家能一起成长。

这次总结的内容也都是我自己的所思所想所感，由于自己刚开始学习水平有限，如果总结中有什么不对的地方希望大家能帮我指出，感谢大家。

## 前言

我觉得程序员的技能天赋树主要有两个子树：`外功招式`和`内功心法`。

在“码农”这片腥风血雨的江湖中，能力是我们赖以生存的唯一途径，如果不想被奔涌而来的后浪所淘汰，就要不断修炼自己，点满更深的天赋点，学会更多的武功招式。而修炼的前提，是需要先找到自己的门派，学习门派中的各个招式，待到学成之后就可以行走江湖了。其中的门派，其实就是我们日常工作中所用到的技术栈，所有的招式，其实就是这种技术栈下所用到的各种技术框架和编程思想。我觉得以上这些可以统一称为程序员的外功招式。

但是，如果想在江湖中走得更远，那最重要的其实是我们的内功心法即算法。想虚竹子获得了无崖子70年的内功之后，可以平地飞天，松子杀人是多么的潇洒飘逸，但是我们没有主角光环，没法直升70级，那只能靠我们自己不断修炼最终成为主角了。

## 第一招：入门式

最近有个微博上很火的段子，不知道大家看过没有：

![isEven](https://github.com/lzl82891314/AlgorithmDeliberatePractice/blob/master/src/geek%20time/summarize/resource/isEven.png)

虽然这只是个段子（我不知道是不是在黑C#程序员（笑）），但是已经可以看出算法的重要性了，这个题大家肯定都可以当God，这种利用计算机`快速稳定`解决重复问题的方法我觉得就可以被定义为一种算法了。而这里又引出了两个概念：快速和稳定。

快速其实就是指程序的`时间复杂度`，与其对应的概念就是`空间复杂度`。在如今这个内存已经普遍充裕的时代，一个算法的好坏的唯一度量标准可能就是时间复杂度了。这两个概念统一都用大O表示法表示，其主要可以分为O(1)、O(logn)、O(n)、O(nlogn)、O(n^2)、O(2^n)。这些都是数据结构的基础概念了，想必大家的理解比我更深，我就不在这班门弄斧了。但是得说一个重要的概念就是：

> 程序算法的优劣其实就是时间和空间的博弈

目前一个设计无误的算法基本都是利用基础数据结构进行空间换时间的，因为目前不存在一种数据结构即快速又占用率低，因为如果那样就不会存在其他基础的数据结构了。

接着上面的那个图，有人在评论里给出了一种解决方式，我在这写出来让大家观赏一下：

``` C#
private IsEven(int number) {
    if (number == 1) return false;
    if (number == 2) return true;
    return IsEven(number - 2);
}
```

这里就引出了我们日常工作中可能用到的最多的一种算法了，递归。而说到递归就又回到那个基础的绕不开的一个经典问题：斐波那契数列问题。

相信一个普通的斐波那契数列的代码大家可以很轻松的写出来：

``` C#
private int Fibonacci(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

上述的斐波那契数列代码看起来其实十分整洁简练，但是大部人可能不知道这个算法的时间复杂度是`O(2^n)`是一个最慢的算法类型，因为其状态展开树为：

![Fibonacci](https://github.com/lzl82891314/AlgorithmDeliberatePractice/blob/master/src/geek%20time/summarize/resource/fibonacci.jpg)

可以从图中看到递归代码中出现了很多重复计算的子函数，因此随着n的增加，其算法时间复杂度是指数型增加的。

上面这个算法问题的一个好的解决方案有两个：第一个就是我最开始提到的`时间空间博弈`的方法，可以用空间来换时间，比如：

``` C#
private IDictionary<int, int> hashMap = new Dictionary<int, int>() { {0, 0}, {1, 1} };
private int Fibonacci(int n) {
    if (n < 0) return 0;
    if (hashMap.ContainsKey(n)) return hashMap[n];
    var ans = Fibonacci(n - 1) + Fibonacci(n - 2);
    hashMap.Add(n, ans);
    return ans;
}
```

利用一个哈希表存储中间运算结果，就可以将此算法`优化为O(n)`的时间复杂度，但是与之带来的，是又`增加了O(n)的空间复杂度`。

那是不是还有不增加空间复杂度的优化方法呢？第二种方法就是这种方式。第二种方法就是通过`数学归纳法`计算求得其问题的重复性`通项公式`，也就是所谓的DP动态规划，将递归问题手动改写为迭代问题：

``` C#
private int Fibonacci(int n) {
    if (n <= 0) return 0;
    if (n <= 2) return 1;
    var f1 = 1; var f2 = 2;
    for (var i = 3; i < n; i++) {
        f2 += f1;
        f1 = f2 - f1;
    }
    return f2;
}
```

通过计算通项公式，可以将其转换为迭代的方式，而一般来说迭代的中间结果需要通过数组存储，但是这个问题比较特殊，可以直接通过两个变量记录中间值，因此这个问题就可以被优化为时间O(n)和空间O(1)的复杂度。

可以看到，这么一个简单的问题，里面的算法内容真的不少，如果还想深入了解相关问题，大家可以看[如何优雅地计算斐波那契数列？](https://time.geekbang.org/dailylesson/detail/100028406)。而通过不断地学习，也可以增加我们看问题的角度和广度。比如学习的算法，以后在工作中碰到了类似的问题，我们是不是就会比以前多考虑一步，多想想我们写出的程序是否有可以优化的地方。


相信看到这里，大家已经对算法有了初步的概念：算法是一种解决问题的思路和方法，通过不断地优化可以将程序设计为更方便计算机处理的方法。并且算法的性能是时间空间博弈的结果，可以通过增加空间复杂度来优化时间复杂度。

### 第二招：修炼式

> 纸上得来终觉浅，绝知此事要躬行

既然拜了门派，就要开始修炼了。算法是一门很基础并且没有捷径可寻的一门知识体系，能做的就是多做多练。而我们的修炼场就是[LeetCode](https://leetcode-cn.com/)。这个站点相信大家或多或少都听说过，是一个程序员的算法题库，相信公司也有很多大牛都在里面做过很多练习了（刚入职新人介绍的那天，同部门的 Chaos 同学说他LeetCode 排名前300，这也让我发现了我和同事之间的差距，也是一个我想攻克算法的一个诱因吧 （笑））

既然要修炼，那就要有得道的方法，这样才能事半功倍并且大大增加自己的满足感降低挫败感。

关于挫败感的问题我想单独说一下，我觉得这个问题挺重要的，也是我之前`从打鸡血到入门`，再`从入门到放弃`无数次想要攻克算法而不能的主要原因。因为当时有一个很明显的误区，碰到做不出来的问题就以为自己还可以抢救一下，最终经过冥思苦想还是做不出来。

![3Sum_ErrorList](https://github.com/lzl82891314/AlgorithmDeliberatePractice/blob/master/src/geek%20time/summarize/resource/3Sum_ErrorList.png)

这是我4年之前想要学习算法时做[3Sum](https://leetcode.com/problems/3sum/)这个题时留下的记录，当时就是想和这个问题死磕到底，结果最终还是失败了，就直接导致了我当时放弃了算法学习，因为当时觉得自己做不出来就是自己水平不够，这种入门的题都不会做后面的就更别想了。

所以通过训练营，我学习到了正确的修炼方式：`五步刷题法`。这是训练营覃超老师自创的学习方法，即一个问题做5遍：

1. 第一遍：先自己审题并且思考，如果没有思路则直接选择跳过，看官方题解以及高票回答，选择一个简洁清晰的代码背诵下来
2. 第二遍：将之前背诵理解的代码用自己熟悉的语言实现出来
3. 第三遍：第二天再次反复做这个问题，直到将这种解题方式烂熟于心
4. 第四遍：隔几天再来看这道题，将这个问题的解题思路和核心代码使用脑图等方式自己记录下来
5. 第五遍：在以后工作中或者需要面试的时候回看第四遍脑图中总结的概念，对相关问题有解题概念

相信大家现在的想法和我之前的想法完全一样，第一次听说需要这么学习算法的时候我感觉我上当了，白花了钱买了这个训练营，竟然教学的方法是让我们背代码（笑）。但其实，作为一个刚入门的小白来说，在什么都不会的阶段就是需要积累前人的经验，将其转换成自己的经验才能在日后付诸实践。我们不是计算机科学家，那些题解里的解法基本都是很多科学家冥思苦想以及做了很多实验而得到的解决，我们要做的就是将这些结果转换为自己的知识体系。

就这样，摆脱了之前的误区，使用这个方法进行算法题训练，果然效果很明显，虽然只完成了一半，但是现在遇到很多类似的问题我都有了解题思路并且最终自己完成Accept，也可以很快的看出代码中的不足之处。

配合这种刷题的方法，我也推荐一个软件`滴答清单`，里面可以很方便地记录每天刷的题，并且可以设置`艾宾浩斯记忆法`来重复推送我们需要反复修炼的问题：

![dida_list](https://github.com/lzl82891314/AlgorithmDeliberatePractice/blob/master/src/geek%20time/summarize/resource/dida_list.png)

除了五步刷题法之前，遇到具体的问题还有一个做题的方法：

1. 明确题目要求：有些题目会隐晦的边界问题，最常见的有int —— long之间的转换，以及空数组等等
2. 列举所有解法：一个问题一般都是有多种解法的，比如上述的斐波那契数列问题就有至少3种解法，把每一种能想到的解法都列出来
3. 选择一个时间复杂度最优的解法

相信看到这里，大家也有了一些概念：算法的修炼就是对问题的刻意练习，更多的练习意味着我们能够积累更多的解题思路和方法，从而提升自己的内功水平。

最后还要给大家说一下，LeetCode中的题目有三个等级：

1. easy题基本就是固定算法的直接使用，如果想要看自己掌握一种算法与否，那么刷一遍相关算法的easy题即可。
2. medium题都是特定算法的变种应用，想要看自己对特定算法的熟练度就应该刷medium题。
3. hard题一般都是比较难的算法的应用题，解题方法不单一，而且可能不光使用一种算法解决，是想成为大牛的必备。

### 第三招：技巧式

上面两部分我都是在列举一些概念性的问题，从这节开始我来总结一下这一个月的算法学习的所有知识点吧。我会从我学习到的知识点开始回顾总结，每一个主要的知识点我会列出一个相关的最经典问题出来，用代码和注释的方式解释相关的问题思路。如果你已经练习过很多算法，那这部分我觉得应该可以跳过，当然也欢迎大家指出我的问题，共同进步。

#### 双指针法

部分数组和链表的问题可以通过这种方法进行解答。比如[141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)这个判断链表是否有环的问题，我们可以通过快慢指针的方法求得，初始化两个指针，一个每次后移两步，一个后移一步，最终如果两个指针相遇了，说明这个链表是有环的：

``` C#
public bool HasCycle(ListNode head) {
    if (head == null || head.next == null) return false;
    // 初始化快慢指针
    var fast = head.next; var slow = head;
    // 注意判断条件，如果不判断fast.next则会导致空引用异常
    while (fast != null && fast.next != null) {
        // 只有两个指针相遇，则说明链表有环
        if (fast == slow) return true;
        fast = fast.next.next;
        slow = slow.next;
    }
    return false;
}
```

在不看题解之前我是肯定不会想到还有空间复杂度O(1)的这种解法的，但是也可以看到，这个问题类似于操场跑步套圈的问题，说明算法的源头其实还是来源于生活。当然上述的问题还有其他O(n)的解法，比如HashMap存储中间值。

第二个特别出名的双指针技巧题就是我上面提到的[15. 3Sum](https://leetcode.com/problems/3sum/)问题了。不同于上一个链表题，这个问题是使用两个指针分别指向指向头和尾，然后双端递进，最终使用以前初中数学学到的夹逼法解决：

``` C#
public IList<IList<int>> ThreeSum(int[] nums) {
    var ans = new List<IList<int>>();
    if (nums == null || nums.Length == 0) return ans;
    // 首先需要排序，在计算3sum时由于是双重循环，时间复杂度是O(n^2)大于O(nlogn)，因此排序不算在最终的时间复杂度计算内
    Array.Sort(nums); 
    // 注意这里的循环终止条件应该是length - 2，因为要预留两个位置给双指针
    for (var i = 0; i < nums.Length - 2; i++) {
        // 注意两个判重条件，当两次循环的值相等时，跳过此次计算
        if (i > 0 && nums[i] == nums[i-1]) continue;
        // 当遍历最左侧nums[i] > 0时，说明不可能存在三个数的和等于0，因此结束循环
        if (nums[i] > 0) break;
        var left = i + 1; var right = nums.Length - 1;
        while (left < right) {
            var sum = nums[i] + nums[left] + nums[right];
            if (sum < 0) {
                // 说明此时left的值过小，需要移动left，使用while循环判重
                while (left < right && nums[left++] == nums[left]);
            } else if (sum > 0) {
                // 同理移动right
                while (left < right && nums[right--] == nums[right]);
            } else {
                ans.Add(new int[] { nums[i], nums[left], nums[right] });
                while (left < right && nums[left++] == nums[left]);
                while (left < right && nums[right--] == nums[right]);
            }
        }
    }
    return ans;
}
```

夹逼法还有 一道比较经典的题[11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)，计算面积最大的矩形：

``` C#
public int MaxArea(int[] height) {
    var ans = 0;
    if (height == null || height.Length == 0) return ans;
    var left = 0; var right = height.Length - 1;
    while (left < right) {
        // 只要高度小的那一方移动计算，高的那一边不动
        var minHeight = height[left] < height[right] ? height[left++] : height[right--];
        ans = Math.Max(ans, minHeight * (right - left + 1));
    }
    return ans;
}
```

这个题需要思考一下才能理解，并且其中不会漏掉一些可能得计算内容，这部分数学证明可以在题解中找到。当然这个问题也可以使用暴力法双重循环求解，但是不会得到最终O(n)的时间复杂度。

#### 链表

对于链表的题基本没有任何算法逻辑，有的只有熟练度。这里重点说一下我遇到的第一个劝退题[25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)，题目的意思很简单，就是将一个链表按照参数k分组反制，这个题我真的是做了一天最终才做到完全不看题解随手就写，链表的题还是需要推荐用纸和笔画出步骤来一步一步看的，不然很难得到最终的正确结果：

``` C#
public ListNode ReverseKGroup(ListNode head, int k) {
    if (head == null) return head;
    // 对于这种需要翻转的题，刚开始都需要有一个头节点接住反转的第一个head节点
    // 因此一般的做法是创建一个dummy节点当这个头
    var dummy = new ListNode();
    dummy.next = head; 
    // 并且这个题不光需要一个前置节点，还需要一个tail节点，因为需要用这个节点去判断是否满足按k分组的条件
    var pre = dummy; var cur = head; var tail = head;
    while (cur != null) {
        var count = k;
        while (count-- > 0 && tail != null) tail = tail.next;
        if (count > -1) {
            pre.next = cur;
            break;
        }
        pre.next = Reverse(cur, k);
        pre = cur; cur = tail;
    }
    return dummy.next;
}
private ListNode Reverse(ListNode head, int k) {
    // 此外，这个题还很好地考查到了链表反转这个子问题
    ListNode pre = null;
    while (k-- > 0) {
        var next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre;
}
```

我觉得这个题算是链表题的一个坎，跨过去了之后就会发现链表的题都可以掌握。

#### 堆

堆可能是我们日常工作中比较常用的一个数据结构问题了，类似于TopK之类的问题都是用堆来解决的，算法学习中，遇到的一个堆的问题是[239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)，计算滑动窗口的最大值问题。这个题，如果是Java程序员是很好解决的，因为Java的JDK中就有`PriorityQueue`这样的数据结构，但是C#没有。用来模拟堆的是一个`SortedSet<T>`的数据结构，这个数据结构有一个很大的弊端就是值不可重复，这样就为解这个题带来了很高的复杂度，需要另外的哈希表排重：

``` C#
public int[] MaxSlidingWindow(int[] nums, int k) {
    var ans = new int[nums.Length - k + 1];
    if (nums == null || nums.Length == 0) return ans;
    var heap = new SortedSet<int>();
    // 需要一个hashMap做排重操作
    var hashMap = new Dictionary<int, int>();
    var left = -k + 1;
    for (var i = 0; i < nums.Length; i++, left++) {
        // 每次有新的相同值插入时，hashMap的值做更新操作
        var cur = nums[i]; hashMap[cur] = i;
        // 当移除滑动窗口的元素就是哈希表中的对应元素时，将其移出heap
        // 配合上述的更新操作，就可以合并起到排重的作用
        if (left - 1 >= 0 && hashMap[nums[left - 1]] == left - 1) heap.Remove(nums[left - 1]);
        heap.Add(cur);
        if (left >= 0) ans[left] = heap.Max;
    }
    return ans;
}
```

另外，Java中的PriorityQueue是用`斐波那契堆`实现的，其性能比二叉堆要好。C#中的SortedSet<T>是用`红黑树`实现的，其代码复杂度更高。

#### 递归

又回到了递归的问题。递归的算法有一定的代码编写模板：

``` C#
public object Recursion(params) {
    // 递归终止条件
    if (args == XXX) return value;

    // 处理当前层逻辑
    value = ABDCD;
    // ...

    // 下钻到下一层
    value = Recursion(params - 1);

    // [回调修改当前层状态]（可选）
    param += 1;
}
```

按照这个模板可以很顺利的写出递归的程序，也是因为这个模板，我也可以一眼看出工作代码中编写的递归程序的问题。而日常工作中递归用到的最多的地方就是树的遍历，树由于其数据结构的特殊性，很难通过普通的迭代函数遍历，但是却可以完全通过递归的形式遍历出来。其树的遍历方式就是数据结构的相关内容了，按深度遍历就是DFS，按层遍历就是BFS，在这我就不多说了。

递归比较入门的题目就是[70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)，这个题和斐波那契数列完全相同就不写了。另一个比较经典的纯递归题是[236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)，给定一个二叉树的两个节点，求这两个节点的公共父亲节点。

``` C#
public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    // 分别递归计算左右子树的祖先left和right
    var left = LowestCommonAncestor(root.left, p, q);
    var right = LowestCommonAncestor(root.right, p, q);
    // 如果left和right节点都是其子树的祖先，说明root才是公共祖先
    return left != null ? right != null ? root : left : right;
}
```

可见，纯递归问题一般都是求解树子树问题，所以以后碰到树的子树相关问题应该第一时间想到使用递归求解。

递归的另一个分支就是求解排列组合相关的问题，比如：[46. Permutations](https://leetcode.com/problems/permutations/)、[77. Combinations](https://leetcode.com/problems/combinations/)、[47. Permutations II](https://leetcode.com/problems/permutations-ii/)。这一系列的问题都是递归的另一个分支`回溯法`。以全排列这个问题为例：

``` C#
public IList<IList<int>> Permute(int[] nums) {
    var ans = new List<IList<int>>();
    if (nums == null || nums.Length == 0) return ans;
    Backtracking(nums, new HashSet<int>(), ans);
    return ans;
}
private void Backtracking(int[] nums, HashSet<int> hashSet, IList<IList<int>> ans) {
    // 这个题的解题形式就完全符合上述的递归模板
    // 首先是递归终止条件
    if (hashSet.Count == nums.Length) {
        ans.Add(new List<int>(hashSet));
        return;
    }
    // 其次处理当前层逻辑
    foreach (var num in nums) {
        if (hashSet.Contains(num)) continue;
        hashSet.Add(num);
        // 下钻到下一层递归
        Backtracking(nums, hashSet, ans);
        // 回溯修改当前层状态
        hashSet.Remove(num);
    }
}
```

这种需要使用到回溯法解决的递归问题在入门阶段还是有很大的门槛需要迈过去的。在多做多练之后，对相关的问题也就有了一定的概念和解题的方向。

递归的下一个分支是`分治法`，最经典的一个问题就是[50. Pow(x, n)](https://leetcode.com/problems/powx-n/)，计算x的n次方问题。这个问题的常规解法其实是使用循环在O(n)的时间复杂度内完成的，但是利用递归和分治的思想，可以优化到O(logn)：

``` C#
public double MyPow(double x, int n) {
    if (n < 0) {
        // 如果n为负数，则利用数学方法将x改为x分之1
        x = 1 / x; n = -n;
    }
    return RecursivePow(x, n);
}
private double RecursivePow(double x, int n) {
    if (n == 0) return 1.0d;
    // 利用分治法，将一个大问题切分成了logn个子问题的乘积，原理和二分查找相同
    var half = RecursivePow(x, n / 2);
    return n % 2 == 0 ? half * half : half * half * x;
}
```

#### BFS和DFS

这两种遍历方法就和迭代中的for循环一样，是基础中的基础，此类问题一般是解决图相关的遍历搜索问题，比如比较经典的问题[127. Word Ladder](https://leetcode.com/problems/word-ladder/)，给定两个单词和一个单词列表，问从第一个单词到第二个单词的变化的最小次数问题。第一次碰到这个题的时候我是懵的，不知道用什么方法解，但是看了题解才知道原来BFS是这样使用的：

``` C#
public int LadderLength(string beginWord, string endWord, IList<string> wordList) {
    if (beginWord.Equals(endWord) || wordList.Count == 0) return 0;
    var hashMap = new Dictionary<string, IList<string>>();
    foreach (var word in wordList) {
        for (var i = 0; i < word.Length; i++) {
            // 首先将每个词分别拆分成 *ot, h*t 和 ho*这样的匹配模式存入哈希表中加速，为后续的BFS做准备
            var commonWord = word.Substring(0, i) + "*" + word.Substring(i + 1, word.Length - i - 1);
            if (!hashMap.ContainsKey(commonWord)) hashMap.Add(commonWord, new List<string>() { word });
            else hashMap[commonWord].Add(word);
        }
    }
    // 对于图的遍历算法中，visited集合是必须要的，因为如果没有visited，则会在有环图中无限循环
    var visited = new HashSet<string>();
    // BFS的基础queue
    var queue = new Queue<KeyValuePair<string, int>>();
    queue.Enqueue(new KeyValuePair<string, int>(beginWord, 1));
    while (queue.Count > 0) {
        var cur = queue.Dequeue();
        for (var i = 0; i < cur.Key.Length; i++) {
            // 将每次匹配到的值做匹配模式处理，由此可以得到该词的下一个可以到达的词
            var commonWord = cur.Key.Substring(0, i) + "*" + cur.Key.Substring(i + 1, cur.Key.Length - i - 1);
            if (!hashMap.ContainsKey(commonWord)) continue;
            var matchedList = hashMap[commonWord];
            foreach (var matchedWord in matchedList) {
                // 匹配到则直接返回
                if (matchedWord.Equals(endWord)) return cur.Value + 1;
                if (visited.Contains(matchedWord)) continue;
                // 否则将子问题加入BFS的queue中继续下一层计算
                visited.Add(matchedWord);
                queue.Enqueue(new KeyValuePair<string, int>(matchedWord, cur.Value + 1));
            }
        }
    }
    return 0;
}
```

对于上述的问题还有一个点需要指明，就是这个BFS算法的时间复杂度问题。一般我们都会将循环的层数默认判定为时间复杂度，但是这个问题中，最终入队列的while循环其实其最大值就是wordList的长度n，而字符串匹配的for循环算法的长度是每个单词的长度m，所以这个算法的时间复杂度应该是O(mn)。其中的foreach循环体的值其实是n的一个子集此处可以当做常数来判断。

关于DFS的题，有一个经典问题就是[529. Minesweeper](https://leetcode.com/problems/minesweeper/)扫雷，相信大家都玩过Windows自带的扫雷游戏，这个题也是类似的，给定一个雷图，和一次点击的位置，返回雷图的最终结果：

``` C#
public char[][] UpdateBoard(char[][] board, int[] click) {
    if (click == null || click.Length == 0) return board;
    var x = click[0]; var y = click[1];
    if (x < 0 || x >= board.Length || y < 0 || y > board[0].Length) return board;
    if (board[x][y] == 'M') {
        // 第一次点击直接点中了雷，游戏结束返回
        board[x][y] = 'X';
        return board;
    }
    // 一个方向数组，为了方便计算DFS的8个扩展的方向
    var direction = new int[] { 0, 1, -1 };
    DFS(board, x, y, direction);
    return board;
}
private void DFS(char[][] board, int x, int y, int[] direction) {
    // 递归终止条件
    if (x < 0 || x >= board.Length || y < 0 || y >= board[0].Length) return;
    if (board[x][y] == 'B' || board[x][y] == 'M' || char.IsDigit(board[x][y])) return;
    var mines = 0;
    // 需要先做一遍搜索计算出此刻这个方格周围的地雷数量
    for (var i = 0; i < direction.Length; i++) {
        for (var j = 0; j < direction.Length; j++) {
            if (direction[i] == 0 && direction[j] == 0) continue;
            var tempX = x + direction[i]; var tempY = y + direction[j];
            if (tempX < 0 || tempX >= board.Length || tempY < 0 || tempY >= board[0].Length) continue;
            if (board[tempX][tempY] == 'M') mines++;
        }
    }
    board[x][y] = mines == 0 ? 'B' : char.Parse(mines.ToString());
    // 这句话是所有DFS的重点，题目中要求只能DFS方格为E的点，因此当此时的方格周围有雷，则应该立刻停止DFS
    // 这也是为什么要把计算地雷的逻辑放在DFS之外的原因
    if (mines > 0) return;
    for (var i = 0; i < direction.Length; i++) {
        for (var j = 0; j < direction.Length; j++) {
            if (direction[i] == 0 && direction[j] == 0) continue;
            // 递归下钻
            DFS(board, x + direction[i], y + direction[j], direction);
        }
    }
}
```

#### 二分查找

一个简单的二分查找我相信大家都能写出来，但是二分查找的问题其实主要是计算二分查找的边界值问题，最经典的就是[74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)，搜索单调递增二维矩阵的值：

``` C#
public bool SearchMatrix(int[][] matrix, int target) {
    if (matrix == null || matrix.Length == 0 || matrix[0].Length == 0) return false;
    var left = 0; var right = matrix.Length - 1;
    int[] searchArray = null;
    while (left <= right) {
        // 利用二分查找通过计算矩阵边界获得需要搜索的一维数组
        var mid = left + (right - left) / 2;
        // 通过对数组初始值判断边界
        var firstValue = matrix[mid][0];
        if (target < firstValue) {
            // 通过判断target所在的一维数组的边界值来确定具体的target可能存在的一维数组
            if (mid == 0) return false;
            else if (target > matrix[mid - 1][0]) {
                searchArray = matrix[mid - 1];
                break;
            } else right = mid - 1;
        } else if (target > firstValue) {
            // 同上
            if (mid == matrix.Length - 1 || target < matrix[mid + 1][0]) {
                searchArray = matrix[mid];
                break;
            } else left = mid + 1;
        } else return true;
    }
    left = 0; right = searchArray.Length - 1;
    while (left <= right) {
        // 最终通过标准的二分查找计算结果
        var mid = left + (right - left) / 2;
        if (target < searchArray[mid]) right = mid - 1;
        else if (target > searchArray[mid]) left = mid + 1;
        else return true;
    }
    return false;
}
```

与这个题类似的还有一个问题[153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)是计算半有序数组的最小值问题，所谓的半有序数组即[3, 4, 5, 1, 2]。这个问题同样可以用二分查找来解决：

``` C#
public int FindMin(int[] nums) {
    if (nums == null || nums.Length == 0) return -1;
    var left = 0; var right = nums.Length - 1;
    if (nums[left] <= nums[right]) return nums[left];
    while (left <= right) {
        var mid = left + (right - left) / 2;
        // 此问题和上述问题类似，也是通过二分查找计算边界值问题
        if (nums[mid] > nums[mid+1]) return nums[mid+1];
        else if (nums[mid] < nums[mid-1]) return nums[mid];
        else if (nums[mid] > nums[0]) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

#### 贪心算法

在此次学习之前，我一直认为贪心算法根本没有用武之地，因为就是每次抉择选最优解，这种问题几乎不会存在。但是现在发现确实是自己目光短浅了，贪心算法并不是很简单的无脑使用，而是配合一些技巧性以及大问题中的子问题使用。此外，如果一个问题可以使用贪心算法解决，那么贪心算法肯定是这个问题的最优解。

比如这个题[860. Lemonade Change](https://leetcode.com/problems/lemonade-change/description/)就是贪心算法的入门题，其解法非常简单，就是模拟题目描述直接计算求解即可：

``` C#
public bool LemonadeChange(int[] bills) {
    var five = 0; var ten = 0;
    foreach (var bill in bills) {
        if (bill == 5) five++;
        else if (bill == 10) {
            if (five-- > 0) ten++;
            else return false;
        } else {
            // 直接使用贪心法，如果存在10元，则直接兑换，否则兑换3个5元
            if (ten > 0 && five > 0) {
                ten--; five--;
            } else if (five >= 3) five -= 3;
            else return false;
        }
    }
    return true;
}
```

但是这么简单的题为啥我要单独拿出来说呢，是因为这个题有一个很明显可以使用贪心的原因：题目中的条件5，10，20相互成倍出现的，这样就可以完全用10代替20，用5代替10，满足使用贪心的要求。如果题目中的零钱变成了1，9，10那么这个问题使用贪心就无法解决了，需要全局最优解的动态规划来完成。

贪心的另外一个比较有技巧性的问题是[55. Jump Game](https://leetcode.com/problems/jump-game/)，这个问题的技巧是不是直接使用贪心计算结果，而是逆向反过来计算最终得到答案：

``` C#
public bool CanJump(int[] nums) {
    var ans = false;
    if (nums == null || nums.Length == 0) return ans;
    // 记录当前能够到达的最远路径
    var destination = nums.Length - 1;
    // 此处i从nums.Length - 1开始计算是为了解决nums == [1]这种只有一个值的边界条件
    for (var i = nums.Length - 1; i >= 0; i--) {
        // 说明当前节点可以到达最终它可以达到的目的地
        if (nums[i] + i >= destination) {
            // 此时贪心将目的地改为当前位置
            destination = i;
            ans = true;
        } else ans = false;
    }
    return ans;
}
```

以上就是我最近的学习中遇到的重点问题的总结，虽然列举了很多算法题有些流水账，但是我也把我最近所学的重点内容再此列出，希望能够帮助到想要学习算法还没有入门的同学。

### 第四招：进阶式

这部分内容我其实并不想写的，因为我自己都没学完也是个半吊子，但是为了对仗工整，我还是说一下自己对算法进阶的理解和我打算未来要做的事。

首先，刷题是不能停的，LeetCode中文版有每日一题的功能，每天打卡刷一题这种形式特别适合我，自增加了成就感又能学习，一举两得。但是目前由于我没有掌握所有的基础算法，刚学了一般，LeetCode每日一题有的题超纲了，这部分我会在最终学习完之后每天练习。

其次，光修炼了这么久，没有办法学以致用，最终也没法独步江湖。对于学习到的算法怎么能用到工作中这个问题，我还是十分强烈的建议大家都去学习一下王争老师在极客时间的专栏[数据结构与算法之美](https://time.geekbang.org/column/article/39922)的。这个专栏里将所有常见的算法和使用场景做了很详细的解答，并且从字里行间也能看出王争老师的用心之处，光从极客排名第一将近9万的订阅量就可以看出这个专栏的优秀之处。我觉得这是我进阶的最好的学习资料。

最后，我还是希望公司内有经常刷题的大牛组建一个类似于健身社一样的刷题社，每天固定时间打卡刷题，我觉得这种打卡的机制很适合学习这种曲线陡峭但是学会了又会有很大成就感的知识。

### 写在最后

感谢大家可以在百忙中看到这里，希望有兴趣的同学可以早日攻克算法提升内功心法，也希望早有所成的武林大侠带带我们这些想要起步的后辈。这篇总结写的比较匆忙，里面可能会有一些漏洞或者是因为我自身水平问题说错的地方，希望看到的朋友能够帮我指出，多谢大家。

最后，算法的学习是枯燥的，但是如果能坚持下去，他日一定受益匪浅。

> 耐得住寂寞，才能守得住繁华。