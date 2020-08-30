Solution1:
beat 5%, 5%.
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
 // 一旦是这种需要把东西先存着的，不用说肯定要用辅助的栈或队列。
 // 再根据它的特点，是深度优先，所以要用栈，后进先出（广度优先的话就用队列）。
 // 接着就是构建。遇到左括号，证明右边有东西，所以前面的那个东西一定得入栈。
 // 又因为其实这个东西是顺序遍历，不好维护所谓的“前面”的。所以可以默认入栈，遇到左括号的相反项“右括号”再做相反的“出栈”操作。
 // 所以逻辑就是，
 // 1. 遇到数字就建节点，（childFlagcount如果>0，就出栈一个节点，左树不空放左树，否则放右数。childFlagcount--。） 然后入栈。
 // 2. 遇到左括号，就childFlagCount++
 // 3. 遇到右括号，出栈一个数字。
class Solution {
    public TreeNode str2tree(String oris) {
        TreeNode res = null;
        Stack<TreeNode> stack = new Stack<>();
        int leftParCount = 0;
        for(String s : toStringArray(oris)){
            if(isNum(s)){
                TreeNode n = new TreeNode(Integer.valueOf(s));
                res = res == null ? n : res;
                if(leftParCount > 0){
                    TreeNode p = stack.peek();
                    if(p.left == null){
                        p.left = n;
                    }else if(p.right == null){
                        p.right = n;
                    }
                    leftParCount--;
                }
                stack.push(n);
            }else if(s.equals("(")){
                leftParCount++;
            }else if(s.equals(")")){
                stack.pop();
            }
        }
        return res;
        
    }

    private String[] toStringArray(String oris){
        Stack<String> res = new Stack<String>();
        for(char c: oris.toCharArray()){
            if(isPartOfNum(c)){
                if(res.isEmpty()){
                    res.push(String.valueOf(c));
                }else{
                    String pre = res.pop();
                    if(pre.equals("(") || pre.equals(")")){
                        res.push(pre);
                        res.push(String.valueOf(c));
                    }else{
                        pre = pre + c;
                        res.push(pre);
                    }
                }
            }else{
                res.push(String.valueOf(c));
            }
        }
        String[] s = new String[res.size()];
        return res.toArray(s);
    }

    private boolean isPartOfNum(char c){
        return c == '-' || (c >= '0' && c <= '9');
    }

    private boolean isNum(String s){
        return !(s.equals("(") || s.equals(")"));
    }
}

```
Solution2: 
beat: 5%, 47%  
这个时间复杂度已经是线性的啦。怎么才打败5%。
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
 // 一旦是这种需要把东西先存着的，不用说肯定要用辅助的栈或队列。
 // 再根据它的特点，是深度优先，所以要用栈，后进先出（广度优先的话就用队列）。
 // 接着就是构建。遇到左括号，证明右边有东西，所以前面的那个东西一定得入栈。
 // 又因为其实这个东西是顺序遍历，不好维护所谓的“前面”的。所以可以默认入栈，遇到左括号的相反项“右括号”再做相反的“出栈”操作。
 // 所以逻辑就是，
 // 1. 遇到数字就建节点，（childFlagcount如果>0，就出栈一个节点，左树不空放左树，否则放右数。childFlagcount--。） 然后入栈。
 // 2. 遇到左括号，就childFlagCount++
 // 3. 遇到右括号，出栈一个数字。
class Solution {
    public TreeNode str2tree(String oris) {
        TreeNode res = null;
        Stack<TreeNode> stack = new Stack<>();
        int leftParCount = 0;
        String preNumString = "";
        char[] chars = oris.toCharArray();
        // for(String s : toStringArray(oris)){
        for(int i = 0; i < chars.length; i++){
            char c = chars[i];
            String s = null;
            if(isPartOfNum(c)){
                preNumString+=c;
                if(i == chars.length - 1){
                    s = preNumString; 
                }else{
                    continue;
                }
            }else{
                if(preNumString == ""){
                    s = String.valueOf(c);
                }else{
                    s = preNumString;
                    i--;
                    preNumString = "";
                }
            }
            
            if(isNum(s)){
                TreeNode n = new TreeNode(Integer.valueOf(s));
                res = res == null ? n : res;
                if(leftParCount > 0){
                    TreeNode p = stack.peek();
                    if(p.left == null){
                        p.left = n;
                    }else if(p.right == null){
                        p.right = n;
                    }
                    leftParCount--;
                }
                stack.push(n);
            }else if(s.equals("(")){
                leftParCount++;
            }else if(s.equals(")")){
                stack.pop();
            }
        }
        return res;
        
    }

    private boolean isPartOfNum(char c){
        return c == '-' || (c >= '0' && c <= '9');
    }

    private boolean isNum(String s){
        return !(s.equals("(") || s.equals(")"));
    }
}




```
Solution3: 
beat 36%, 88%. 别人的方案。
```java
class Solution {

    public TreeNode str2tree(String s) {
        if (s == null && s.length() == 0) return null;
        Stack<TreeNode> stack = new Stack<>();
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == ')') stack.pop();
            else if (s.charAt(i) >= '0' && s.charAt(i) <= '9' || s.charAt(i) == '-') {
                int start = i;
                //找到根元素的值
                while (i < s.length() - 1 && s.charAt(i + 1) >= '0' && s.charAt(i + 1) <= '9') {
                    i++;
                }
                TreeNode root = new TreeNode(Integer.valueOf(s.substring(start, i + 1)));
                //获取父节点
                if (!stack.isEmpty()) {
                    TreeNode parent = stack.peek();
                    if (parent.left == null) parent.left = root;
                    else parent.right = root;
                }

                //压栈
                stack.push(root);
            }
        }
        if (stack.isEmpty()) return null;
        return stack.peek();
    }
}

```

