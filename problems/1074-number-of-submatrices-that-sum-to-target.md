# 1074. 元素和为目标值的子矩阵数量

- LeetCode：[1074. 元素和为目标值的子矩阵数量](https://leetcode.cn/problems/number-of-submatrices-that-sum-to-target/)
- 难度：困难

## 解题思路

直接枚举子矩阵的上、下、左、右四条边界，需要检查大量组合，效率较低。

可以先固定子矩阵的上边界和下边界，把这两行之间每一列的元素累加起来。这样，一个二维子矩阵就被压缩成了一个一维连续子数组。

例如固定第 `top` 行到第 `bottom` 行后：

```text
原矩阵中第 0 列在这几行的和 -> columnSums[0]
原矩阵中第 1 列在这几行的和 -> columnSums[1]
原矩阵中第 2 列在这几行的和 -> columnSums[2]
```

此时，选择连续的若干列，就等于在 `columnSums` 中选择一个连续子数组。因此问题转换为：

> 一维数组中，和为 `target` 的连续子数组有多少个？

一维问题可以使用“前缀和 + 哈希表”解决。

如果当前位置的前缀和为 `prefixSum`，之前存在一个前缀和为：

```text
prefixSum - target
```

那么这两个位置之间的连续子数组之和就是 `target`。

## 用到的数据结构及原因

### 数组 `columnSums`

`columnSums[col]` 保存固定上、下边界后，第 `col` 列在这段行区间内的元素和。

选择数组的原因：

1. 矩阵的列下标是连续整数，适合直接用数组下标访问。
2. 当下边界向下移动一行时，可以直接累加新一行的元素。
3. 它把二维子矩阵问题转换成了一维连续子数组问题。

### `Map`（哈希表）

`Map` 保存：

```text
前缀和 -> 该前缀和已经出现的次数
```

这里记录的是“出现次数”，而不是最后出现的位置。因为同一个前缀和可能出现多次，每一次都能组成一个符合条件的子数组。

选择 `Map` 的原因：

1. 可以平均 `O(1)` 地查询 `prefixSum - target` 出现过多少次。
2. 可以平均 `O(1)` 地更新当前前缀和的出现次数。
3. 前缀和可能是负数，不适合直接用普通数组下标记录。

## 用到的方法及原因

### 二维降维

固定子矩阵的上、下边界，把边界之间的各列分别求和，将二维问题转换为一维问题。

这样只需要枚举行边界，列边界交给一维前缀和方法处理，不必再显式枚举左、右边界。

### 前缀和

假设遍历到当前位置时的前缀和是 `currentPrefix`，之前某个位置的前缀和是 `previousPrefix`，则它们之间的元素和为：

```text
currentPrefix - previousPrefix
```

要让这段元素和等于 `target`，就需要：

```text
previousPrefix = currentPrefix - target
```

因此，每得到一个新的前缀和，只需在 `Map` 中查询 `currentPrefix - target` 的出现次数。

### `map.get(key)`

获取某个前缀和已经出现的次数。如果不存在，则按照 `0` 次处理。

### `map.set(key, value)`

记录或更新某个前缀和的出现次数。

## 为什么先记录 `0 -> 1`

在统计一维数组前，需要先向 `Map` 中放入：

```text
前缀和 0 -> 出现 1 次
```

它代表“还没有选择任何元素时，前缀和为 0”。

如果从数组开头到当前位置的元素和正好等于 `target`，那么：

```text
currentPrefix - target = 0
```

这时预先记录的 `0 -> 1` 可以让这段子数组被正确计数。

## 伪代码

```text
answer = 0

枚举上边界 top：
    创建长度等于列数的 columnSums，并全部初始化为 0

    枚举下边界 bottom，从 top 开始向下移动：
        把第 bottom 行的每个元素累加到 columnSums 对应列

        创建 prefixCount 哈希表
        记录前缀和 0 出现 1 次
        prefixSum = 0

        遍历 columnSums 中的每个元素：
            prefixSum += 当前元素
            answer += prefixCount 中 prefixSum - target 的出现次数
            将 prefixSum 的出现次数加 1

返回 answer
```

## JavaScript 代码

```js
/**
 * @param {number[][]} matrix
 * @param {number} target
 * @return {number}
 */
var numSubmatrixSumTarget = function (matrix, target) {
  const rowCount = matrix.length;
  const columnCount = matrix[0].length;
  let answer = 0;

  for (let top = 0; top < rowCount; top++) {
    const columnSums = new Array(columnCount).fill(0);

    for (let bottom = top; bottom < rowCount; bottom++) {
      for (let column = 0; column < columnCount; column++) {
        columnSums[column] += matrix[bottom][column];
      }

      const prefixCount = new Map();
      prefixCount.set(0, 1);

      let prefixSum = 0;

      for (const value of columnSums) {
        prefixSum += value;

        const neededPrefix = prefixSum - target;
        answer += prefixCount.get(neededPrefix) ?? 0;

        const oldCount = prefixCount.get(prefixSum) ?? 0;
        prefixCount.set(prefixSum, oldCount + 1);
      }
    }
  }

  return answer;
};
```

## 复杂度分析

假设矩阵有 `m` 行、`n` 列：

- 时间复杂度：`O(m² × n)`。需要枚举上下边界，并对压缩后的一维数组进行一次遍历。
- 空间复杂度：`O(n)`。`columnSums` 和前缀和哈希表最多都需要与列数同阶的空间。

如果行数远大于列数，也可以反过来固定左、右边界并压缩每一行，使时间复杂度优化为：

```text
O(min(m, n)² × max(m, n))
```

当前实现优先保持逻辑直观，采用固定上、下边界的写法。

## 容易出错的地方

### 1. `Map` 必须记录出现次数

同一个前缀和可能出现多次，不能只记录它是否存在，也不能只记录最后一次出现的位置。

### 2. 忘记初始化 `0 -> 1`

这会漏掉从压缩数组第一个元素开始、元素和正好等于 `target` 的子数组。

### 3. 每次更换上边界时要重置 `columnSums`

不同的 `top` 对应不同的行区间。开始枚举新的上边界时，所有列的累计和必须重新从 `0` 开始。

### 4. `prefixCount` 要为每组上下边界重新创建

每一组上、下边界都会得到一个新的压缩数组，不能沿用上一组边界的前缀和统计。

### 5. 不能使用滑动窗口

矩阵中可能含有负数。窗口扩大后，元素和不一定增大；窗口缩小后，元素和也不一定减小，所以普通滑动窗口不适用。
