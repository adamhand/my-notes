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

### 链表的环
#### [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)
判断一个链表是不是有环。使用快慢两个指针。

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null || head.next == null)
            return false;
        
        ListNode fast = head.next, slow = head;
        while(fast != null && slow != null && fast.next != null){
            if(fast == slow)
                return true;
            fast = fast.next.next;
            slow = slow.next;
        }
        return false;
    }
}
```

#### [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)
找链表环的入口，没有返回`null`。

一种方法需要求得链表长度，另一种方法不需要。分别如下：

```java
public ListNode detectCycle(ListNode head) {
    if(head == null || head.next == null)
        return null;

    ListNode meetingNode = findMeetingNode(head);
    if(meetingNode == null)
        return null;

    ListNode node = meetingNode;
    int nodeInLoop = 1;
    while (node.next != meetingNode){
        node = node.next;
        nodeInLoop++;
    }

    node = head;
    for(int i = 0; i < nodeInLoop; i++)
        node = node.next;

    ListNode node2 = head;
    while (node != node2){
        node = node.next;
        node2 = node2.next;
    }

    return node;
}
```

```java
public ListNode detectCycle(ListNode head) {
    if(head == null || head.next == null)
        return null;
    
    ListNode meetingNode = findMeetingNode(head);
    if(meetingNode == null)
        return null;
    
    ListNode node1 = head;
    ListNode node2 = meetingNode;
    
    while (node1 != node2){
        node1 = node1.next;
        node2 = node2.next;
    }
    
    return node1;
}
```
`findMettingNode`方法如下：

```java
private ListNode findMeetingNode(ListNode head){
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null){
        fast = fast.next.next;
        slow = slow.next;
        if(fast == slow)
            return fast;
    }
    return null;
}
```

## 删除链表中倒数第n个节点
### [19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

```
Given linked list: 1->2->3->4->5, and n = 2.
After removing the second node from the end, the linked list becomes 1->2->3->5.
```
需要注意特殊情况，如果链表中有n个元素，正好要求移除倒数第n个节点(即链表的第一个节点)的时候。可以设置一个dummy节点，dummy.next=head。

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    ListNode fast = dummy;
    ListNode slow = dummy;
    
    for(int i = 0; i <= n; i++){
        fast = fast.next;
    }
    
    while(fast != null){
        fast = fast.next;
        slow = slow.next;
    }
    
    slow.next = slow.next.next;
    
    return dummy.next;
}
```

## 交换链表中的相邻结点
### [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

```java
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

```java
public ListNode swapPairs(ListNode head) {
    if(head == null || head.next == null)
        return head;

    ListNode nHead = new ListNode(0);
    nHead.next = head;
    ListNode cur = nHead;
    while (cur.next != null && cur.next.next != null){
        ListNode left = cur.next;
        ListNode right = cur.next.next;
        left.next = right.next;
        right.next = left;
        cur.next = right;
        cur = cur.next.next;
    }
    return nHead.next;
}
```

可以用递归实现。

```java
public ListNode swapPairs(ListNode head) {
    if ((head == null)||(head.next == null))
        return head;
    ListNode n = head.next;
    head.next = swapPairs(head.next.next);
    n.next = head;
    return n;
}
```

## 链表翻转
### [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

```java
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

