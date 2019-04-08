# LeetCode笔记之数组(一)
---
# 1. Sum
## 1.1 Two Sum
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/TwoSum.PNG">
</center>

- 解法1:最容易想到的方法就是两个for循环嵌套，但这显然不是最简单的方法。

```
public int[] twoSum(int[] nums, int target) {
        for(int i = 0; i < nums.length; i++)
        {
            for(int j = i + 1; j < nums.length; j++)
            {
                if(nums[i] + nums[j] == target)
                {
                    return new int[] {i, j};
                }
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
```

- 解法2:使用HashMap存储键值对信息，其中"值"就是题目中要返回的"位置"。

```
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for(int i = 0; i < nums.length; i++){
        int component = target - nums[i];    //需要构造一个A+B=target的关系
        if(!map.containsKey(component)){
            map.put(nums[i], i);
        }else{
            return new int[] {map.get(component), i};
        }
    }
    throw new IllegalArgumentException("no solution");
}
```

- 解法3:使用***双指针***。但是这种方法需要对数组进行排序，改变了原来的位置，而题目要求返回的是位置而不是数，所以这个方法并不适合与该题。如果题目要求返回的是数，可以用这种方法。
解法的思路如下：首先对输入的数组有小到大进行排序，然后用两个指针分别从数组头和数组尾开始遍历，如果两个数的和正好等于target就返回（因为题目表明如果有满足条件的数，只能有一组）；如果两数之和小于target，让左指针加一，反之右指针加一。

```
//这里要求返回位置而不是数，所以这种方法不能用，因为排了序，位置改变了
public int[] twoSum(int[] nums, int target) {
    Arrays.sort(nums);
    int lo = 0, hi = nums.length - 1;
    while(lo < hi){
        if(nums[lo] + nums[hi] == target){
            //return new int[]{nums[lo], nums[hi]};
            return new int[] {lo, hi};
        }else if(nums[lo] + nums[hi] < target){
            while(lo < hi && nums[lo] == nums[lo+1]) lo++;  //如果有重复元素就跳过
            lo++;
        }else{
            while(lo < hi && nums[hi] == nums[hi-1]) hi--;
            hi--;
        }
    }
    throw new IllegalArgumentException("no solution");
}
```
## 1.2 ThreeSum
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/ThreeSum.PNG">
</center>

- 解法:和TwoSum的思想类似，利用双指针，首先固定一个数nums[i]，然后改变另外两个数nums[lo]和nums[hi]。这里需要注意的是，for循环的边界条件nums.length-2是怎么确定的，当i=nums.length-3时，lo=nums.length-2,hi=nums.length-1，三个数正好是数组最后面的三位数。

```
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new LinkedList<>();
    for(int i = 0; i < nums.length - 2; i++){
        if(i == 0 || (i > 0 && nums[i] != nums[i-1])){
            int lo = i+1, hi = nums.length-1, sum = 0-nums[i];
            while(lo < hi){
                if(nums[lo] +nums[hi] == sum){
                    res.add(Arrays.asList(nums[i], nums[lo], nums[hi]));
                    while(lo < hi && nums[lo] == nums[lo+1]) lo++;  //题目要求，元素不能重复
                    while(lo < hi && nums[hi] == nums[hi-1]) hi--;
                    lo++;
                    hi--;
                }else if(nums[lo] + nums[hi] < sum){
                    lo++;
                }else{
                    hi--;
                }
            }
        }
    }
    return res;
}
```

## 1.3 ThreeSumClosest
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/ThreeSumClosest.PNG">
</center>

- 解法1:借鉴ThreeSum的思想。有一点需要理解，在数学上，绝对值表示"距离"，所以第11行的语句
ans = (Math.abs(target-sum) > Math.abs(target-ans)) ? ans : sum;
意思是比较target到sum的距离和target到ans的距离，取小值。

```
public int threeSumClosest(int[] nums, int target) {
    int sum = 0;
    //int ans = Integer.MAX_VALUE;
    int ans = nums[0] + nums[1] + nums[nums.length-1];
    Arrays.sort(nums);
    for(int i = 0; i < nums.length-2; i++){
        int lo = i+1, hi = nums.length-1;
        // sum = nums[i] + nums[lo] + nums[hi];
        while(lo < hi){
            sum = nums[i] + nums[lo] + nums[hi];
            ans = (Math.abs(target-sum) > Math.abs(target-ans)) ? ans : sum;
            if(sum == target){
                return target;
            }else if(sum < target){
                while(lo < hi && nums[lo] == nums[lo+1]) lo++;
                lo++;
            }else{
                while(lo < hi && nums[hi] == nums[hi-1]) hi--;
                hi--;
            }
        }
    }
    return ans;
}
```

