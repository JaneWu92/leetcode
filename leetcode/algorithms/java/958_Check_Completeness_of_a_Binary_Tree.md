Solution1:  
beat: 94%, 65%
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
        int type;//1: full, 2: nearly full, 3: both not
        int height;
        public Info(int t, int h){
            type = t;
            height = h;
        }
    }
    public boolean isCompleteTree(TreeNode root) {
        Info info = doIsCT(root);
        return info.type == 1 || info.type == 2;
    }
    private Info doIsCT(TreeNode root){
        if(root == null){
            return new Info(1, 0);
        }
        Info info1 = doIsCT(root.left);
        Info info2 = doIsCT(root.right);
        int height = Math.max(info1.height, info2.height) + 1;
        int type = 3;
        if(info1.type == 1 && info2.type == 1){
            if(info1.height == info2.height){
                type = 1;
            }else if(info1.height == info2.height + 1){
                type = 2;
            }
        }else if(info1.type == 1 && info2.type == 2){
            if(info1.height == info2.height){
                type = 2;
            }
        }else if(info1.type == 2 && info2.type == 1){
            if(info1.height == info2.height + 1){
                type = 2;
            }
        }
        return new Info(type, height);
    }
}

```