# LeetCode题解
---

# 基本算法
## 递归
#### [105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

## 动态规划
#### [198. House Robber](https://leetcode.com/problems/house-robber/)
**描述**：假如有一个专业盗贼计划在一条路上盗取房屋钱财，每一个房屋都藏着一些钱。但是相邻房屋之间有安全系统相连，安全系统与警察相连，所以在同一晚上不能偷盗相邻的房屋。

给一个非负整数代表每一个房屋的藏的钱数，求在没有惊动警察的情况下，能盗取的最多的钱数。

**分析**：很显然，当面对房屋`i`时，有两种选择：偷或不偷。这个选择会影响后面的选择，所以可以考虑使用动态规划。

如果面对房屋`i`时，选择：

- 偷。这时就不能偷取`i-1`的房子，而`i-2`和之前的房子可以再次做选择
- 不偷。这时`i-1`和之前的房子都可以再次做选择

状态转移方程为：
`rob(i) = Math.max(rob(i - 2) +currentHousrValue, rob(i - 1))`

`rob(i)`表示面对房屋`i`时能够偷得的最大钱数。

**第一种写法，使用自顶向下的递归**：
```java
public int rob(int[] nums) {
    return rob(nums, nums.length - 1);
}
private int rob(int[] nums, int i){
    if (i <0){
        return 0;
    }
    return Math.max(rob(nums, i - 2) + nums[i], rob(nums, i - 1));
}
```
这种方法的复杂度太高，说不定还有栈溢出。

**第二种写法，使用自顶向下的递归+memo**：
这种方法的时间和空间复杂度都为O(n)。
```java
int[] memo;
public int rob(int[] nums){
    memo = new int[nums.length + 1];
    Arrays.fill(memo, -1);
    return rob(nums, nums.length - 1);
}
private int rob(int[] nums, int i){
    if (i < 0){
        return 0;
    }
    if (memo[i] >= 0){
        return memo[i];
    }
    int result = Math.max(rob(nums, i - 2) + nums[i], rob(nums, i - 1));
    memo[i] = result;
    return result;
}
```

**第三种写法，自底向上的递推+memo**：
```java
public int rob(int[] nums){
    if (nums == null || nums.length == 0)
        return 0;
    int[] memo = new int[nums.length + 1];
    memo[0] = 0;
    memo[1] = nums[0];
    for (int i = 1; i < nums.length; i++){
        int value = nums[i];
        memo[i + 1] = Math.max(memo[i - 1] + value, memo[i]);
    }
    return memo[nums.length];
}
```

**第四种写法，继续对空间进行优化**：
因为考虑到写法三种的`memo`其实只用了两个数，就不必申请一个数组。当前房屋为`i`，则`pre2`代表`i-2`的房屋，`pre1`代表`i-1`的房屋。
```java
public int rob(int[] nums){
    if (nums == null || nums.length == 0)
        return 0;
    int pre1 = 0, pre2 = 0;
    for (int num : nums){
        int temp = pre1;
        pre1 = Math.max(pre2 + num, pre1);
        pre2 = temp;
    }
    return pre1;
}
```

# 数据结构中的算法
---

## 链表
### 单链表
---
#### [328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/)
---

**描述：**
将单链表中索引为奇数的节点放在一链表的前半部分，索引为偶数的节点放在链表的后半部分。不改变原链表节点之间的顺序。

要求所有操作在原地进行，并且算法的时间复杂度和空间复杂度在`O(nodes)`和`O(1)`内。
```
Example 1:

Input: 1->2->3->4->5->NULL
Output: 1->3->5->2->4->NULL
Example 2:

Input: 2->1->3->5->6->4->7->NULL
Output: 2->3->6->7->1->5->4->NULL
```

**思路：**
使用两个游标`odd`和`even`，分别指向第一个奇数节点和第一个偶数节点，然后交替向后移动，通过这种移动可以将链表分为两个分叉，一个分叉包含奇数节点，另一个分叉包含偶数节点。这种移动方法很向人的两条腿走路，所以可以形象地称之为“**左右腿法**”。

如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/odd_even.PNG">
</center>

代码如下：
```
public class Solution {
    public ListNode oddEvenList(ListNode head) {
        if(head == null)
            return head;

        ListNode odd = head, even = head.next, evenHead = even;
        while (even != null && even.next != null){
            //像两条腿走路一样
            odd.next = even.next;
            odd = odd.next;
            even.next = odd.next;
            even = even.next;
        }
        odd.next = evenHead;
        return head;
    }
}
```

