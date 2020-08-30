Solution1:
beat: 8%, 28%
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
 // 后序遍历：左右头。当到叶子节点的时候，判断。
 // 然后返回值的时候就返回子树头节点，如果是null，就要对当前的节点再做子节点的判断。
class Solution {
    public TreeNode removeLeafNodes(TreeNode root, int target) {
        return pos(root, target);
    }
    private TreeNode pos(TreeNode head, int target){
        TreeNode l = null;
        TreeNode r = null;
        if(isLeaf(head)){
            if(head. val == target){
                return null;
            }else{
                return head;
            }
        }
        if(head.left != null){
            l = pos(head.left, target);
        }
        if(head.right != null){
            r = pos(head.right, target);
        }
        head.left = l;
        head.right = r;
        if(l == null && r == null && head.val == target){
            head = null;
        }
        return head;
    }
    private boolean isLeaf(TreeNode head){
        return head.left == null && head.right == null;
    }
}
```
Solution2: 
beat: 100%, 65%  
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
 // 后序遍历：左右头。当到叶子节点的时候，判断。
 // 然后返回值的时候就返回子树头节点，如果是null，就要对当前的节点再做子节点的判断。
class Solution {
    public TreeNode removeLeafNodes(TreeNode root, int target) {
        return pos(root, target);
    }
    private TreeNode pos(TreeNode head, int target){
        if(head == null){
            return null;
        }
        head.left = pos(head.left, target);
        head.right = pos(head.right, target);
        if(head.left == null && head.right == null && head.val == target){
            head = null;     
        }
        return head;
    }
}
```