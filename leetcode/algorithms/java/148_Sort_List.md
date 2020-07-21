Solution 1:  
空间复杂度应该是o(logn)而不是常数
```shell
执行用时：4 ms, 在所有 Java 提交中击败了 60.45% 的用户
内存消耗： 41.5 MB, 在所有 Java 提交中击败了5.88%的用户
```
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        int total_size = size(head);
        int size = total_size >> 1;
        if(total_size == 0 || total_size == 1){
            return head;
        }
        ListNode[] split = splitListNode(head, size);
        ListNode l = sortList(split[0]);
        ListNode r = sortList(split[1]);
        ListNode a = merge(l, r);
        return a;
        
    }

    //input: 2 sorted listnodes; output: 1 sorted listnode
    private ListNode merge(ListNode left, ListNode right){
        
        if(left == null || right == null){
            return left == null ? right : left;
        }
        ListNode head = new ListNode(-1);
        ListNode cur = head;

        while(left!=null && right!=null){
            if(left.val <= right.val){
                cur.next = left; 
                cur = left; 
                left = left.next;
            }else{
                cur.next = right; 
                cur = right; 
                right = right.next;               
            }
        }
        if(left != null){
            cur.next = left;
        }   
        if(right != null){
            cur.next = right;
        }
        return head.next;
    }

    private int size(ListNode head){
        int count = 0;
        while(head!=null){
            count++;
            head = head.next;
        }
        return count;
    }
    private ListNode[] splitListNode(ListNode list, int size){
        ListNode left = list; 
        ListNode left_end = list;
        ListNode right = left;
        for(int i = 0; i< size; i++){
            left_end = right;
            right = right.next;
        }
        left_end.next = null;
        return new ListNode[]{left, right};
    }
}
```
Solution 2:  
改进Solution1的几点:
1. 链表快慢指针找中点
2. merge函数三元表达式的小改变  
但是因为用到递归，空间复杂度是o(logn)而不是题目规定的常数。  
得用非递归的方法应该才可以使空间复杂度到常数项。
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }
        ListNode slow = head, fast = head.next;
        while(fast != null && fast.next != null && fast.next.next != null){
            fast = fast.next.next;
            slow = slow.next;
        }
        fast = slow.next;
        slow.next = null;
        slow = head;
        return merge(sortList(slow), sortList(fast));
    }

    //input: 2 sorted listnodes; output: 1 sorted listnode
    private ListNode merge(ListNode left, ListNode right){
        ListNode help = new ListNode(-1);
        ListNode result = help;
        while(left != null && right != null){
            if(left.val < right.val){
                help.next = left;
                left = left.next;
            }else{
                help.next = right;
                right = right.next;
            }
            help = help.next;
        }
        help.next = left != null ? left : right;
        return result.next;
    }

}
```