# 347. 前 K 个高频元素

- LeetCode：[347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)
- 难度：中等

## 解题思路

这道题可以分成两个步骤：

1. 统计每个数字出现了多少次。
2. 按照出现次数从高到低，取出前 `k` 个数字。

第一步可以使用 `Map`。

第二步如果直接对所有数字按照频率排序，时间复杂度会达到 `O(m log m)`，其中 `m` 是不同数字的数量。但一个数字最多出现 `n` 次，因此频率一定处于：

```text
1 到 nums.length
```

可以创建一组“桶”，用频率作为下标：

```text
buckets[频率] = 所有具有这个频率的数字
```

例如：

```text
nums = [1, 1, 1, 2, 2, 3]
```

频率统计结果是：

```text
1 出现 3 次
2 出现 2 次
3 出现 1 次
```

放入桶中：

```text
buckets[1] = [3]
buckets[2] = [2]
buckets[3] = [1]
```

从下标最大的桶向前遍历，就会依次得到频率最高的元素：

```text
1、2、3
```

如果 `k = 2`，取前两个即可得到 `[1, 2]`。

## 用到的数据结构及原因

### `Map`（哈希表）

保存：

```text
数字 -> 该数字出现的次数
```

选择 `Map` 的原因：

1. 可以平均 `O(1)` 地查询一个数字当前的出现次数。
2. 可以平均 `O(1)` 地更新出现次数。
3. 数字可能是负数，使用 `Map` 比使用普通数组下标更合适。

### 二维数组 `buckets`

外层数组下标代表频率，内层数组保存拥有该频率的所有数字：

```text
buckets[count].push(number)
```

选择桶数组的原因：

1. 一个数字的最大频率不会超过 `nums.length`。
2. 使用频率作为下标，可以避免对不同元素进行比较排序。
3. 从后向前遍历桶，就能按照频率从高到低取得元素。

### 数组 `result`

按照频率从高到低保存最终找到的前 `k` 个数字。

## 用到的方法及原因

### 频率统计

遍历原数组，每遇到一个数字，就将它在 `Map` 中的计数加一。

### 桶排序思想

传统排序需要比较两个元素谁的频率更高，而桶排序直接把元素放到其频率对应的位置。

本题适合使用桶排序，是因为频率范围是已知且有限的：

```text
1 到 nums.length
```

### `map.get(key)` 和 `map.set(key, value)`

- `get` 获取数字当前出现次数。
- `set` 保存更新后的出现次数。

数字第一次出现时，`map.get(number)` 会得到 `undefined`，可以使用空值合并运算符按 `0` 处理：

```text
(frequency.get(number) ?? 0) + 1
```

### `Array.from()`

为每个频率创建一个独立的空数组：

```text
Array.from({ length: nums.length + 1 }, () => [])
```

数组长度是 `nums.length + 1`，因为需要使用 `nums.length` 作为最大频率下标。

### `push()`

用于把数字放进对应的频率桶，也用于把高频数字加入结果数组。

## 伪代码

```text
创建 frequency Map

遍历 nums：
    将当前数字的出现次数加 1

创建 nums.length + 1 个互相独立的桶

遍历 frequency 中的每组 数字和频率：
    将数字加入 buckets[频率]

创建结果数组 result

从最高频率向最低频率遍历：
    遍历当前频率桶中的所有数字：
        将数字加入 result

        如果 result 中已有 k 个数字：
            返回 result
```

## JavaScript 代码

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
var topKFrequent = function (nums, k) {
  const frequency = new Map();

  for (const number of nums) {
    const oldCount = frequency.get(number) ?? 0;
    frequency.set(number, oldCount + 1);
  }

  const buckets = Array.from(
    { length: nums.length + 1 },
    () => []
  );

  for (const [number, count] of frequency) {
    buckets[count].push(number);
  }

  const result = [];

  for (let count = nums.length; count >= 1; count--) {
    for (const number of buckets[count]) {
      result.push(number);

      if (result.length === k) {
        return result;
      }
    }
  }

  return result;
};
```

## 复杂度分析

设 `n` 是 `nums` 的长度：

- 时间复杂度：`O(n)`。统计频率、分配到桶中以及遍历桶的总工作量都是线性的。
- 空间复杂度：`O(n)`。`Map`、桶数组和结果数组最多保存与输入规模同阶的数据。

## 为什么不直接排序

也可以把 `Map` 中的所有条目转换成数组，再按照频率从大到小排序：

```text
统计频率 -> 排序 -> 取前 k 个
```

这种方法更容易想到，但如果有 `m` 个不同数字，排序需要：

```text
O(m log m)
```

本题要求算法的时间复杂度优于 `O(n log n)`，所以桶排序更符合题目要求。

## 容易出错的地方

### 1. 桶数组长度少一位

如果所有元素都相同，某个数字的频率会等于 `nums.length`，所以桶数组长度必须是：

```text
nums.length + 1
```

### 2. 使用 `fill([])` 创建桶

不要写成：

```js
new Array(nums.length + 1).fill([])
```

`fill([])` 会让所有位置引用同一个数组。向一个桶中加入数字时，其他所有桶也会一起变化。

应该为每个位置分别创建数组：

```js
Array.from({ length: nums.length + 1 }, () => [])
```

### 3. 从低频桶向高频桶遍历

题目要求前 `k` 个高频元素，因此必须从 `nums.length` 开始向 `1` 遍历。

### 4. 忘记同一个频率可能对应多个数字

`buckets[count]` 必须是数组，因为可能有多个数字拥有相同的出现次数。

### 5. 把数字值当作数组下标统计

输入数字可能是负数，也可能很大。频率统计应使用 `Map`，不能直接使用普通数组下标。
