# 528. 按权重随机选择

- LeetCode：[528. 按权重随机选择](https://leetcode.cn/problems/random-pick-with-weight/)
- 难度：中等

## 解题思路

权重表示每个下标被选中的相对概率。

例如：

```text
w = [1, 3]
```

总权重为 `4`，因此：

```text
下标 0 被选中的概率 = 1 / 4
下标 1 被选中的概率 = 3 / 4
```

可以把总权重看成从 `1` 到 `4` 的四个等概率整数：

```text
随机数： 1 | 2  3  4
下标：   0 | 1  1  1
```

下标 `0` 占一个数字，下标 `1` 占三个数字。因为每个随机数出现的概率相同，所以某个下标占据的数字越多，被选中的概率就越高。

对于更一般的权重：

```text
w = [2, 3, 1]
```

可以划分为：

```text
随机数： 1  2 | 3  4  5 | 6
下标：   0  0 | 1  1  1 | 2
```

前缀和为：

```text
[2, 5, 6]
```

它表示每个下标所负责区间的右边界：

```text
下标 0：1 到 2
下标 1：3 到 5
下标 2：6 到 6
```

每次生成 `1` 到总权重之间的随机整数，再寻找第一个大于或等于该随机数的前缀和，其位置就是应该返回的下标。

## 用到的数据结构及原因

### 前缀和数组 `prefixSums`

保存权重的累计和：

```text
prefixSums[i] = w[0] + w[1] + ... + w[i]
```

选择前缀和数组的原因：

1. 它把每个下标的权重转换成一段连续区间。
2. 区间长度正好等于对应权重。
3. 前缀和数组严格递增，可以使用二分查找。
4. 不需要按照权重重复存储下标，避免权重很大时占用大量空间。

### 总权重 `totalWeight`

等于前缀和数组的最后一个值，用于确定随机整数范围：

```text
1 到 totalWeight
```

## 用到的方法及原因

### 前缀和

用累计权重确定每个下标对应区间的右边界。

如果前缀和是：

```text
[2, 5, 6]
```

那么三个下标分别拥有长度为 `2`、`3`、`1` 的区间，正好对应原权重。

### `Math.random()`

`Math.random()` 返回：

```text
[0, 1)
```

也就是大于或等于 `0`、小于 `1` 的随机小数。

使用下面的转换：

```text
floor(Math.random() * totalWeight) + 1
```

可以得到：

```text
1 到 totalWeight
```

之间的随机整数，并且每个整数的概率相同。

### 二分查找左边界

随机数生成后，需要在递增的 `prefixSums` 中寻找：

> 第一个大于或等于随机数的位置。

例如：

```text
prefixSums = [2, 5, 6]
randomTarget = 4
```

第一个大于或等于 `4` 的前缀和是 `5`，下标为 `1`，因此返回 `1`。

选择二分查找的原因是，每次选择可以从线性查找的 `O(n)` 降到 `O(log n)`。

## 伪代码

```text
初始化：
    创建 prefixSums
    prefixSum = 0

    遍历所有权重：
        prefixSum += 当前权重
        将 prefixSum 加入 prefixSums

    totalWeight = prefixSum

每次随机选择：
    生成 1 到 totalWeight 之间的随机整数 target

    使用二分查找：
        在 prefixSums 中寻找第一个大于或等于 target 的位置

    返回这个位置
```

## JavaScript 代码

```js
/**
 * @param {number[]} w
 */
var Solution = function (w) {
  this.prefixSums = [];

  let prefixSum = 0;

  for (const weight of w) {
    prefixSum += weight;
    this.prefixSums.push(prefixSum);
  }

  this.totalWeight = prefixSum;
};

/**
 * @return {number}
 */
Solution.prototype.pickIndex = function () {
  const target =
    Math.floor(Math.random() * this.totalWeight) + 1;

  let left = 0;
  let right = this.prefixSums.length - 1;

  while (left < right) {
    const mid = left + Math.floor((right - left) / 2);

    if (this.prefixSums[mid] >= target) {
      right = mid;
    } else {
      left = mid + 1;
    }
  }

  return left;
};
```

## 复杂度分析

假设权重数组长度为 `n`：

- 初始化时间复杂度：`O(n)`。需要构建前缀和数组。
- 每次选择时间复杂度：`O(log n)`。使用二分查找定位随机数所属区间。
- 空间复杂度：`O(n)`。需要保存前缀和数组。

## 为什么不能按权重重复存储下标

对于：

```text
w = [2, 3, 1]
```

可以创建：

```text
[0, 0, 1, 1, 1, 2]
```

然后随机选择一个位置。这种做法在小权重时能工作，但如果权重非常大，就需要创建非常大的数组，空间复杂度与权重总和有关。

前缀和只需要保存 `n` 个数字，因此更加稳定。

## 容易出错的地方

### 1. 随机数范围少了 `+ 1`

使用整数区间方案时，需要生成：

```text
1 到 totalWeight
```

所以应该写成：

```js
Math.floor(Math.random() * this.totalWeight) + 1
```

### 2. 二分查找条件写成严格大于

因为随机整数区间包含前缀和右边界，所以需要寻找第一个：

```text
prefixSums[i] >= target
```

### 3. 二分查找找到任意匹配位置就停止

本题要寻找第一个满足条件的位置，而不是普通的精确值查找，因此满足条件后要继续保留左半区间：

```text
right = mid
```

### 4. 误以为少量调用必须严格符合比例

概率只表示大量调用后的长期比例。权重是 `[1, 3]` 时，调用四次不保证下标 `0` 恰好出现一次、下标 `1` 恰好出现三次。

### 5. 直接随机选择数组下标

直接生成 `0` 到 `w.length - 1` 的随机下标会让所有下标等概率，无法体现不同权重。
