# LeetCode题解之贪心算法和动态规划

## [121.Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

> 思路：只能进行一次买卖。可以使用一种类似贪心的算法。
> - 初始化数组的第一个元素为最低价格
> - 从左到右遍历，如果遇到最低价格，就更新最低价格，否则更新最大收益值

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0)
            return 0;
        
        int minPrice = prices[0];
        int maxProfit = 0;
        
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] < minPrice)
                minPrice = prices[i];  //更新最低价格
            else
                maxProfit = Math.max(maxProfit, prices[i] - minPrice);   //更新最大收益
        }
        return maxProfit;
    }
}
```

## [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

> 思路： 对于 [a, b, c, d]，如果有 a <= b <= c <= d ，那么最大收益为 d - a。而 d - a = (d - c) + (c - b) + (b - a) ，因此当访问到一个 prices[i] 且 prices[i] -
> prices[i-1] > 0，那么就把 prices[i] - prices[i-1] 添加到收益中，从而在局部最优的情况下也保证全局最优。贪心算法。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0)
            return 0;
        
        int ans = 0;
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i - 1]) {
                ans += prices[i] - prices[i - 1];
            }
        }
        return ans;
    }
}
```

## [714. Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

> 思路：和上一题不同，这道题有了交易费，所以当卖出的利润小于交易费的时候，我们就不应该卖了，不然亏了。考虑使用**动态规划**。
> - sold[i]表示第i天卖掉股票此时的最大利润
> - hold[i]表示第i天保留手里的股票此时的最大利润

> 递推公式分析如下：
> - 在第i天，如果我们要卖掉手中的股票，那么此时我们的总利润应该是前一天手里有股票的利润，加上此时的卖出价格，减去交易费得到的利润总值，跟前一天卖出的利润相比，取其中较大值，如果前一天卖出的利润较大，那么我们就前一天卖了，不留到今天了。
> - 如果第i天不卖，就是昨天股票卖了然后今天再买入股票，得减去今天的价格，得到的值和昨天股票保留时的利润相比，取其中的较大值，如果昨天保留股票的利润大，那么我们就继续保留到今天。

> 所以递推时可以得到：
> - sold[i] = max(sold[i - 1], hold[i - 1] + prices[i] - fee);
> - hold[i] = max(hold[i - 1], sold[i - 1] - prices[i]);

```java
class Solution {
    public int maxProfit(int[] prices, int fee) {
        int[] sold = new int[prices.length];
        int[] hold = new int[prices.length];
        sold[0] = 0;
        hold[0] = -prices[0];
        
        for (int i = 1; i < prices.length; i++) {
            sold[i] = Math.max(sold[i - 1], hold[i - 1] + prices[i] - fee);
            hold[i] = Math.max(hold[i - 1], sold[i - 1] - prices[i]);
        }
        
        return sold[prices.length - 1];
    }
}
```

> 优化方法。我们发现不管是卖出还是保留，第i天到利润只跟第i-1天有关系，所以我们可以优化空间，用两个变量来表示当前的卖出和保留的利润。
> 新方法和上面的基本相同，就是开始要保存sold的值，不然sold先更新后，再更新hold时就没能使用更新前的值了。

```java
public int maxProfit(int[] prices, int fee) {
    int sold = 0, hold = -prices[0];
    for (int i = 1; i < prices.length; i++) {
        sold = Math.max(sold, hold + prices[i] - fee);
        hold = Math.max(hold, sold - prices[i]);
    }
    return sold;
}
```

参考：[参考1](https://www.cnblogs.com/grandyang/p/7776979.html) [参考2](https://blog.csdn.net/zarlove/article/details/78323469)  [参考3](https://blog.csdn.net/zarlove/article/details/78323469)  [参考4](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/discuss/108870/most-consistent-ways-of-dealing-with-the-series-of-stock-problems)

## [309. Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

思路。因为当前日期买卖股票会受到之前日期买卖股票行为的影响，首先考虑到用DP解决。对于某一天，股票有三种状态: buy, sell, cooldown, sell与cooldown我们可以合并成一种状态，因为手里最终都没股票，最终需要的结果是sell，即手里股票卖了获得最大利润。所以使用两个数组记录当前持股和未持股的状态。

- sold[i]表示第i天没持股能够得到的最大利润
- hold[i]表示第i天持股能够得到的最大利润。

对于当天最终未持股的状态，最终最大利润有两种可能，一是今天没动作跟昨天未持股状态一样，二是昨天持股了，今天卖了。所以我们只要取这两者之间最大值即可，表达式如下：

- sold[i] = max(sold[i - 1], hold[i - 1] + prices[i])

对于当天最终持股的状态，最终最大利润有两种可能，一是今天没动作跟昨天持股状态一样，二是前天还没持股，今天买了股票，这里是因为cooldown的原因，所以今天买股要追溯到前天的状态。我们只要取这两者之间最大值即可，表达式如下：

- hold[i] = max(hold[i - 1], sold[i - 2] - prices[i])

最终我们要求的结果是sold[n - 1]，表示最后一天结束时手里没股票时的累积最大利润。

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length <= 1)
        return 0;
    
    int[] sold = new int[prices.length];
    int[] hold = new int[prices.length];
    hold[0] = -prices[0];
    hold[1] = Math.max(-prices[0], -prices[1]);
    sold[0] = 0;
    sold[1] = Math.max(sold[0], hold[0] + prices[1]);
    for (int i = 2; i < prices.length; i++) {
        sold[i] = Math.max(sold[i - 1], hold[i - 1] + prices[i]);
        hold[i] = Math.max(hold[i - 1], sold[i - 2] - prices[i]);
    }
    
    return sold[sold.length - 1];
}
```

上述的方法还可以简化空间。由于i只依赖于i-1和i-2，所以可以在O(1)的空间中解决。

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }

    int currBuy = -prices[0];
    int currSell = 0;
    int prevSell = 0;
    for (int i = 1; i < prices.length; i++) {
        int temp = currSell;
        currSell = Math.max(currSell, currBuy + prices[i]);
        if (i >= 2) {
            currBuy = Math.max(currBuy, prevSell - prices[i]);
        } else {
            currBuy = Math.max(currBuy, -prices[i]);
        }
        prevSell = temp;
    }
    return currSell;
}
```

参考：[参考](https://segmentfault.com/a/1190000004193861)

## [123. Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)

## [188. Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)