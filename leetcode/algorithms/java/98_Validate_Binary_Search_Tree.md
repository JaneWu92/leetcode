Solution1:
beat: 36%, 89%  
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
    static class Info{
        int min; 
        int max; 
        boolean isBST;
        public Info(int mi, int ma, boolean is){
            this.min = mi;
            this.max = ma; 
            this.isBST = is;
        }
    }
    public boolean isValidBST(TreeNode root) {
        Info info = doIsValidBST(root);
        if(info == null){
            return true;
        }
        return info.isBST;
    }
    private Info doIsValidBST(TreeNode root){
        if(root == null){
            return null;
        }
        Info info1 = doIsValidBST(root.left);
        Info info2 = doIsValidBST(root.right);
        int min = root.val;
        int max = root.val;
        boolean isBST = true;
        if(info1 != null){
            min = Math.min(info1.min, min);
            max = Math.max(info1.max, max);
            isBST = info1.isBST && isBST && info1.max < root.val;
        }
        if(info2 != null){
            min = Math.min(info2.min, min);
            max = Math.max(info2.max, max);
            isBST = info2.isBST && isBST && root.val < info2.min;
        }
        return new Info(min, max, isBST);
    }        
}
```