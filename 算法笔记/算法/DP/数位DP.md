
对于数位DP，有几个特征
1. 求区间[L, R]内，满足某种性质的数的**个数** 
2. 要使用前缀和思想，将问题转为[0, R] - [0, L - 1] 问题
3. 个人感觉记忆化搜索好理解一些
## [LC.2719. 统计整数数目](https://leetcode.cn/problems/count-of-integers/description/)

>给你两个数字字符串 num1 和 num2 ，以及两个整数 max_sum 和 min_sum 。如果一个整数 x 满足以下条件，我们称它是一个好整数：
num1 <= x <= num2
min_sum <= digit_sum(x) <= max_sum.
请你返回好整数的数目。答案可能很大，请返回答案对 109 + 7 取余后的结果。
注意，digit_sum(x) 表示 x 各位数字之和。
示例 1：
输入：num1 = "1", num2 = "12", min_sum = 1, max_sum = 8
输出：11
解释：总共有 11 个整数的数位和在 1 到 8 之间，分别是 1,2,3,4,5,6,7,8,10,11 和 12 。所以我们返回 11 。
示例 2：
输入：num1 = "1", num2 = "5", min_sum = 1, max_sum = 5
输出：5
解释：数位和在 1 到 5 之间的 5 个整数分别为 1,2,3,4 和 5 。所以我们返回 5
提示：
1 <= num1 <= num2 <= 1022
1 <= min_sum <= max_sum <= 400
### 分析
1. 给定了上界与下界，可求[0, R] - [0, L - 1]。
2. 限制条件是什么？是上界，是digit_sum。首先肯定不能到上界以上了，当然也不能小于min_sum, 也不能大于max_sum，这就是限制条件。
3. 对于数位DP,就要时刻找好各类条件，定义好条件，在条件下进行搜索。
### AC 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = __int128;

// 处理借位问题
string deHelper(string str) {
    for (int i = str.size() - 1; i >= 0; --i) {
        if (str[i] != '0') {
            str[i]--;
            break;
        }
        str[i] = '9';
    }
    if (str[0] == '0' && str.size() > 1) 
        str = str.substr(1);
    return str;
}

ll countSolve(string num, int min_sum, int max_sum) {
    ll memo[24][2][500];
    memset(memo, -1, sizeof(memo));
    function<ll(int, int, int)> dfs = [&](int pos, bool isLimit, int able)->ll {
        if (pos == num.size()) {
            // 判断是否满足最低点
            // if (isLead) return 0;
            if (max_sum - able >= min_sum) return 1;
            return 0;
        }
        // 查询memo
        if (memo[pos][isLimit][able] != -1) {
            return memo[pos][isLimit][able];
        }
        // 计算当前up是多少
        int up = 9;
        if (isLimit) up = num[pos] - '0';
        if (able < up) up = able;
        ll res = 0;
        for (int i = 0; i <= up; ++i) {
            res += dfs(pos + 1, isLimit && (i == up), able - i);
        }
        memo[pos][isLimit][able] = res;
        return res;
    };
    return dfs(0, true, max_sum);
}

int count(string num1, string num2, int min_sum, int max_sum) {
    // [L, R] -> [0, R] - [0, L - 1]
    ll R = countSolve(num2, min_sum, max_sum);
    num1 = deHelper(num1);
    ll L = countSolve(num1, min_sum, max_sum);
    return (R - L) % (long long)(1e9 + 7);
}

int main() {
    cout << deHelper("4179205230");
    //cout << count("8990", "9927", 1, 3);
    return 0;
}
```

### 小结
1. 这题遇到第一个坑：要开__int128 要不然答案太大了会爆炸
2. 第二个坑：在处理 **L-1** 时候没有考虑借位问题，导致出错
3. 对于数位DP,我们要考虑好这个dfs参数咋写 核心参数：``pos`` ``isLimit`` 等其他参数 
## [LC.1012. 至少有 1 位重复的数字](https://leetcode.cn/problems/numbers-with-repeated-digits/)
	给定正整数 n，返回在 [1, n] 范围内具有 至少 1 位 重复数字的正整数的个数。

	示例 1：
	输入：n = 20
	输出：1
	解释：具有至少 1 位重复数字的正数（<= 20）只有 11 。
	
	示例 2：
	输入：n = 100
	输出：10
	解释：具有至少 1 位重复数字的正数（<= 100）有 11，22，33，44，55，66，77，88，99 和 100 。
	示例 3：
	输入：n = 1000
	输出：262

	提示：
	1 <= n <= 109
### 思路
1. 正难则反，我们就考虑他的反命题，也就是``n - (没有重复的数量)``
2. 我们需要记录之前数字是否出现过，我们可以第一时间想到使用``unordered_set``，但是这东西不好存在memo记忆化数组里面，我们可以用位运算来模拟实现这个set,去记录当前数字是否已经在之前的中出现过了。
### AC 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int numDupDigitsAtMostN(int n) {
    string strNum = to_string(n);
    int memo[20][2][1 << 10];
    memset(memo, -1, sizeof(memo));
    auto dfs = [&](auto&& dfs, int pos, bool isLimit, int mask)->ll {
        if (pos == strNum.size()) {
            return mask != 0;
        }
        if (memo[pos][isLimit][mask] != -1) {
            return memo[pos][isLimit][mask];
        }
        int up = (isLimit) ? strNum[pos] - '0': 9;
        ll res = 0;
        // mask == 0 意味着当前是空的 还没有数字
        if (mask == 0) {
            res += dfs(dfs, pos + 1, false, mask); // 既然最高位是空的，自然也就不需要这个isLimit限制了
        }
        for (int i = 1 - (mask != 0); i <= up; ++i) {
            if (((mask >> i) & 1) == 0) { // 验证是否出现过了
                res += dfs(dfs, pos + 1, isLimit && (up == i), mask | (1 << i));
            }
        }
        memo[pos][isLimit][mask] = res;
        return res;
    };
    return n - dfs(dfs, 0, true, 0);
}

int main() {
    cout << numDupDigitsAtMostN(20);
    return 0;
}
```