- 解法2:和解法1思想一样，但是简洁一些

```
public int threeSumClosest(int[] num, int target) {
    int result = num[0] + num[1] + num[num.length - 1];
    Arrays.sort(num);
    for (int i = 0; i < num.length - 2; i++) {
        int start = i + 1, end = num.length - 1;
        while (start < end) {
            int sum = num[i] + num[start] + num[end];
            if (sum > target) {
                end--;
            } else {
                start++;
            }
            if (Math.abs(sum - target) < Math.abs(result - target)) {
                result = sum;
            }
        }
    }
    return result;
}
```

## 1.4 FourSum
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/FourSum.PNG">
</center>

- 解法：从towsum到threeesum可以看出一些规律，可以用递归的方式解决xSum的问题。这是因为kSum的问题可以分解成两个问题：
    - twoSum问题
    - 将K sum问题简化为K-1 sum问题

```java
public class Solution {
    int len = 0;
    public List<List<Integer>> fourSum(int[] nums, int target) {
        len = nums.length;
        Arrays.sort(nums);
        return kSum(nums, target, 4, 0);
    }
   private ArrayList<List<Integer>> kSum(int[] nums, int target, int k, int index) {
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        if(index >= len) {
            return res;
        }
        if(k == 2) {
        	int i = index, j = len - 1;
        	while(i < j) {
                //find a pair
        	    if(target - nums[i] == nums[j]) {
        	    	List<Integer> temp = new ArrayList<>();
                	temp.add(nums[i]);
                	temp.add(target-nums[i]);
                    res.add(temp);
                    //skip duplication
                    while(i<j && nums[i]==nums[i+1]) i++;
                    while(i<j && nums[j-1]==nums[j]) j--;
                    i++;
                    j--;
                //move left bound
        	    } else if (target - nums[i] > nums[j]) {
        	        i++;
                //move right bound
        	    } else {
        	        j--;
        	    }
        	}
        } else{
            for (int i = index; i < len - k + 1; i++) {
                //use current number to reduce ksum into k-1sum
                ArrayList<List<Integer>> temp = kSum(nums, target - nums[i], k-1, i+1);
                if(temp != null){
                    //add previous results
                    for (List<Integer> t : temp) {
                        t.add(0, nums[i]);
                    }
                    res.addAll(temp);
                }
                while (i < len-1 && nums[i] == nums[i+1]) {
                    //skip duplicated numbers
                    i++;
                }
            }
        }
        return res;
    }
}
```
## 1.5 FourSum II
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/FourSum%20II.PNG">
</center>

看到这个问题时，最先想到的方法就是暴力法，四个for循环嵌套，但可想而知算法的速度是多么慢解决这种复杂问题的一个思路就是***分治***，把复杂问题分解成几个比较简单的问题。

