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
    List<Integer> list = new ArrayList<Integer>();

    public List<Integer> preorderTraversal(TreeNode root) {
        doPreorder(root);
        return list;
    }

    public void doPreorder(TreeNode root){
        //head, left, right
        if(root == null){
            return;
        }
        list.add(root.val);
        preorderTraversal(root.left);
        preorderTraversal(root.right);
    }
}

```

Solution 2:
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

    public List<Integer> preorderTraversal(TreeNode root) {
        List res = new ArrayList<Integer>();
        if(null == root){
            return res;
        }
        Stack<TreeNode> stack = new Stack<TreeNode>();
        stack.push(root);
        while(! stack.isEmpty()){
            TreeNode n = stack.pop();
            res.add(n.val);
            if(null != n.right){
                stack.push(n.right);
            }
            if(null != n.left){
                stack.push(n.left);
            }
        }
        return res;
    }

}
```