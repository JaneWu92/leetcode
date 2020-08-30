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
beat: 