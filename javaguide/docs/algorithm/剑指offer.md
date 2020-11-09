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