#### [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

**描述：**
```
Given an array nums containing n + 1 integers where each integer is between 1 and n (inclusive), prove that at least one duplicate number must exist. Assume that there is only one duplicate number, find the duplicate one.
```
Example 1:
```
Input: [1,3,4,2,2]
Output: 2
```
Example 2:
```
Input: [3,1,3,4,2]
Output: 3
```
Note:

- You must not modify the array (assume the array is read only).
- You must use only constant, O(1) extra space.
- Your runtime complexity should be less than O(n2).
- There is only one duplicate number in the array, but it could be repeated more than once.

**思路一：**二分法+抽屉原理 **时间 O(NlogN) 空间 O(1)**
可以参考[链接1](https://segmentfault.com/a/1190000003817671) [链接2](https://www.cnblogs.com/puyangsky/p/5271778.html) [链接3](https://www.cnblogs.com/grandyang/p/4843654.html)
但是感觉解释的都不是特别正确，然而代码又是正确的。下面先贴出正确的代码，后面再看吧。、

```
public int findDuplicate(int[] nums) {
    int min = 0, max = nums.length - 1;
    while(min <= max){
        int mid = min + (max - min) / 2;
        int cnt = 0;
        for(int i = 0; i < nums.length; i++){
            if(nums[i] <= mid){
                cnt++;
            }
        }
        if(cnt > mid){
            max = mid - 1;
        } else {
            min = mid + 1;
        }
    }
    return min;
}
```

**思路二：**链表找环法 **时间 O(N) 空间 O(1)**
假设数组中没有重复，那我们可以做到这么一点，就是将数组的下标和1到n每一个数一对一的映射起来。比如数组是`[213]`,则映射关系为`0->2, 1->1, 2->3`。假设这个一对一映射关系是一个函数`f(n)`，其中n是下标，`f(n)`是映射到的数。如果我们从下标为0出发，根据这个函数计算出一个值，以这个值为新的下标，再用这个函数计算，以此类推，直到下标超界。实际上可以产生一个类似链表一样的序列。比如在这个例子中有两个下标的序列，`0->2->3`。

但如果有重复的话，这中间就会产生多对一的映射，比如数组`[2131]`,则映射关系为`0->2, {1，3}->1, 2->3`。这样，我们推演的序列就一定会有环路了，这里下标的序列是`0->2->3->1->1->1->1->...`，而环的起点就是重复的数。

所以该题实际上就是**找环路起点的题**。我们先用快慢两个下标都从0开始，快下标每轮映射两次，慢下标每轮映射一次，直到两个下标再次相同。这时候保持慢下标位置不变，再用一个新的下标从0开始，这两个下标都继续每轮映射一次，当这两个下标相遇时，就是环的起点，也就是重复的数。

注意，上面使用了一个环判定算法：**`floyd`判环算法**(`Floyd cycle detection`)，就是通常所说的**快慢指针法**。

```
public int findDuplicate(int[] nums){
    int slow = 0;
    int fast = 0;
    //找到快慢指针相遇的地方
    do{
        slow = nums[slow];
        fast = nums[nums[fast]];
    }while (slow != fast);
    int find = 0;
    //找“环”的起点
    while (find != slow){
        slow = nums[slow];
        find = nums[find];
    }
    return find;
}
```
**参考**[链接1](https://segmentfault.com/a/1190000003817671)

#### [202. Happy Number](https://leetcode.com/problems/happy-number/)
**描述：**
```
Write an algorithm to determine if a number is "happy".

A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or it loops endlessly in a cycle which does not include 1. Those numbers for which this process ends in 1 are happy numbers.
```
Example: 
```
Input: 19
Output: true
Explanation: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

**思路一：**
如果按照上述方法，最后停止时结果为1，就是快乐数，否则，就会先放入一个循环，也就是说，前面出现过的数在后面又会重复出现。这就想到可以使用`Java`的`HashSet`，因为它其中存放的元素是不能重复的，如果将某个数放入`HashSet`中时，发现其中已经有了，说明陷入了循环，该数不是快乐数。

```
public boolean isHappy(int n) {
    Set<Integer> set = new HashSet<>();
    while (n != 1 && !set.contains(n)){
        set.add(n);
        int sum = 0;
        while (n > 0){
            int remain = n % 10;
            sum += remain * remain;
            n /= 10;
        }
        n = sum;
    }
    return n == 1;
}
```

**思路二：**
由上面的思路，容易联想到“链表找环法”，可以将这个问题转换为一个链表问题，使用`287`题中的**弗洛伊德判环法**。
```
public boolean isHappy(int n) {
    int slow, fast;
    slow = fast = n;
    do{
        slow = digit(slow);
        fast = digit(digit(fast));
    }while (slow != fast);

    return slow == 1;
}

public int digit(int n){
    int sum = 0;
    while (n > 0){
        int remain = n % 10;
        sum += remain * remain;
        n /= 10;
    }
    return sum;
}
```

---

## 树
### BST
#### [108. Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)
---

**描述：**
将一个给定的有序数组转变成一个平衡二叉搜索树。
```
Given the sorted array: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5
```
**思路：**二叉查找树之所以叫“二叉”是因为它使用二叉搜索的方法来进行查找。中序遍历二叉查找树会得到一个有序数组。所以，如果将一个有序数组看成一棵树（以中点为根，左右为左右子树，依次下去），数组就相当于一棵二叉查找树**。如果要构造这棵树，那就是把中间元素转化为根，然后递归构造左右子树。**
```java
public TreeNode sortedArrayToBST(int[] nums) {
    int left = 0, right = nums.length - 1;

    return helper(nums, left, right);
}

private TreeNode helper(int[] nums, int left, int right){
    if(left > right)
        return null;

    int mid = left + (right - left) / 2;
    TreeNode node = new TreeNode(nums[mid]);
    node.left = helper(nums, left, mid - 1);
    node.right = helper(nums, mid + 1, right);

    return node;
}
```

#### [109. Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)
---

**描述：**
将一个给定的有序数组转变成一个平衡二叉搜索树。
```
Given the sorted linked list: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5
```

**思路：**和上面的题目思路一样，不同之处是对于链表无法用下标的方式访问中间元素。可以按照中序遍历的顺序对链表进行访问，访问的顺序就是二叉搜索树中序遍历的顺序。
```
private ListNode head;

private int countSize(ListNode head){
    int count = 0;
    ListNode cur = head;
    while (cur != null){
        cur = cur.next;
        count++;
    }
    return count;
}

private TreeNode convertListToBST(int l, int r){
    if(l > r)
        return null;

    int mid = (l + r) / 2;

    //左子树
    TreeNode left = convertListToBST(l, mid - 1);
    //根
    TreeNode root = new TreeNode(head.val);
    System.out.println(head.val);
    root.left = left;

    head = head.next;
    //右子树
    root.right = convertListToBST(mid + 1, r);

    return root;
}

public TreeNode sortedListToBST(ListNode head) {
    int size = countSize(head);
    this.head = head;

    return convertListToBST(0, size - 1);
}
```

# 数学
---
#### [172. Factorial Trailing Zeroes](https://leetcode.com/problems/factorial-trailing-zeroes/)

**描述：**
```
Given an integer n, return the number of trailing zeroes in n!.
```
Example 1:
```
Input: 3
Output: 0
Explanation: 3! = 6, no trailing zero.
```
Example 2:
```
Input: 5
Output: 1
Explanation: 5! = 120, one trailing zero.
```
Note: Your solution should be in logarithmic time complexity.

**思路一：**
直接求阶乘。这种做法可能出现大数问题，可能溢出。
```
public int trailingZeroes(int n) {
    int sum = 1;
    for (int i = 1; i <= n; i++) {
        sum *= i;
    }

    int count = 0;
    while ((sum % 10) == 0) {
        sum /= 10;
        count++;
    }
    return count;
}
```

**思路二：**
根据数学原理。

考虑n!的质数因子。后缀`0`总是由质因子`2`和质因子`5`相乘得来的，如果我们可以计数`2`和`5`的个数，问题就解决了。

考虑例子：`n = 5`时，`5!`的质因子中`(2 * 2 * 2 * 3 * 5)`包含一个`5`和三个`2`。因而后缀`0`的个数是`1`。
`n = 11`时，`11!`的质因子中`((2 ^ 8) * (3 ^ 4) * (5 ^ 2) * 7)`包含两个`5`和八个`2`。于是后缀`0`的个数就是`2`。

很容易观察到质因子中`2`的个数总是大于等于`5`的个数,因此只要计数`5`的个数即可。如何计算`5`的个数呢？

比如，`n=25`。计算`n/5`，得到`5`，即包含`5`个`5`，分别来自其中的`5`, `10`, `15`, `20`, `25`，但是在`25中其实是包含`2`个5`的，所以除了计算`n/5`， 还要计算`n/5/5`, `n/5/5/5`, `n/5/5/5/5, ..., n/5/5/5,,,/5`直到商为`0`，然后就和，就是最后的结果。

可以使用递归的方法或普通方法求解：
```
//递归方法
public int trailingZeroes(int n) {
    return n == 0 ? 0 : n / 5 + trailingZeroes(n / 5);
}
//普通方法
public int trailingZeroes(int n) {
    int count = 0;

    while (n != 0) {
        count += n / 5;
        n /= 5;
    }

    return count;
}
```

# 字符串

## 最长子串系列(`longest substring`)

### [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

```
Given a string, find the length of the longest substring without repeating characters.
```

Example 1:
```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
```
Example 2:
```
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```
Example 3:
```
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 

Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

> 可以使用`HashSet`配合两个指针使用。两个指针一前一后，前面的指针所指的字符如果不在`set`中，就将其放入，并且指针后移一位，然后比较当前前后指针之间的距离和已知的最大距离，取较大值；
> 如果前面指针所指的字符在`set`中，就将后面的指针所指的字符在`set`中删除，后面的指针右移一位。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s == null)
            return -1;
        HashSet<Character> set = new HashSet<Character>();
        int i = 0, j = 0;
        int ans = 0;
        while (j < s.length()) {
            if (!set.contains(s.charAt(j))) {
                set.add(s.charAt(j++));
                ans = Math.max(ans, j - i);
            } else {
                set.remove(s.charAt(i++));
            }
        }
        return ans;
    }
}
```

### [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/submissions/)
```
Given an array A of positive integers, call a (contiguous, not necessarily distinct) subarray of A good if the number of different integers in that subarray is exactly K.

