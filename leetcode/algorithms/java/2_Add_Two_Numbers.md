Solution 1:  
Pass
```java

/**
 * loop each listnode, and add, maintain a carry int, to keep the carry.
 * each loop should add node1, node2, and carry. and then set carry 0, and set carry in this round of add.
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode n1 = l1;
        ListNode n2 = l2;
        int carry = 0;
        ListNode res = new ListNode(-1);
        ListNode head = res;
        while (n1 != null || n2 != null) {
            int cur;
            if (n1 == null) {
                cur = n2.val + carry;
                n2 = n2.next;
            } else if (n2 == null) {
                cur = n1.val + carry;
                n1 = n1.next;
            } else {
                cur = n1.val + n2.val + carry;
                n1 = n1.next;
                n2 = n2.next;
            }
            carry = cur / 10;
            cur = cur % 10;
            ListNode node = new ListNode(cur);
            res.next = node;
            res = node;

        }
        if(carry != 0){
            ListNode node = new ListNode(carry);
            res.next = node;
        }
        return head.next;
    }
}

class ListNode {
    int val;
    ListNode next;

    ListNode(int x) {
        val = x;
    }
}

```
