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
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<Integer>();
        doPost(root, list);
        return list;
    }

    private void doPost(TreeNode node, List<Integer> list){
        if(node == null){
            return;
        }
        doPost(node.left, list);
        doPost(node.right, list);
        list.add(node.val);
    }
}
```

Solution 2:
这里很重要的一点是，怎么样判断一个节点什么时候是要被打印出来的
```java

```