使用头插法。
```java
public ListNode reverseList(ListNode head) {
    if(head == null || head.next == null)
        return head;

    ListNode nHead = new ListNode(0);
    ListNode pNode = head;
    while (pNode != null){
        ListNode next = pNode.next;
        pNode.next = nHead.next;
        nHead.next = pNode;
        pNode = next;
    }

    return nHead.next;
}
```
使用递归。

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode next = head.next;
    ListNode newHead = reverseList(next);
    next.next = head;
    head.next = null;
    return newHead;
}
```

### [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

```java
Input: 1->2->3->4->5->NULL, m = 2, n = 4
Output: 1->4->3->2->5->NULL
```
使用栈来做m和n之间的节点的翻转，并用pre和back记录m前一个和n后一个节点。

```java
public ListNode reverseBetween(ListNode head, int m, int n) {
    if(head == null || head.next == null)
        return head;

    Stack<ListNode> stack = new Stack<>();
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode cur = dummy;
    ListNode pre = dummy;
    ListNode back = dummy;
    int i = 1;

    while (i < m){
        pre = pre.next;
        i++;
    }
    cur = pre.next;
    
    while (i <= n){
        stack.push(cur);
        cur = cur.next;
        i++;
    }
    back = cur;
    
    while (!stack.isEmpty()){
        ListNode node = stack.pop();
        pre.next = node;
        pre = pre.next;
    }
    pre.next = back;
    
    return dummy.next;

}
```

## 链表循环移位
### [61. Rotate List](https://leetcode.com/problems/rotate-list/)

```
Input: 1->2->3->4->5->NULL, k = 2
Output: 4->5->1->2->3->NULL
Explanation:
rotate 1 steps to the right: 5->1->2->3->4->NULL
rotate 2 steps to the right: 4->5->1->2->3->NULL
```

```java
public ListNode rotateRight(ListNode head, int k) {
    if (head == null || head.next == null)
        return head;
    ListNode nHead = new ListNode(0);
    nHead.next = head;
    ListNode fast = nHead, slow = nHead;

    int i;
    for(i = 0; fast.next != null; i++)
        fast = fast.next;

    for(int j = i - (k % i); j > 0; j--)
        slow = slow.next;
    
    fast.next = nHead.next;
    nHead.next = slow.next;
    slow.next = null;
    
    return nHead.next;
}
```

## 链表排序
### [148. Sort List](https://leetcode.com/problems/sort-list/)

```
Input: 4->2->1->3
Output: 1->2->3->4
```
要求原地操作。

使用归并排序。

```java
public ListNode sortList(ListNode head) {
    if(head == null || head.next == null)
        return head;

    //将链表进行拆分。
    ListNode fast = head, slow = head, pre = head;
    while(fast != null && fast.next != null){
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    pre.next = null;

    ListNode l1 = sortList(head);
    ListNode l2 = sortList(slow);

    return merge(l1 ,l2);
}

private ListNode merge(ListNode l1, ListNode l2){
    ListNode dummy = new ListNode(0);
    ListNode moving = dummy;

    while (l1 != null && l2 !=  null){
        if(l1.val < l2.val){
            moving.next = l1;
            l1 = l1.next;
        }else {
            moving.next = l2;
            l2 = l2.next;
        }
        moving = moving.next;
    }
    if(l1 != null)
        moving.next = l1;
    if(l2 != null)
        moving.next = l2;

    return dummy.next;
}
```

### [147. Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/)
链表的插入排序。

```java
public ListNode insertionSortList(ListNode head) {
    if(head == null || head.next == null)
        return head;

    ListNode cur = head, next = null;
    ListNode l = new ListNode(0);

    while(cur != null){
        //保存一下下一个节点的值
        next = cur.next;
        ListNode p = l;
        while(p.next != null && p.next.val < cur.val)
            p = p.next;

        //将cur插入到p的后面
        cur.next = p.next;
        p.next = cur;
        cur = next;
    }
    return l.next;
}
```

## 分隔链表
### [86. Partition List](https://leetcode.com/problems/partition-list/)

```
Input: head = 1->4->3->2->5->2, x = 3
Output: 1->2->2->4->3->5
```
将链表分为两部分，一部分小于3，另一部分大于3。

```java
public ListNode partition(ListNode head, int x) {
    if(head == null || head.next == null)
        return head;
    
    ListNode dummy1 = new ListNode(0);
    ListNode dummy2 = new ListNode(0);
    ListNode cur1 = dummy1, cur2 = dummy2;
    while (head != null){
        if(head.val < x){
            cur1.next = head;
            cur1 = head;
        }else {
            cur2.next = head;
            cur2 = head;
        }
        head = head.next;
    }
    cur2.next = null;
    cur1.next = dummy2.next;
    
    return dummy1.next;
}
```

### 回文链表
[234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

```
Input: 1->2->2->1
Output: true
```
要求O(n)的空间和O(1) 的空间复杂度。切成两半，把后半段反转，然后比较两半是否相等。

```java
public static boolean isPalindrome(ListNode head) {
    if(head == null || head.next == null)
        return true;
    
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null){
        fast = fast.next.next;
        slow = slow.next;
    }
    
    //链表个数为奇数
    if(fast != null){
        slow = slow.next;
    }
    
    slow = reverse(slow);
    fast = head;
    
    while (slow != null){
        if(fast.val != slow.val)
            return false;
        
        slow = slow.next;
        fast = fast.next;
    }
    
    return true;
}

//就地翻转即可
private static ListNode reverse(ListNode head){
    ListNode nHead = new ListNode(0);
    ListNode temp = head;
    while (temp != null){
        ListNode next = temp.next;
        temp.next = nHead.next;
        nHead.next = temp;
        temp = next;
    }

    return nHead.next;
}
```

## 链表元素按奇偶聚集
### [328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/)

```
Input: 1->2->3->4->5->NULL
Output: 1->3->5->2->4->NULL
```

```java
public ListNode oddEvenList(ListNode head) {
    if (head == null) {
        return head;
    }
    ListNode odd = head, even = head.next, evenHead = even;
    while (even != null && even.next != null) {
        odd.next = odd.next.next;
        odd = odd.next;
        even.next = even.next.next;
        even = even.next;
    }
    odd.next = evenHead;
    return head;
}
```