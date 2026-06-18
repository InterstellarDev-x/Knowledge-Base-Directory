# Pattern 8: Greedy Array Algorithms

## Identify This Pattern
- "Can you reach the end?" / "Minimum jumps to reach end"
- "Can you complete the circuit?"
- "Distribute candies / minimum fairness"
- "Best time to buy and sell stock with cooldown/fee"
- Locally optimal choice at each step leads to globally optimal solution

---

## Templates

### Jump Game (Can you reach end?)
```cpp
// Greedy: track max reachable index
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (i > maxReach) return false;        // stuck before i
        maxReach = max(maxReach, i + nums[i]); // extend reach
    }
    return true;
}
```

### Minimum Jumps
```cpp
// Greedy BFS-like: track current range, next range, jumps
int jump(vector<int>& nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < nums.size() - 1; i++) {  // stop before last
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {      // exhausted current jump range
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

### Gas Station (circular route)
```cpp
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < gas.size(); i++) {
        int net = gas[i] - cost[i];
        tank += net;
        total += net;
        if (tank < 0) {        // can't continue from 'start'
            start = i + 1;     // try starting after this station
            tank = 0;
        }
    }
    return (total >= 0) ? start : -1;  // total < 0 → impossible
}
```

### Candy (minimum candies with neighbor constraints)
```cpp
int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> candy(n, 1);
    // Left pass: if rating[i] > rating[i-1], give one more than left
    for (int i = 1; i < n; i++)
        if (ratings[i] > ratings[i-1]) candy[i] = candy[i-1] + 1;
    // Right pass: if rating[i] > rating[i+1], ensure more than right
    for (int i = n-2; i >= 0; i--)
        if (ratings[i] > ratings[i+1]) candy[i] = max(candy[i], candy[i+1] + 1);
    return accumulate(candy.begin(), candy.end(), 0);
}
```

### Stock Problems
```cpp
// Best time to buy and sell (once) — covered in Kadane's: track min, calc profit
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, maxProfit = 0;
    for (int p : prices) {
        minPrice = min(minPrice, p);
        maxProfit = max(maxProfit, p - minPrice);
    }
    return maxProfit;
}

// Multiple transactions (unlimited buys)
int maxProfitII(vector<int>& prices) {
    int profit = 0;
    for (int i = 1; i < prices.size(); i++)
        if (prices[i] > prices[i-1]) profit += prices[i] - prices[i-1];  // collect every upward move
    return profit;
}

// With transaction fee
int maxProfitFee(vector<int>& prices, int fee) {
    int hold = -prices[0], cash = 0;
    for (int i = 1; i < prices.size(); i++) {
        cash = max(cash, hold + prices[i] - fee);  // sell
        hold = max(hold, cash - prices[i]);         // buy
    }
    return cash;
}

// With cooldown (1 day rest after sell)
int maxProfitCooldown(vector<int>& prices) {
    int hold = -prices[0], sold = 0, rest = 0;
    for (int i = 1; i < prices.size(); i++) {
        int prevHold = hold, prevSold = sold, prevRest = rest;
        hold = max(prevHold, prevRest - prices[i]);  // buy (can only buy from rest)
        sold = prevHold + prices[i];                  // sell
        rest = max(prevRest, prevSold);               // rest (choose better of previous rest or after-sell)
    }
    return max(sold, rest);
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [55](https://leetcode.com/problems/jump-game/) | Jump Game | Track max reachable. If i > max, return false | Greedy: always maintain the furthest we can reach |
| [121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Best Time to Buy/Sell I | Track min price seen so far, max profit | Single transaction: buy low, sell high after buy |
| [122](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/) | Best Time to Buy/Sell II | Sum every positive daily difference | Equivalent to buying every valley, selling every peak |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [45](https://leetcode.com/problems/jump-game-ii/) | Jump Game II | Greedy BFS: jump when current range exhausted | `jumps++` only when `i == curEnd`, not at every step |
| [134](https://leetcode.com/problems/gas-station/) | Gas Station | When tank < 0, restart from next station | Key: if total gas ≥ total cost, a solution exists — the valid start found by the greedy process is it |
| [135](https://leetcode.com/problems/candy/) | Candy | Two passes: left-to-right then right-to-left | Ensure each pass's constraint is met; take max for conflicting requirements |
| [309](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) | Buy/Sell with Cooldown | 3 states: hold, sold, rest. State machine transitions | After selling, must rest for 1 day before buying |
| [714](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) | Buy/Sell with Fee | 2 states: hold and cash. Deduct fee on sell | Fee reduces each profit; otherwise same as unlimited transactions |
| [738](https://leetcode.com/problems/monotone-increasing-digits/) | Monotone Increasing Digits | Find first decrease point. Set it to -1, propagate 9s right | Walk backwards from violation |
| [406](https://leetcode.com/problems/queue-reconstruction-by-height/) | Queue Reconstruction by Height | Sort by height desc (then k asc). Insert each by their k position | Taller people don't affect shorter people's k-counts |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) | Buy/Sell III (at most 2 transactions) | Track buy1/sell1/buy2/sell2 states | 4 state variables, update in one pass |
| [188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) | Buy/Sell IV (at most k transactions) | DP: `dp[k][i]` = max profit with at most k txns up to day i | If k ≥ n/2, reduce to unlimited transactions |
| [871](https://leetcode.com/problems/minimum-number-of-refueling-stops/) | Minimum Refueling Stops | Greedy + max-heap: add reachable stations to heap, pick max when fuel empty | Equivalent to: always defer the most profitable refuel |

---

## Stock Problems Decision Tree

```
1 transaction         → #121 — track min, max profit
Unlimited             → #122 — sum positive diffs
With cooldown         → #309 — 3-state DP (hold, sold, rest)
With fee              → #714 — 2-state DP (hold, cash)
At most 2 transactions → #123 — 4 state variables
At most k transactions → #188 — k × n DP
```

## Gas Station — Why It Works

```
If total gas >= total cost, a valid starting point exists.
The greedy approach finds it: whenever cumulative tank goes negative,
the current starting station is invalid. Start fresh from i+1.

Key: why doesn't the section before 'start' invalidate start?
Because if total >= 0 and we've been "subsidized" from the start..i-1 section,
starting from 'start' and having excess from the rest will cover it.
```

## Common Mistakes

- Jump Game II: counting jumps at every position instead of only when `i == curEnd`
- Gas Station: returning index after loop without checking `total >= 0`
- Candy: doing only one pass (left to right) and missing violations going right to left
- Stock with cooldown: transitioning `hold = max(hold, cash - price)` instead of `hold = max(hold, rest - price)` — buy can only happen from rest state

## Follow-up Interviewers Love

- "Jump Game II — is greedy provably optimal?" → Yes: each jump extends coverage as far as possible, so you can never do better
- "Gas Station — what if there are multiple valid starting points?" → There's at most one; if total ≥ 0, the greedy finds it
- "Candy — why two passes instead of one?" → Each constraint is directional; one pass can only enforce one direction
