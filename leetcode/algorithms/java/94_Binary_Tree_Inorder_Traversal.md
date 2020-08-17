Solution 1:
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
    List res = new ArrayList<Integer>();
    public List<Integer> inorderTraversal(TreeNode root) {
        in(root);
        return res;
    }
    private void in(TreeNode root){
        if(null == root){
            return;
        }
        in(root.left);
        res.add(root.val);
        in(root.right);
    }
}
```
Solution 2:
这里面的一个子树的子问题的代码复用，其实是用这个node节点来完成。
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
    public List<Integer> inorderTraversal(TreeNode root) {
        List res = new ArrayList<Integer>();
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode node = root;
        while(true){
            //左中右
            while(node != null){
                stack.push(node);
                node = node.left;    
            }
            if(stack.isEmpty()){
                break;
            }
            node = stack.pop();
            res.add(node.val);
            node = node.right;
        }
        return res;
    }

}
```