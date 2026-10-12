# 698. 划分为 K 个相等的子集

- LeetCode：[698. 划分为 K 个相等的子集](https://leetcode.cn/problems/partition-to-k-equal-sum-subsets/)
- 难度：中等

## 先理解题意

需要把数组中的所有数字分成恰好 `k` 个非空子集，并且每个子集的元素和都相同。

每个数字必须使用一次，不能遗漏，也不能重复使用。

例如：

```text
nums = [4, 3, 2, 3, 5, 2, 1]
k = 4
```

所有数字之和是：

```text
4 + 3 + 2 + 3 + 5 + 2 + 1 = 20
```

如果分成 `4` 组，每组目标和应该是：

```text
20 / 4 = 5
```

可以划分为：

```text
[5]
[4, 1]
[3, 2]
[3, 2]
```

所以返回 `true`。

## 第一步：先判断是否可能

假设所有数字总和为 `totalSum`。

如果要分成 `k` 个相等子集，每组的和只能是：

```text
target = totalSum / k
```

如果：

```text
totalSum % k !== 0
```

说明总和不能被 `k` 整除，不可能划分，直接返回 `false`。

另外，如果数组中最大的数字大于 `target`，它无法放进任何一组，也应该直接返回 `false`。

## 解题思路：把子集看成桶

创建 `k` 个桶，每个桶的容量是 `target`。

```text
buckets[i] = 第 i 个子集当前的元素和
```

然后从大到小处理每个数字，尝试把它放进某个不会超过 `target` 的桶中。

对于当前数字，有多种选择：

```text
放进第 0 个桶
放进第 1 个桶
...
放进第 k - 1 个桶
```

选择一个桶后，继续处理下一个数字。如果后续无法完成划分，就撤销这次选择，把数字放进其他桶，这就是回溯。

## 用到的数据结构及原因

### 数组 `buckets`

长度为 `k`，保存每个桶当前的元素和：

```text
buckets = [0, 0, 0, 0]
```

选择数组的原因：

1. 每个位置对应一个子集。
2. 可以快速判断当前数字放入某个桶后是否超过目标和。
3. 回溯时只需要把当前数字从桶的总和中减掉。

### `Set`：`triedBucketSums`

在处理同一个数字时，记录本层递归已经尝试过的桶总和。

如果两个桶当前总和相同，那么把同一个数字放进这两个桶，得到的状态本质相同，没有必要重复搜索。

例如：

```text
buckets = [3, 3, 0, 0]
当前数字 = 2
```

把 `2` 放进第一个和为 `3` 的桶，和放进第二个和为 `3` 的桶没有本质区别，都会得到：

```text
一个桶为 5，另一个仍为 3
```

所以同一层只尝试一次桶和 `3`。

## 用到的方法及原因

### 回溯

对于每个数字，枚举它可以放入的桶：

```text
选择：buckets[bucket] += number
递归：处理下一个数字
撤销：buckets[bucket] -= number
```

如果所有数字都成功放入，说明存在合法划分。

### 降序排序

先执行：

```js
nums.sort((a, b) => b - a);
```

优先放置较大的数字，因为大数字能放入的桶更少。如果某个选择不合理，可以更早发现并停止搜索。

排序不会改变答案，因为子集只关心包含哪些数字，不关心原数组顺序。

## 关键剪枝

### 剪枝一：总和不能整除

```text
totalSum % k !== 0
```

直接返回 `false`。

### 剪枝二：最大数字超过目标和

排序后，如果：

```text
nums[0] > target
```

最大数字无法放入任何桶，直接返回 `false`。

### 剪枝三：放入后超过目标和

如果：

```text
buckets[i] + number > target
```

当前数字不能放进该桶，跳过。

### 剪枝四：跳过相同桶状态

同一层递归中，桶的当前总和相同，就属于等价选择，只尝试一次。

这也包含多个空桶的情况。把当前数字放进第一个空桶和第二个空桶没有区别，所以不需要重复尝试所有空桶。

## 为什么所有数字放完就一定成功

代码保证每个桶的和永远不会超过 `target`。

所有数字总和又恰好是：

```text
k * target
```

当所有数字都放入 `k` 个桶后，如果每个桶都不超过 `target`，它们只能全部刚好等于 `target`。否则只要有一个桶小于 `target`，总和就不可能达到 `k * target`。

## 伪代码

```text
计算 totalSum

如果 totalSum 不能被 k 整除：
    返回 false

target = totalSum / k
将 nums 从大到小排序

如果最大数字大于 target：
    返回 false

创建 k 个和为 0 的桶

定义 backtrack(index)：
    如果 index 等于 nums.length：
        返回 true

    number = nums[index]
    创建本层 triedBucketSums

    遍历每个桶：
        如果当前桶和已经尝试过：
            跳过

        如果放入 number 后超过 target：
            跳过

        记录当前桶和已经尝试
        把 number 放入当前桶

        如果处理下一个数字成功：
            返回 true

        从当前桶撤销 number

    返回 false

返回 backtrack(0)
```

## JavaScript 代码

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {boolean}
 */
var canPartitionKSubsets = function (nums, k) {
  const totalSum = nums.reduce((sum, number) => sum + number, 0);

  if (totalSum % k !== 0) {
    return false;
  }

  const target = totalSum / k;
  nums.sort((a, b) => b - a);

  if (nums[0] > target) {
    return false;
  }

  const buckets = new Array(k).fill(0);

  const backtrack = function (index) {
    if (index === nums.length) {
      return true;
    }

    const number = nums[index];
    const triedBucketSums = new Set();

    for (let bucket = 0; bucket < k; bucket++) {
      const bucketSum = buckets[bucket];

      if (triedBucketSums.has(bucketSum)) {
        continue;
      }

      if (bucketSum + number > target) {
        continue;
      }

      triedBucketSums.add(bucketSum);
      buckets[bucket] += number;

      if (backtrack(index + 1)) {
        return true;
      }

      buckets[bucket] -= number;
    }

    return false;
  };

  return backtrack(0);
};
```

## 复杂度分析

设数组长度为 `n`：

- 时间复杂度：最坏可达到 `O(k^n)`，因为每个数字都可能尝试放入多个桶。排序和剪枝会显著减少实际搜索状态。
- 空间复杂度：最坏为 `O(n × k)`。递归深度最多为 `n`，每层用于剪枝的 `Set` 最多记录 `k` 种桶和；另外桶数组长度为 `k`。

回溯问题的重点通常不是只看最坏复杂度，而是通过问题约束和剪枝减少实际搜索量。本题 `nums.length <= 16`，适合这种搜索方法。

## 容易出错的地方

### 1. 总和不能整除时仍然回溯

应先判断：

```text
totalSum % k === 0
```

否则不存在整数目标和。

### 2. 没有降序排序

不排序仍可能得到正确答案，但搜索会慢很多。大数字优先能更快触发“超过 target”的剪枝。

### 3. 回溯后忘记撤销选择

递归失败后必须执行：

```text
buckets[bucket] -= number
```

否则后面的选择会使用错误的桶状态。

### 4. 对相同状态的空桶重复搜索

多个空桶完全等价。如果每个空桶都尝试一次，会产生大量重复分支。`triedBucketSums` 可以统一处理空桶和其他相同桶和。

### 5. 把子集理解成连续子数组

子集不要求元素在原数组中连续，因此不能使用滑动窗口。

### 6. 忘记所有数字都必须使用

回溯的结束条件是 `index === nums.length`，表示每个数字都已经分配到某个桶。

### 7. 使用默认 `sort()`

JavaScript 默认按照字符串规则排序，必须提供数字比较函数：

```js
nums.sort((a, b) => b - a);
```

### 8. 认为 K 个子集有顺序

这些子集没有编号意义。交换两个桶得到的是同一种划分，这正是可以跳过相同桶状态的原因。