(For example, [1,2,3,1,2] has 3 different integers: 1, 2, and 3.)

Return the number of good subarrays of A.
```
Example 1:
```
Input: A = [1,2,1,2,3], K = 2
Output: 7
Explanation: Subarrays formed with exactly 2 different integers: [1,2], [2,1], [1,2], [2,3], [1,2,1], [2,1,2], [1,2,1,2].
```
Example 2:
```
Input: A = [1,2,1,3,4], K = 3
Output: 3
Explanation: Subarrays formed with exactly 3 different integers: [1,2,1,3], [2,1,3], [1,3,4].


Note:

1 <= A.length <= 20000
1 <= A[i] <= A.length
1 <= K <= A.length
```

> 和上题有类似之处，但是此题中，相同的字符算做出现一次，所以考虑使用`HashMap`。

```java
public int subarraysWithKDistinct(int[] A, int K) {
    return atMostK(A, K) - atMostK(A, K - 1);
}
int atMostK(int[] A, int K) {
    int i = 0, res = 0;
    Map<Integer, Integer> count = new HashMap<>();
    for (int j = 0; j < A.length; ++j) {
        if (count.getOrDefault(A[j], 0) == 0) K--;
        count.put(A[j], count.getOrDefault(A[j], 0) + 1);
        while (K < 0) {
            count.put(A[i], count.get(A[i]) - 1);
            if (count.get(A[i]) == 0) K++;
            i++;
        }
        res += j - i + 1;
    }
    return res;
}
```

下面这个更好理解，但是运行时间超过限制。

```java
public int subarraysWithKDistinct(int[] A, int K) {
    if (A == null || A.length == 0)
        return 0;
    int ans = 0, curLen = 0;
    int i = 0, j = 0;
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    while (j < A.length) {
        map.put(A[j], map.getOrDefault(A[j], 0) + 1);
        if (map.size() > K) {
            map.clear();
            i++;
            j = i;
            continue;
        }
        
        if (map.size() == K) {
            ans++;
            j++;
            if (j == A.length) {
                map.clear();
                i++;
                j = i;
            }
            continue;
        }
        j++;
    }
    return ans;
}
```

### [14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)
```
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".
```
Example 1:
```
Input: ["flower","flow","flight"]
Output: "fl"
```
Example 2:
```
Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
Note:

