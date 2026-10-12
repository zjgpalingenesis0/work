# 200. 岛屿数量

- LeetCode：[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)
- 难度：中等

## 先理解题意

网格中：

```text
"1" 表示陆地
"0" 表示水
```

上下左右相邻的陆地属于同一座岛。斜着相邻不算连通。

例如：

```text
1 1 0 0
1 0 0 1
0 0 1 1
```

左上角的三个 `1` 互相连通，是一座岛；右侧的三个 `1` 互相连通，是另一座岛，所以答案是 `2`。

## 解题思路

从左到右、从上到下遍历整个网格：

- 遇到水 `"0"`，直接跳过。
- 遇到尚未处理的陆地 `"1"`，说明发现了一座新岛，答案加一。
- 从这块陆地出发，通过深度优先搜索找到所有上下左右相连的陆地，并把它们标记为已访问。

标记完成后，这座岛中的其他陆地不会再次被统计。

这个过程也常被称为“沉岛”：发现一座岛后，把整座岛的 `"1"` 都改成 `"0"`。

## 为什么发现一个 `1` 就能让答案加一

遍历到 `"1"` 时，它不可能属于之前统计过的岛。因为之前发现岛屿时，已经把该岛所有相连的陆地都标记成了 `"0"`。

因此，当前仍然是 `"1"` 的位置一定属于一座尚未统计的新岛。

## 用到的数据结构及原因

### 栈 `stack`

栈中保存接下来需要继续搜索的网格坐标：

```text
[row, column]
```

选择显式栈的原因：

1. 可以实现深度优先搜索。
2. JavaScript 递归层数过深时可能发生调用栈溢出，显式栈更加稳定。
3. `push()` 和 `pop()` 都是数组末尾操作，效率较高。

### 方向数组 `directions`

保存上下左右四个移动方向：

```text
上：[-1, 0]
下：[1, 0]
左：[0, -1]
右：[0, 1]
```

这样可以用一个循环统一检查四个相邻位置，避免重复写四段相似代码。

### 原网格 `grid`

直接把访问过的陆地从 `"1"` 改成 `"0"`，利用原网格充当访问记录，不需要额外创建 `visited` 矩阵。

这种方式会修改输入网格。如果题目要求保留原数据，则需要额外的访问集合或布尔矩阵。

## 用到的方法及原因

### 深度优先搜索 DFS

从一块陆地开始，不断访问与它相邻的陆地，直到整座岛都被标记。

选择 DFS 的原因：

1. 一座岛就是网格中的一个连通区域。
2. DFS 能从一个起点访问整个连通区域。
3. 每个网格位置最多被处理一次。

### 边界检查

访问相邻位置前，必须保证：

```text
0 <= nextRow < rowCount
0 <= nextColumn < columnCount
```

否则会访问网格之外的位置。

### 加入栈前立即标记

发现相邻陆地时，应先把它改成 `"0"`，再加入栈：

```text
grid[nextRow][nextColumn] = "0"
stack.push([nextRow, nextColumn])
```

如果等到弹出栈时才标记，同一个位置可能被多个相邻节点重复加入栈中。

## 过程示例

网格：

```text
1 1 0
1 0 1
0 0 1
```

遍历到左上角第一个 `1`：

```text
islandCount = 1
```

从它开始搜索，把左上角相连的陆地全部沉掉：

```text
0 0 0
0 0 1
0 0 1
```

继续遍历，遇到右侧的 `1`：

```text
islandCount = 2
```

把右侧整座岛沉掉：

```text
0 0 0
0 0 0
0 0 0
```

最终答案是 `2`。

## 伪代码

```text
islandCount = 0

遍历网格中的每个位置：
    如果当前位置不是陆地：
        跳过

    发现新岛，islandCount 加 1
    将当前位置标记成水
    把当前位置加入栈

    当栈不为空：
        弹出一个位置

        检查它的上下左右：
            如果相邻位置没有越界，并且是陆地：
                将相邻位置标记成水
                把相邻位置加入栈

返回 islandCount
```

## JavaScript 代码

```js
/**
 * @param {character[][]} grid
 * @return {number}
 */
var numIslands = function (grid) {
  const rowCount = grid.length;
  const columnCount = grid[0].length;
  const directions = [
    [-1, 0],
    [1, 0],
    [0, -1],
    [0, 1]
  ];

  let islandCount = 0;

  for (let row = 0; row < rowCount; row++) {
    for (let column = 0; column < columnCount; column++) {
      if (grid[row][column] !== "1") {
        continue;
      }

      islandCount++;
      grid[row][column] = "0";

      const stack = [[row, column]];

      while (stack.length > 0) {
        const [currentRow, currentColumn] = stack.pop();

        for (const [rowChange, columnChange] of directions) {
          const nextRow = currentRow + rowChange;
          const nextColumn = currentColumn + columnChange;

          if (
            nextRow >= 0 &&
            nextRow < rowCount &&
            nextColumn >= 0 &&
            nextColumn < columnCount &&
            grid[nextRow][nextColumn] === "1"
          ) {
            grid[nextRow][nextColumn] = "0";
            stack.push([nextRow, nextColumn]);
          }
        }
      }
    }
  }

  return islandCount;
};
```

## 复杂度分析

假设网格有 `m` 行、`n` 列：

- 时间复杂度：`O(m × n)`。每个位置最多被访问和标记一次。
- 空间复杂度：最坏为 `O(m × n)`。当大量陆地同时进入 DFS 栈时，栈可能保存很多坐标。

## DFS 和 BFS 都可以吗

可以。这个问题的本质是找到网格中的连通区域：

- DFS 使用栈，优先沿一条路径深入。
- BFS 使用队列，一层一层向周围扩散。

两者都能访问一座岛中的全部陆地，时间复杂度相同。本笔记使用显式栈，是因为 JavaScript 数组的 `push()` 和 `pop()` 使用方便，并且不会有递归调用栈溢出的风险。

## 容易出错的地方

### 1. 把斜线方向也当成相连

题目只认可上下左右四个方向，斜着接触的两块陆地属于不同岛屿。

### 2. 字符 `"1"` 和数字 `1` 混淆

网格中保存的是字符串字符，应比较：

```js
grid[row][column] === "1"
```

### 3. 发现岛屿后只标记起点

必须搜索并标记与起点相连的整座岛，否则同一座岛的其他位置会被重复计数。

### 4. 加入栈后才等待标记

应该在加入栈时立即标记，避免同一个位置被不同邻居重复加入。

### 5. 忘记检查上下左右是否越界

每次访问 `grid[nextRow][nextColumn]` 前都要先检查行列范围。

### 6. 忽略算法会修改输入网格

沉岛法会把陆地改成水。如果后续仍需要原始网格，应复制网格或使用独立的 `visited` 记录。

### 7. 使用递归时忽略 JavaScript 调用栈限制

递归 DFS 写法更短，但大型连通岛屿可能导致调用栈过深。显式栈更稳妥。
