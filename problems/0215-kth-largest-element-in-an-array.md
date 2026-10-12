# 215. 数组中的第 K 个最大元素

- LeetCode：[215. 数组中的第 K 个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
- 难度：中等

## 先理解题意

题目要找数组排序后从大到小排列的第 `k` 个元素，不是第 `k` 个不同的元素。

例如：

```text
nums = [3, 2, 1, 5, 6, 4]
k = 2
```

从大到小排列：

```text
[6, 5, 4, 3, 2, 1]
```

第 `2` 大是 `5`。

重复元素也要正常计数：

```text
nums = [3, 2, 3, 1, 2, 4, 5, 5, 6]
k = 4
```

从大到小是：

```text
[6, 5, 5, 4, 3, 3, 2, 2, 1]
```

第 `4` 大是 `4`。

## 方法一：大小固定为 K 的最小堆

### 解题思路

维护一个最多只能保存 `k` 个数字的最小堆。

遍历数组时：

1. 把当前数字加入最小堆。
2. 如果堆中数字超过 `k` 个，就删除堆顶最小值。

遍历结束后，堆中保存的是整个数组中最大的 `k` 个数字。由于这是最小堆，堆顶是这 `k` 个数字中最小的，也就是整个数组的第 `k` 大。

例如：

```text
nums = [3, 2, 1, 5, 6, 4]
k = 2
```

最终堆中保留：

```text
[5, 6]
```

其中最小值 `5` 就是第 `2` 大。

### 用到的数据结构及原因

#### 最小堆

最小堆具有两个重要性质：

1. 堆顶始终是堆中最小的数字。
2. 插入元素和删除堆顶的时间复杂度都是 `O(log k)`。

选择最小堆的原因：

1. 只保留最大的 `k` 个数字，不需要对全部元素排序。
2. 每当出现更大的数字，最小堆能快速淘汰当前候选中最小的数字。
3. 额外空间只与 `k` 有关。

### 最小堆的数组下标关系

对于下标 `index`：

```text
父节点：(index - 1) / 2 向下取整
左子节点：index * 2 + 1
右子节点：index * 2 + 2
```

插入后通过向上交换恢复堆结构；删除堆顶后，将最后一个元素放到堆顶，再通过向下交换恢复结构。

### 伪代码

```text
创建空的最小堆

遍历 nums：
    将当前数字加入堆

    如果堆的大小超过 k：
        删除堆顶最小值

返回堆顶
```

### JavaScript 代码

```js
class MinHeap {
  constructor() {
    this.heap = [];
  }

  get size() {
    return this.heap.length;
  }

  peek() {
    return this.heap[0];
  }

  push(value) {
    this.heap.push(value);

    let index = this.heap.length - 1;

    while (index > 0) {
      const parent = Math.floor((index - 1) / 2);

      if (this.heap[parent] <= this.heap[index]) {
        break;
      }

      [this.heap[parent], this.heap[index]] =
        [this.heap[index], this.heap[parent]];
      index = parent;
    }
  }

  pop() {
    if (this.heap.length === 1) {
      return this.heap.pop();
    }

    const minValue = this.heap[0];
    this.heap[0] = this.heap.pop();

    let index = 0;

    while (true) {
      const leftChild = index * 2 + 1;
      const rightChild = index * 2 + 2;
      let smallest = index;

      if (
        leftChild < this.heap.length &&
        this.heap[leftChild] < this.heap[smallest]
      ) {
        smallest = leftChild;
      }

      if (
        rightChild < this.heap.length &&
        this.heap[rightChild] < this.heap[smallest]
      ) {
        smallest = rightChild;
      }

      if (smallest === index) {
        break;
      }

      [this.heap[index], this.heap[smallest]] =
        [this.heap[smallest], this.heap[index]];
      index = smallest;
    }

    return minValue;
  }
}

/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var findKthLargest = function (nums, k) {
  const minHeap = new MinHeap();

  for (const number of nums) {
    minHeap.push(number);

    if (minHeap.size > k) {
      minHeap.pop();
    }
  }

  return minHeap.peek();
};
```

### 复杂度分析

设数组长度为 `n`：

- 时间复杂度：`O(n log k)`。每个数字最多进行一次堆插入和一次删除。
- 空间复杂度：`O(k)`。堆中最多保存 `k` 个数字。

## 为什么使用最小堆而不是最大堆

我们希望始终保留最大的 `k` 个数字。当候选数量超过 `k` 时，需要淘汰其中最小的数字，所以最小堆最合适。

如果使用最大堆，堆顶是最大的数字，但我们真正需要快速删除的是候选中的最小数字。

## 方法二：快速选择

### 解题思路

如果数组按升序排列，第 `k` 大元素的下标是：

```text
nums.length - k
```

例如长度为 `6`，第 `2` 大在升序数组中的下标是：

```text
6 - 2 = 4
```

快速选择使用快速排序的分区操作：

1. 选择一个基准值 `pivot`。
2. 把小于或等于基准值的元素放到左边，大于基准值的元素放到右边。
3. 基准值归位后，判断它的下标与目标下标的关系。
4. 只继续处理包含目标下标的那一半。

不需要把整个数组完全排序。

### 用到的数据结构及原因

快速选择直接在原数组中交换元素，只使用左右边界和若干下标变量，额外空间为 `O(1)`。

### 为什么随机选择基准值

如果始终选择固定位置作为基准值，在已经有序或构造特殊的输入上，分区可能极不均衡，退化为 `O(n²)`。

随机选择基准值可以降低持续出现最差分区的概率。

### JavaScript 代码

```js
var partition = function (nums, left, right) {
  const randomIndex =
    left + Math.floor(Math.random() * (right - left + 1));

  [nums[randomIndex], nums[right]] =
    [nums[right], nums[randomIndex]];

  const pivot = nums[right];
  let storeIndex = left;

  for (let i = left; i < right; i++) {
    if (nums[i] <= pivot) {
      [nums[i], nums[storeIndex]] =
        [nums[storeIndex], nums[i]];
      storeIndex++;
    }
  }

  [nums[storeIndex], nums[right]] =
    [nums[right], nums[storeIndex]];

  return storeIndex;
};

/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var findKthLargest = function (nums, k) {
  const targetIndex = nums.length - k;
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    const pivotIndex = partition(nums, left, right);

    if (pivotIndex === targetIndex) {
      return nums[pivotIndex];
    }

    if (pivotIndex < targetIndex) {
      left = pivotIndex + 1;
    } else {
      right = pivotIndex - 1;
    }
  }
};
```

### 复杂度分析

- 平均时间复杂度：`O(n)`。每轮通常只继续处理一部分数组。
- 最坏时间复杂度：`O(n²)`。连续出现极端不均衡的分区时会退化。
- 空间复杂度：`O(1)`。迭代写法直接修改原数组。

## 两种方法如何选择

| 方法 | 时间复杂度 | 空间复杂度 | 特点 |
| --- | --- | --- | --- |
| 大小为 `k` 的最小堆 | `O(n log k)` | `O(k)` | 性能稳定，适合数据流和只保留前 `k` 大 |
| 快速选择 | 平均 `O(n)` | `O(1)` | 平均更快，但会修改原数组且最坏为 `O(n²)` |

学习时建议先理解最小堆为什么只保留最大的 `k` 个数字，再学习快速选择如何避免完整排序。

## 容易出错的地方

### 1. 把第 K 大理解成第 K 个不同元素

重复数字要正常参与排名。例如 `[5, 5, 4]` 的第 `2` 大仍然是 `5`。

### 2. 最小堆大小超过 K 后没有删除堆顶

堆必须始终最多保留 `k` 个元素。超过时删除的正是候选中最小的元素。

### 3. 最后从最小堆中取最大值

遍历结束后，堆顶最小值就是第 `k` 大，不需要继续寻找最大值。

### 4. 快速选择目标下标写错

按升序排名时，第 `k` 大的下标是：

```text
nums.length - k
```

### 5. 分区后两边都继续处理

快速选择只需要继续处理目标下标所在的一边，这也是它比完整快速排序更快的原因。

### 6. 直接排序但忘记数字比较函数

如果使用排序法，JavaScript 必须写数字比较函数：

```js
nums.sort((a, b) => b - a);
```

直接调用 `sort()` 会按照字符串规则排序。