- 解法1：使用HashMap。首先将数组A和数组B中元素的所有可能的和算出来，放入hashmp中，如果某个和出现多于一次，每出现一次就将其出现的次数加一；否则将出现次数设置为一。然后将数组C和数组D中所有可能出现的和算出来并取相反数，和hashmap中的数比较，最终算出所有相同的数的个数，即为最终结果。

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Map<Integer, Integer> sums = new HashMap<>();
        int count = 0;
        for(int i = 0; i < A.length; i++){
            for(int j = 0; j < B.length; j++){
                int sum = A[i] + B[j];
                if(sums.containsKey(sum)){
                    sums.put(sum, sums.get(sum)+1);
                }else{
                    sums.put(sum, 1);
                }
            }
        }

        for(int k = 0; k < C.length; k++){
            for(int v = 0; v < D.length; v++){
                int sum = -(C[k] + D[v]);
                if(sums.containsKey(sum)){
                    count += sums.get(sum);
                }
            }
        }
        return count;
    }
}
```

- 解法1的简化写法，使用HashMap中的getOrDefault方法，这个方法有两个参数：getOrDefault(A, B)，参数A表示key值，参数B表示默认值。如果HashMap中包含该key值，就使用key值对应的value，否则就使用B。

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    Map<Integer, Integer> sums = new HashMap<>();
    int count = 0;
    for(int i = 0; i < A.length; i++){
        for(int j = 0; j < B.length; j++){
            int sum = A[i] + B[j];
            sums.put(sum, sums.getOrDefault(sum, 0)+1);
        }
    }

    for(int k = 0; k < C.length; k++){
        for(int v = 0; v < D.length; v++){
            count += sums.getOrDefault(-1 * (C[k]+D[v]), 0);
        }
    }
    return count;
}
```
- 解法2：使用排序。

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    int nAB = A.length * B.length;
    int[] sumAB = new int[nAB];
    int i = 0;
    for(int a : A){
        for(int b : B){
            sumAB[i++] = a +b;
        }
    }

    int nCD = C.length * D.length;
    int[] negSumCD = new int[nCD];
    i = 0;
    for(int c : C){
        for(int d : D){
            negSumCD[i++] = -(c + d);
        }
    }

    Arrays.sort(sumAB);
    Arrays.sort(negSumCD);

    i = 0;
    int j = 0, count = 0;
    while(i < nAB && j < nCD){
        if(sumAB[i] == negSumCD[j]){
            int countAB = 1, countCD = 1;
            while(++i < nAB && sumAB[i]==sumAB[i-1]) countAB += 1;
            while(++j < nCD && negSumCD[j]==negSumCD[j-1]) countCD += 1;
            count += countAB * countCD;
        }else if(sumAB[i] < negSumCD[j]){
            i++;
        }else{
            j++;
        }
    }
    return count;
}
```
# 2. Trapping  Water(盛水问题)
## 2.1 Trapping Rain Water
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/trapping%20rain%20water.PNG">
</center>

- 解法：这种问题的比较好的解决方法就是***双指针法***。需要注意的问题是，程序中`if(height[left] < height[right])`这句话是关键，它比较出此时`left`指针和`right`指针指向的`height`的高度，然后移动高度小的指针。因为盛水多少是由`height`比较短的一端决定的。如果移动`height`比较长的指针，比如当`left`指向下标`3`而`right`指向下标`7`的时候，此时移动`right`，下一步就会执行`right_max - height[right]`出错，因为很明显水位没有到达此时`height[right]`的高度。

```java
class Solution{
    public int trap(int[] height){
        int left = 0, right = height.length-1;
        int left_max = 0, right_max = 0;
        int ans = 0;
        while(left < right){
            if(height[left] < height[right]){
                if(height[left] >= left_max){
                    left_max = height[left];
                }else{
                    ans += left_max - height[left];
                }
                left++;
            }else{
                if(height[right] >= right_max){
                    right_max = height[right];
                }else{
                    ans += right_max - height[right];
                }
                right--;
            }
        }
        return ans;
    }
}
```
## 2.2 Container With Most Water
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/containing%20with%20most%20water.PNG">
</center>

- 解法：这个题的思路和上个题目类似，不同的是需要移动`height`值较短的指针。

假设现在有一个容器，则容器的盛水量取决于容器的底和容器较短的那条高。则我们可以从最大的底长入手，即当容器的底等于数组的长度时，则容器的盛水量为较短边的长乘底

可见 只有较短边会对盛水量造成影响，因此移动较短边的指针，并比较当前盛水量和当前最大盛水量。直至左右指针相遇。主要的困惑在于如何移动双指针才能保证最大的盛水量被遍历到

假设有左指针left和右指针right，且left指向的值小于right的值，假如我们将右指针左移，则右指针左移后的值和左指针指向的值相比有三种情况：

（1）右指针指向的值大于左指针
这种情况下，容器的高取决于左指针，但是底变短了，所以容器盛水量一定变小
（2）右指针指向的值等于左指针
这种情况下，容器的高取决于左指针，但是底变短了，所以容器盛水量一定变小
（3）右指针指向的值小于左指针
这种情况下，容器的高取决于右指针，但是右指针小于左指针，且底也变短了，所以容量盛水量一定变小了

综上所述，容器高度较大的一侧的移动只会造成容器盛水量减小。所以应当移动高度较小一侧的指针，并继续遍历，直至两指针相遇。

***更严谨的证明：反证法***
&emsp; 之前证明的只是在左指针不改变的情况下，左移右指针只会造成容器的容量减小。但是一旦紧接着左指针发生变化，就无法证明以该左指针为一侧高，右指针右侧的值生成的容器的容量比当前值小。
&emsp; 以下补充一个简单的反证法证明算法的合理性
&emsp; 当前的算法为：使用两个指针分别指向数组的头和尾。指向的值较小的那个指针移动，即左指针右移，右指针左移。当左右指针相遇时，
&emsp; 假设：该算法并没有遍历到容量最大的情况
&emsp; 我们令容量最大时的指针为p_left和p_right。根据题设，我们可以假设遍历时左指针先到达p_left，但是当左指针为p_left时，右指针还没有经过p_right左指针就移动了。
&emsp; 已知当左指针停留在p_left时，它只有在两种场景下会发生改变:

- 左指针和右指针在p_left相遇，则右指针一定在前往p_left的途中经过p_right，与题设矛盾
- 右指针位于p_right右侧且当前的值大于左指针。则在这种情况下，此时容器的盛水量比题设中最大的盛水量还要大，与题设矛盾
因此该算法的遍历一定经过了最大的盛水量的情况
```java
class Solution {
    public int maxArea(int[] height){
        int max = 0, l = 0, r = height.length-1;

        while(l < r){
            max = Math.max(max, Math.min(height[l], height[r]) * (r - l));
            if(height[l] < height[r]){
                l++;
            }else{
                r--;
            }
        }
        return max;
    }
}
```

# 3. Best Time to Buy and Sell(利润最大问题)
## 3.1 Best Time to Buy and Sell Stock
![](https://raw.githubusercontent.com/adamhand/LeetCode-images/master/best%20time%20to%20buy%20and%20sell%20stock.PNG)
***注意：***题目中要求，只能进行一次交易。
- 解法1：暴力循环。因为只能进行一次交易，只需找到满足条件(这里的条件值的是小值要在大值之前)的“最大值”和“最小值”即可。外层就是一个冒泡排序法的嵌套循环。

```java
public class Solution {
    public int maxProfit(int prices[]) {
        int maxprofit = 0;
        for (int i = 0; i < prices.length - 1; i++) {
            for (int j = i + 1; j < prices.length; j++) {
                int profit = prices[j] - prices[i];
                if (profit > maxprofit)
                    maxprofit = profit;
            }
        }
        return maxprofit;
    }
}
```

- 解法2：一个窍门。方法一中暴力循环方法其实是用的冒泡排序法的思想，但是显然这里不需要排序，只需要找到最大值和最小值就行可以。先看下面的图：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/121_profit_graph.png">
</center>

只需要找到“波峰”和“波谷”就行。可以定义两个变量`minprice`和`maxprofit`来解决这个问题。

```java
public class Solution {
    public int maxProfit(int prices[]) {
        int minprice = Integer.MAX_VALUE;
        int maxprofit = 0;
        for (int i = 0; i < prices.length; i++) {
            if (prices[i] < minprice)
                minprice = prices[i];
            else if (prices[i] - minprice > maxprofit)
                maxprofit = prices[i] - minprice;
        }
        return maxprofit;
    }
}
```
## 3.2 Best Time to Buy and Sell Stock II
![](https://raw.githubusercontent.com/adamhand/LeetCode-images/master/best%20time%20to%20buy%20and%20sell%20stock_2.PNG)
***注意：***和上一个题目的不同之处在于可以进行多次买卖。

## [334. Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/)

描述：
```
Given an unsorted array return whether an increasing subsequence of length 3 exists or not in the array.