All given inputs are in lowercase letters a-z.
```

> 首先想到的就是遍历字符串数组的每一个字符串，两两比较求得前缀。

```java
public String longestCommonPrefix(String[] strs) {
    if (strs == null || strs.length == 0)
        return "";

    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (strs[i].indexOf(prefix) != 0) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.length() == 0)
                return "";
        }
    }
    return prefix;
}
```

> 可以考虑使用分治的方法。示意图如下所示。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/14_lcp_diviso_et_lmpera.png">
</div>

```java
public String longestCommonPrefix(String[] strs) {
    if (strs == null || strs.length == 0)
        return "";
    
    return longestCommonPrefix(strs, 0, strs.length - 1);
}

private String longestCommonPrefix(String[] strs, int l, int r) {
    if (l == r) {
        return strs[l];
    }
    else {
        int mid = (l + r) / 2;
        String leftStrs = longestCommonPrefix(strs, l, mid);
        String rightStrs = longestCommonPrefix(strs, mid + 1, r);
        return commonPrefix(leftStrs, rightStrs);
    }
}

private String commonPrefix(String s1, String s2) {
    int minLen = Math.min(s1.length(), s2.length());
    for (int i = 0; i < minLen; i++) {
        if (s1.charAt(i) != s2.charAt(i)) {
            return s1.substring(0, i);
        }
    }
    return s1.substring(0, minLen);
}
```

### [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)
```
Given a string S and a string T, count the number of distinct subsequences of S which equals T.

