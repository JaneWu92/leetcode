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
    public ListNode insertionSortList(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }
        //cur is the node that need to be inserted into previous list.
        ListNode result_head = new ListNode(-1);
        result_head.next = head;
        ListNode cur = head.next;
        ListNode pre = head;
        ListNode next = cur.next;
        ListNode help = result_head;

        while(cur != null){
            help = result_head;
            if(pre.val <= cur.val){
                pre.next = cur; 
                pre = cur;
                cur = next;
            }else{
                while(help.next.val <= cur.val){
                    help = help.next;
                }
                cur.next = help.next;
                help.next = cur;
                cur = next;
            }
            if(next != null) next = next.next;
        }
        pre.next = null;
        return result_head.next;
    }
}
```