Formally the function should:
```
> Return true if there exists i, j, k 
> such that arr[i] < arr[j] < arr[k] given 0 ≤ i < j < k ≤ n-1 else return false.

**Note:** Your algorithm should run in O(n) time complexity and O(1) space complexity.

Example 1:
```
Input: [1,2,3,4,5]
Output: true
```
Example 2:
```
Input: [5,4,3,2,1]
Output: false
```

前面说到了双指针法，即使用两个指针指向数组的两个位置，然后比较两个指针指向元素的大小，可以看到，“指针”在动而数组不动。

这里如果用这种方法比较复杂，但是受它启发，可以使用一种变形的“双指针”。让数组的元素挨个和指针元素进行比较，“指针”不动而数组在动。具体思路是：

定义一个small指针和一个big指针，将数组元素挨个和这两个指针比较，如果小于small，就将数组元素值赋值给small；如果比small大但比big小，就将元素赋值给big；如果这两个条件都不满足，说明当前元素比big大，而big又比small大，满足提议的情况就出现了，返回true。如果遍历完数组还没有出现上述情况，返回false。
```
class Solution {
    public boolean increasingTriplet(int[] nums) {
        if(nums.length < 3)
            return false;

        int small = Integer.MAX_VALUE, big = Integer.MAX_VALUE;
        for(int n : nums){
            if(n <= small)
                small = n;
            else if(n <= big)
                big = n;
            else 
                return true;
        }
        return false;
    }
}
```