A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, "ACE" is a subsequence of "ABCDE" while "AEC" is not).
```
Example 1:
```
Input: S = "rabbbit", T = "rabbit"
Output: 3
Explanation:

As shown below, there are 3 ways you can generate "rabbit" from S.
(The caret symbol ^ means the chosen letters)

rabbbit
^^^^ ^^
rabbbit
^^ ^^^^
rabbbit
^^^ ^^^
```
Example 2:
```
Input: S = "babgbag", T = "bag"
Output: 5
Explanation:

As shown below, there are 5 ways you can generate "bag" from S.
(The caret symbol ^ means the chosen letters)

babgbag
^^ ^
babgbag
^^    ^
babgbag
^    ^^
babgbag
  ^  ^^
babgbag
    ^^^
```

> 考虑使用动态规划来解。可以参考[这里](https://leetcode.com/problems/distinct-subsequences/discuss/37327/Easy-to-understand-DP-in-Java)。


### [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
```
Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:

Open brackets must be closed by the same type of brackets.
Open brackets must be closed in the correct order.
Note that an empty string is also considered valid.
```
```
Example 1:

Input: "()"
Output: true
Example 2:

Input: "()[]{}"
Output: true
Example 3:

Input: "(]"
Output: false
Example 4:

Input: "([)]"
Output: false
Example 5:

Input: "{[]}"
Output: true
```

> 使用栈。遇到左括号就入栈，遇到右括号就出栈。比较当前字符和出栈字符是否能够组成一对括号。

```java
class Solution {
    public boolean isValid(String s) {
        if (s == null)
            return false;
        if (s.length() == 0)
            return true;
        
        Map<Character, Character> map = new HashMap<Character, Character>();
        map.put(')', '(');
        map.put('}', '{');
        map.put(']', '[');
        
        Stack<Character> stack = new Stack<Character>();
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (map.containsKey(ch)) {
                char temp = stack.isEmpty() ? '#' : stack.pop();
                if (map.get(ch) != temp)
                    return false;
            } else {
                stack.push(ch);
            }
        }
        return stack.isEmpty();
    }
}
```

### [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/)

### [32. Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)

### [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)
```
Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

For example, given n = 3, a solution set is:

[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

> 采用回溯法。

```java
public List<String> generateParenthesis(int n) {
    List<String> ans = new ArrayList<>();
    backtracking(ans, "", 0, 0, n);

    return ans;
}

public void backtracking(List<String> ans, String cur, int open, int close, int max){
    if (cur.length() == 2 * max){
        ans.add(cur);
        return;
    }

    if (open < max){
        backtracking(ans, cur + "(", open + 1, close, max);
    }
    if (close < open){
        backtracking(ans, cur + ")", open, close + 1, max);
    }
}
```

如果使用`StringBuilder`，每次要删除最后一个元素，因为传入的`s`参数始终是一个。而使用`String`则不需要，因为每次都会生成一个新的字符串。

```java
public List<String> generateParenthesis(int n) {
    List<String> ans = new ArrayList<String>();
    StringBuilder sb = new StringBuilder();
    backtracking(ans, sb, 0, 0, n);
    return ans;
}

private void backtracking(List<String> ans, StringBuilder s, int open, int close, int n) {
    if (s.length() == 2 * n) {
        ans.add(s.toString());
        return;
    }
    
    if (open < n){
        backtracking(ans, s.append("("), open + 1, close, n);
        s.deleteCharAt(s.length() - 1);
    }
    if (close < open){
        backtracking(ans, s.append(")"), open, close + 1, n);
        s.deleteCharAt(s.length() - 1);
    }
}
```