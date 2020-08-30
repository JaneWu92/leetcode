Solution1:  
beat: 100%, 25%
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
        boolean isBST;
        int bstMaxCount;
        int min;
        int max;
        public Info(boolean i, int c, int mi, int ma){
            this.isBST = i;
            this.bstMaxCount = c;
            this.min = mi;
            this.max = ma;
        }
    }
    public int largestBSTSubtree(TreeNode root) {
        Info info = doLargestBSTSubtree(root);
        int res = info == null? 0 : info.bstMaxCount;
        return res;
    }
    private Info doLargestBSTSubtree(TreeNode root){
        if(root == null){
            return null;
        }
        Info info1 = doLargestBSTSubtree(root.left);
        Info info2 = doLargestBSTSubtree(root.right);
        boolean isBST = true;
        int count = 0;
        int min = root.val;
        int max = root.val;
        int lC = info1 == null? 0 : info1.bstMaxCount;
        int rC = info2 == null? 0 : info2.bstMaxCount;
        if(info1 != null){
            isBST = isBST && info1.isBST && info1.max < root.val;
//            count = count + info1.bstMaxCount;
            min = Math.min(min, info1.min);
            max = Math.max(max, info1.max);
        }
        if(info2 != null){
            isBST = isBST && info2.isBST && root.val < info2.min;
//            count = count + info2.bstMaxCount;
            min = Math.min(min, info2.min);
            max = Math.max(max, info2.max);
        }
        if(isBST){
            count = 1 + lC + rC;
        }else{
            count = Math.max(lC, rC);
        }
    //    System.out.println(String.format("%b,%d,%d,%d", isBST, count, min, max));
        return new Info(isBST, count, min, max);
    }
}

```