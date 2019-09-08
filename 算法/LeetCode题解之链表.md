# LeetCode题解之链表

## 链表求和
### [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

思路：用carry表示进位，需要注意的是，如果最后carry > 0，还需要为carry新建一个节点。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(-1);
        ListNode p = l1, q = l2, cur = dummy;
        int carry = 0;
        
        while (p != null || q != null) {
            int x = p != null ? p.val : 0;
            int y = q != null ? q.val : 0;
            int sum = x + y + carry;
            carry = sum / 10;
            cur.next = new ListNode(sum % 10);
            cur = cur.next;
            
            if (p != null)
                p = p.next;
            if (q != null)
                q = q.next;
        }
        if (carry > 0) {
            cur.next = new ListNode(carry);
        }
        return dummy.next;
    }
}
```

### [445. Add Two Numbers II (Medium)](https://leetcode.com/problems/add-two-numbers-ii/description/)

```
Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 8 -> 0 -> 7
```

思路：链表头代表高位，尾代表低位，考虑用栈来进行一下转换。并且在新建和链表时要用头插法。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Stack<Integer> stack1 = new Stack<Integer>();
        Stack<Integer> stack2 = new Stack<Integer>();
        
        ListNode p = l1, q = l2;
        while (p != null) {
            stack1.push(p.val);
            p = p.next;
        }
        while (q != null) {
            stack2.push(q.val);
            q = q.next;
        }
        
        ListNode dummy = new ListNode(-1);
        int carry = 0;
        while (!stack1.isEmpty() || !stack2.isEmpty()) {
            int x = !stack1.isEmpty() ? stack1.pop() : 0;
            int y = !stack2.isEmpty() ? stack2.pop() : 0;
            int sum = x + y + carry;
            carry = sum / 10;
            ListNode node = new ListNode(sum % 10);
            node.next = dummy.next;
            dummy.next = node;           
        }
        
        if (carry > 0) {
            ListNode node = new ListNode(carry);
            node.next = dummy.next;
            dummy.next = node;  
        }
        
        return dummy.next;
    }
}
```
