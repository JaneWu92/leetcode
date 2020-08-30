Solution1:  
beat: 11%, 8%
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */

class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        HashMap<TreeNode, TreeNode> map = new HashMap<>();
        map.put(root, null);
        inLoopForParentInfo(root, map);
        HashSet<TreeNode> set = new HashSet<>();
        TreeNode cur = p;
        while(cur != null){
            set.add(cur);
            cur = map.get(cur);
        }
        cur = q;
        while(cur != null){
            if(set.contains(cur)){
                return cur;
            }
            cur = map.get(cur);
        }
        return cur;
    }

    private void inLoopForParentInfo(TreeNode root, HashMap<TreeNode, TreeNode> map){
        if(root == null){
            return;
        }else{
            if(root.left != null){
                map.put(root.left, root);
                inLoopForParentInfo(root.left, map);
            }
            if(root.right != null){
                map.put(root.right, root);
                inLoopForParentInfo(root.right, map);
            }
            
        }

    }
}
```
Solution2:  
beat: 64%, 91%.  
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
 // 递归的本质，是最小的问题有人知道，并且怎么由小问题导向大问题也有人知道。
 // 比如你在电影院，想知道自己是第几层，只要问前面的人是几层，再把那个人的层数加一。
 // 前面的人再用同样的方式去问前面的人。直到问到第一排的人。他与其他人不同，他不用问其他人也知道自己在哪一排。
 // 比如像本题，lowest common ancestor, 
 // 
 //
 //
class Solution {
    static class Info{
        TreeNode ancestor;
        boolean findO1;
        boolean findO2;
        public Info(TreeNode n, boolean f1, boolean f2){
            this.ancestor = n;
            this.findO1 = f1;
            this.findO2 = f2;
        }
    }
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        Info res = doLowestCommonAncestor(root, p, q);
        return res.ancestor;
    }
    public Info doLowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null){
            return new Info(null, false, false);
        }
        Info info1 = doLowestCommonAncestor(root.left, p, q);
        Info info2 = doLowestCommonAncestor(root.right, p, q);
        boolean findO1 = root.val == p.val || info1.findO1 || info2.findO1;
        boolean findO2 = root.val == q.val || info1.findO2 || info2.findO2;
        TreeNode ancestor = null;
        if(info1.ancestor != null){
            ancestor = info1.ancestor;
        }else if(info2.ancestor != null){
            ancestor = info2.ancestor;
        }else{
            // both info1 and info2 is null on ancestor
            if(findO1 && findO2){
                ancestor = root;
            }
        }
        return new Info(ancestor, findO1, findO2);
    }
}

```
