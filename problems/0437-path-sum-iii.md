# 437. 路径总和 III

- LeetCode：[437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)
- 难度：中等

## 先理解题意

需要统计二叉树中有多少条路径的节点值之和等于 `targetSum`。

路径需要满足：

1. 必须从父节点走向子节点，也就是只能向下。
2. 不一定从根节点开始。
3. 不一定在叶子节点结束。

例如一条根到当前节点的路径是：

```text
10 -> 5 -> 3 -> 2
```

其中下面这些都属于合法的向下路径：

```text
10 -> 5 -> 3
5 -> 3 -> 2
3 -> 2
2
```

但不能先向下再返回父节点，也不能从一个分支跨到另一个分支。

## 解题思路

这道题可以把“一维数组的前缀和”思想应用到树的根到当前节点路径上。

从根节点走到当前节点时，维护：

```text
currentSum = 根节点到当前节点的节点值总和
```

假设当前前缀和是 `currentSum`，之前路径上的某个前缀和是 `previousSum`，那么两者之间那段路径的和是：

```text
currentSum - previousSum
```

如果这段路径需要等于 `targetSum`：

```text
currentSum - previousSum = targetSum
```

移项后：

```text
previousSum = currentSum - targetSum
```

所以到达每个节点时，只需要查询：

> 当前递归路径上，前缀和 `currentSum - targetSum` 出现过多少次？

出现几次，就表示有几条以当前节点结尾、总和等于目标值的路径。

## 用到的数据结构及原因

### `Map`（前缀和计数表）

保存当前根到节点路径上的：

```text
前缀和 -> 该前缀和出现的次数
```

选择 `Map` 的原因：

1. 可以平均 `O(1)` 查询所需前缀和出现次数。
2. 相同前缀和可能出现多次，所以必须保存次数，而不只是是否存在。
3. 节点值可能为负数，前缀和也可能为负数，`Map` 比普通数组下标更合适。

### 递归调用栈

DFS 沿着当前分支从父节点走向子节点，天然符合题目“路径只能向下”的条件。

递归返回时进行回溯，让 `Map` 始终只保存当前这条根到节点路径上的前缀和。

## 为什么初始化 `0 -> 1`

开始遍历前加入：

```text
前缀和 0 出现 1 次
```

它表示还没有经过任何节点时，路径和为 `0`。

如果从根节点到当前节点的总和恰好等于 `targetSum`：

```text
currentSum - targetSum = 0
```

这条从根开始的路径就能通过初始的 `0 -> 1` 被统计到。

## 为什么 Map 中保存的是次数

同一个前缀和可能在当前路径中出现多次。

例如节点值中含有正数、负数和 `0` 时，不同位置的累计和可能相同。每个相同前缀和都对应一个不同的路径起点，所以需要把它们全部计入答案。

因此存储的是：

```text
prefixSum -> count
```

而不是只使用一个 `Set`。

## 用到的方法及原因

### 深度优先搜索 DFS

沿根到叶子的方向访问节点，并维护当前路径的前缀和。

### 前缀和

利用两个前缀和之差，快速计算一段向下路径的总和，不需要从当前节点重新向上累加。

### 回溯

处理完当前节点的左右子树后，需要从 `Map` 中撤销当前节点的前缀和：

```text
prefixCount[currentSum] 减 1
```

原因是返回父节点后，接下来可能进入另一条兄弟分支。兄弟分支之间不能组成一条向下路径，所以不能让左子树的前缀和影响右子树。

## 为什么必须回溯

假设树结构是：

```text
       root
       /  \
    left  right
```

从 `root` 进入 `left` 时，左分支的前缀和会加入 `Map`。

处理完左分支后，如果不撤销这些前缀和，再进入 `right` 时，就可能错误地把：

```text
左分支中的节点 -> 右分支中的节点
```

当成一条路径。但这种路径需要先向上回到根节点，再向下进入右分支，不符合题目要求。

所以 `Map` 记录的是“当前递归路径”，不是“整棵树中所有访问过的前缀和”。

## 处理一个节点的顺序

```text
1. 把当前节点值加入 currentSum
2. 查询 currentSum - targetSum 出现次数
3. 把 currentSum 加入 Map
4. 递归处理左子树和右子树
5. 从 Map 中撤销 currentSum
```

必须先统计答案，再加入当前前缀和。否则当 `targetSum = 0` 时，当前前缀和可能与自己配对，形成一条不包含任何节点的错误路径。

## 伪代码

```text
创建 prefixCount Map
记录 0 出现 1 次

定义 dfs(node, currentSum)：
    如果 node 为空：
        返回 0

    currentSum += node.val

    count = prefixCount 中
            currentSum - targetSum 的出现次数

    将 currentSum 的出现次数加 1

    count += dfs(node.left, currentSum)
    count += dfs(node.right, currentSum)

    将 currentSum 的出现次数减 1

    返回 count

返回 dfs(root, 0)
```

## JavaScript 代码

```js
/**
 * @param {TreeNode} root
 * @param {number} targetSum
 * @return {number}
 */
var pathSum = function (root, targetSum) {
  const prefixCount = new Map();
  prefixCount.set(0, 1);

  const dfs = function (node, currentSum) {
    if (node === null) {
      return 0;
    }

    currentSum += node.val;

    const neededPrefix = currentSum - targetSum;
    let count = prefixCount.get(neededPrefix) ?? 0;

    const oldCount = prefixCount.get(currentSum) ?? 0;
    prefixCount.set(currentSum, oldCount + 1);

    count += dfs(node.left, currentSum);
    count += dfs(node.right, currentSum);

    const remainingCount = prefixCount.get(currentSum) - 1;

    if (remainingCount === 0) {
      prefixCount.delete(currentSum);
    } else {
      prefixCount.set(currentSum, remainingCount);
    }

    return count;
  };

  return dfs(root, 0);
};
```

## 复杂度分析

设二叉树有 `n` 个节点，高度为 `h`：

- 时间复杂度：`O(n)`。每个节点访问一次，`Map` 查询与更新平均为 `O(1)`。
- 空间复杂度：`O(h)`。递归栈和当前路径上的前缀和最多与树高同阶；最坏退化链表时为 `O(n)`。

## 与第 1074 题的关系

第 1074 题在一维压缩数组中使用：

```text
当前前缀和 - target
```

寻找符合条件的连续子数组。

本题使用同样的公式，只是“连续数组区间”变成了“树中当前根到节点路径上的连续向下区间”。由于树存在分支，本题还必须通过回溯移除离开分支后的前缀和。

## 容易出错的地方

### 1. 只统计从根节点开始的路径

题目允许路径从任意节点开始。查询 `currentSum - targetSum` 可以统计不从根开始的路径。

### 2. 只使用 `Set`

同一个前缀和可能出现多次，每一次都代表一个不同路径起点，所以必须使用 `Map` 记录出现次数。

### 3. 忘记初始化 `0 -> 1`

这会漏掉所有从根节点开始且总和恰好等于 `targetSum` 的路径。

### 4. 忘记回溯

处理完当前节点的子树后必须把当前前缀和计数减一，否则不同分支可能被错误地拼成一条路径。

### 5. 回溯发生在递归子树之前

当前节点的前缀和必须在处理左右子树期间保持有效，所以只能在左右子树都处理完后撤销。

### 6. 认为出现负数时不能使用前缀和

前缀和差值方法可以处理负数。不能使用普通滑动窗口，正是因为节点值可能为负数。

### 7. 路径可以拐弯

合法路径只能从父节点向子节点移动，不能从左子树经过父节点再进入右子树。
