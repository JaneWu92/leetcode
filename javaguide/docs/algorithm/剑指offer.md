## DFS

剑指 Offer 27. 二叉树的镜像
深度优先遍历这棵树
每个节点在做子节点的时候都左右互换

剑指 Offer 28. 对称的二叉树
这个可以用深度优先。
只是从对称来说，要看成，两颗子树的left和right是反过来的
所以可以
dfs_isSymmetric(TreeNode node1, TreeNode node2)
    boolean res0 = node1.val == node.val
    boolean res1 = dfs(node1.left, node2.right)
    boolean res2 = dfs(node1.right, node2.left)
    return res0 && res1 && res2
    


剑指 Offer 54. 二叉搜索树的第k大节点
注意是二叉搜索树，所以左边的都是小于右边的
因为不能知道左树和右树的数量，所以应该是做不到节省掉一些计算的
应该还是得老老实实把每个的顺序给算出来。
所以就是老老实实中序遍历树，然后输出第k大。



剑指 Offer 55 - I. 二叉树的深度
广度优先
唉呀这个是在DFS的专辑下面，得用深度优先做一下。




## 链表

剑指 Offer 22. 链表中倒数第k个节点
一开始的思路是，
快慢指针，然后快的走到末尾，然后再来算正的要几步
现在一想，什么东西啊不可理喻。
这个东西叫什么快慢指针，跟你就遍历一遍链表，算出总数，再正向走一遍不是一样吗。
===========好在看了题解
就是两个指针，第一个先走k个，然后两个指针等距地向前进
等到一个到达了尾部，中间的那个到末尾的距离就是两个指针之间的距离，k。
。。。


剑指 Offer 24. 反转链表
可以用这个思路，就是一个是反转链慢慢添加
1 2 3 4 5
2，3，4，5 加到1前面 => 2,1 和 3，4，5
3，4，5 加到 2,1前面 => 3，2，1 和4，5



剑指 Offer 25. 合并两个排序的链表
维持三个Node
一条是结果，另外两个是另外两个节点
有一个需要注意的是，为了方便操作，可以做一个dummy的head（不为null）
因为判空处理很麻烦。



剑指 Offer 52. 两个链表的第一个公共节点
两个链表的公共节点。可以用hashmap
但是条件说了O(1)的空间复杂度
感觉可以用快慢指针，一个走两步，一个走一步，
最后总会相遇。
胡扯。。。。。不知道开口就胡说八道
============
这里的解题思路应该是，如果能知道两个链表的长度差。
那我把长度比较长的那段剪掉，两个人一起走，边走边判断相等性。
第一个相等的就是这个公共节点。
所以关键在于怎么找这个长出来的那段。
就是两个指针一起走，短的肯定很快就到头。
然后长的剩下的那段，就是他们的相差数
所以再来一个指针，从长的头开始走这么一段路，就能走到跟短的一样的起点
========这个思路很清晰了。根据这个思路也完成了这道题。
但看了题解，发现有超短的代码如下：
虽然很优雅。但我觉得是比较不那么“非常自然”能够想到的写法。可做参考。
```aidl
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode curA = headA;
        ListNode curB = headB;

        while (curA != curB) {
            curA = curA != null ? curA.next : headB;
            curB = curB != null ? curB.next : headA;
        }
        return curA;
    }
```


## 树

剑指 Offer 68 - I. 二叉搜索树的最近公共祖先
用这个思路，假设我的两个子节点能够给我提供信息，我要什么信息才能够足够给出我的决策。
这个信息应该是，p和q是不是分居我的两个子树。
static class Info
    int type => 1 all in left tree, 2 all in right tree 3 in both tree

Info dfs(Node root, Node a, Node b)
    Info info1 = dfs(root.left, a, b);
    Info info2 = dfs(root.right, a, b);
    Info resultInfo = new Info();
    if((info1 == 1 && info2 == 2) || (info1 == 2 && info2 == 1))
        resultInfo.type = 3
    if(info1 == 3 )

我感觉这种套路不能解决这种找节点的问题，只能解决那种，判断是不是的问题
是不是平衡二叉树，之类的
参考(## 二叉树的递归套路)
===========但我找到一个很好的解答，我觉得就是一样的思路。
https://www.bilibili.com/video/BV1Y4411Y7v3?from=search&seid=12530901791641252630
重新做一下。
要从两个子节点那边拿到的信息是，我这个子树里面，有没有我们的pq任意一个
因为递归的性质，是从最底层开始的。所以第一个找到两个子树返回的信息里，都不是null的
就是这个lowest common ancestor。
所以说这里的信息，就是一个节点。
这个节点代表，在以我这个节点为头的子树下，是出现过我们的target的，无论一个还是两个。
但是这个有点奇怪，我觉得不够直观把。还是有点刻意去设计的，先实现一下，再用其他办法。
```
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if(left == null) return right;
        if(right == null) return left;
        return root;
    }
```
再用更common的思路实现一下






## 二叉树的递归套路
* 可以解决绝大多数的二叉树问题尤其是树型dp问题
* 本质是里利用递归遍历二叉树的便利性
* 题目
    * 给定一棵二叉树的头节点head，返回这颗二叉树是不是平衡二叉树
    * 给定一棵二叉树的头节点head，任何两个节点之间都存在距离，返回整颗二叉树的最大距离
    * 派对最大快乐值
* 套路
    * 先设定一个dfs方法要返回的信息
    * 这个信息是假设我能从我的两个子树都拿到信息
    * 我通过这些信息，加工出属于我当前的节点的信息，然后返回
    * 模板
        * 出口
        * 左右两个节点拿信息
        * 头节点根据当前这个节点加工信息
















