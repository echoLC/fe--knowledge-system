# 动态规划
LeetCode上可以用动态规划来解答的算法题。

### 1.零钱兑换
[322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)。

```js
const coinChange = (coins, amount) => {
  if (amount === 0 || coins.length === 0) return -1

  const dp = new Array(amount + 1).fill(Number.MAX_VALUE)
  db[0] = 0

  for (let i = 0; i < coins.length; i++) {
    for (let j = 0; j <= amount; j++) {
      if (j - coins[i] >= 0) {
        dp[j] = Math.min(dp[j], dp[j - coins[i]] + 1)
      }
    }
  }

  return db[amount] === Number.MAX_VALUE ? -1 :  db[amount]
}
```