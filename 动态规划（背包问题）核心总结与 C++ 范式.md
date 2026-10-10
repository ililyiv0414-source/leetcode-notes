# 动态规划（背包问题）核心总结与 C++ 范式

## 一、 01背包 vs 完全背包 核心区别

| 维度 | 01背包 | 完全背包 |
| --- | --- | --- |
| **物品限制** | 每个物品/元素**只能使用 1 次** | 每个物品/元素**可重复使用无限次** |
| **递归转移** | 选了第 $i$ 个后，必须去看 $i-1$：<br>

<br>`dfs(i-1, c - w[i])` | 选了第 $i$ 个后，依然留在第 $i$ 个：<br>

<br>`dfs(i, c - w[i])` |
| **递推（一维DP）** | **外层**遍历物品，**内层**容量**倒序**：<br>

<br>`for (int c = target; c >= w; c--)` | **外层**遍历物品，**内层**容量**正序**：<br>

<br>`for (int c = w; c <= target; c++)` |
| **正/倒序原理** | 倒序确保计算 `dp[c]` 时使用的是**上一轮未更新的旧值**（防止重复选择） | 正序允许 `dp[c]` 依赖**本轮已更新的新值**（实现多次累加） |

---

## 二、 “至多”、“至少”、“恰好”三种目标的初始化规则

| 目标类型 | 问题举例 | 含义/约束 | `dp[0]` 初始值 | 其他 `dp[i]` 初始值 | 无效值/边界设值 |
| --- | --- | --- | --- | --- | --- |
| **恰好** (Exact) | 零钱兑换 (刚好凑满 `amount`) | 必须**正好**装满容量 | `0` | `INF` (求最小值) 或 `-INF` (求最大值) | 只有容量完全匹配才有效 |
| **至多** (At Most) | 标准01背包 (容量不超 `W`) | 容量不超过 $W$ 即可，可留空 | `0` | `0` | 容量不够时直接忽略 |
| **至少** (At Least) | 满足特定需求的最小代价 | 容量允许溢出（$\ge W$） | `0` | `INF` | 状态计算时容量超出的均映射到 `dp[0]` |

> **极值判定法则**：
> * **求最小值**（如最少硬币）：无效状态设为正无穷（如 `1e9`）。
> * **求最大值**（如最大价值、最长子序列）：无效状态设为负无穷（如 `-1e9`）。
> 
> 

---

## 三、 C++ 代码通用范式

### 1. 完全背包范式（以“零钱兑换”为例）

**递归 + 记忆化（DFS）**

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
private:
    const int INF = 1e9; // 设定不可能达到的大数，防止溢出
    vector<vector<int>> memo;

    int dfs(int i, int c, const vector<int>& coins) {
        // 1. 基础边界：凑齐了返回 0；硬币用完或容量不足返回 INF
        if (c == 0) return 0;
        if (i < 0 || c < 0) return INF;

        if (memo[i][c] != -1) return memo[i][c];

        // 2. 剪枝：装不下当前硬币，只能不选
        if (coins[i] > c) {
            return memo[i][c] = dfs(i - 1, c, coins);
        }

        // 3. 核心决策：不选 vs 选（完全背包：选了第 i 个，下一轮依然是 i）
        int not_pick = dfs(i - 1, c, coins);
        int pick = dfs(i, c - coins[i], coins) + 1; // 硬币数 +1

        return memo[i][c] = min(not_pick, pick);
    }

public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        memo.assign(n, vector<int>(amount + 1, -1));
        
        int ans = dfs(n - 1, amount, coins);
        return ans >= INF ? -1 : ans;
    }
};

```

**递推（一维 DP）**

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int INF = 1e9;
        // 1. 创建 DP 数组并初始化为 INF
        vector<int> dp(amount + 1, INF);

        // 2. 设定基础状态
        dp[0] = 0;

        // 3. 正序递推（完全背包可重复选）
        for (int c = 1; c <= amount; c++) {
            for (int coin : coins) {
                if (c >= coin) {
                    dp[c] = min(dp[c], dp[c - coin] + 1);
                }
            }
        }

        // 4. 检查结果
        return dp[amount] >= INF ? -1 : dp[amount];
    }
};

```

---

### 2. 01背包范式（以“和为 target 的最长子序列”为例）

**递归 + 记忆化（DFS）**

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
private:
    const int INF = 1e9; // 求最大值时，无效状态设置为 -INF
    vector<vector<int>> memo;

    int dfs(int i, int c, const vector<int>& nums) {
        if (c == 0) return 0;
        if (i < 0 || c < 0) return -INF;

        if (memo[i][c] != -1) return memo[i][c];

        if (nums[i] > c) {
            return memo[i][c] = dfs(i - 1, c, nums);
        }

        // 01背包：选了第 i 个之后，必须递减到 i - 1
        int not_pick = dfs(i - 1, c, nums);
        int pick = dfs(i - 1, c - nums[i], nums) + 1;

        return memo[i][c] = max(not_pick, pick);
    }

public:
    int lengthOfLongestSubsequence(vector<int>& nums, int target) {
        int n = nums.size();
        memo.assign(n, vector<int>(target + 1, -1));

        int ans = dfs(n - 1, target, nums);
        return ans < 0 ? -1 : ans;
    }
};

```

**递推（一维 DP）**

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int lengthOfLongestSubsequence(vector<int>& nums, int target) {
        int INF = -1e9;
        vector<int> dp(target + 1, INF);
        dp[0] = 0;

        // 01背包一维DP：外层遍历物品，内层倒序遍历容量
        for (int num : nums) {
            for (int c = target; c >= num; c--) {
                dp[c] = max(dp[c], dp[c - num] + 1);
            }
        }

        return dp[target] < 0 ? -1 : dp[target];
    }
};

```