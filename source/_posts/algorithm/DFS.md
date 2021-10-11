---
title: 深度优先搜索
date: 2021/09/20 20:00
math: true
categories:
  - [algorithm]
tags:
  - [java]
---

# DFS(深度优先搜索)

## 主要解决问题

- 二分搜索
- 大整数乘法
- Strassen矩阵乘法
- 棋盘覆盖
- 合并排序
- 快速排序
- 线性时间选择
- 最接近点对问题
- 循环赛日程表
- 汉诺塔

## 模板

## 题目

### lc 78 子集
#### 题目

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

```
输入：nums = [0]
输出：[[],[0]]
```

#### 分析

#### 代码

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrackList(nums, res, new ArrayList<>(), 0);
    return res;
}

public void backtrackList(int[] nums, List<List<Integer>> res, List<Integer> temp, int start) {
    res.add(new ArrayList<>(temp));
    for (int i = start; i < nums.length; i++) {
        // 如果tempList没有包含nums[i]才添加
        temp.add(nums[i]);
        // 递归调用，此时的tempList一直在变化，直到满足终结条件才结束
        backtrackList(nums, res, temp, i + 1);
        // 它移除tempList最后一个元素的作用就是返回上一次调用时的数据
        // 也就是希望返回之前的节点再去重新搜索满足条件。这样才能实现回溯
        temp.remove(temp.size() - 1);
    }
}
```

**mask 方法**

```java
public List<List<Integer>> subsets2(int[] nums) {
    List<List<Integer>> res = new ArrayList<>(nums.length << 1 - 1);
    int totalNumber = 1 << nums.length;
    for (int mask = 0; mask < totalNumber; mask++) {
        List<Integer> set = new ArrayList<>();
        for (int j = 0; j < nums.length; j++) {
            if ((mask & (1 << j)) != 0) {
                set.add(nums[j]);
            }
        }
        res.add(set);
    }
    return res;
}
```

### lc 90 子集 II

#### 题目

给定一个可能包含重复元素的整数数组 ***nums***，返回该数组所有可能的子集（幂集）。

**说明：**解集不能包含重复的子集。

```
输入: [1,2,2]
输出:
[
  [2],
  [1],
  [1,2,2],
  [2,2],
  [1,2],
  []
]
```

#### 分析

与上题基本一致主要增加要先排序因为这样更好判断重复数字。

#### 代码

```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    backtrackList(nums, res, new ArrayList<>(), 0);
    return res;
}

public void backtrackList(int[] nums, List<List<Integer>> res, List<Integer> temp, int start) {
    res.add(new ArrayList<>(temp));
    for (int i = start; i < nums.length; i++) {
        // 条件
        if (i != start && nums[i] == nums[i - 1]) {
            continue;
        }
        // 如果tempList没有包含nums[i]才添加
        temp.add(nums[i]);
        // 递归调用，此时的tempList一直在变化，直到满足终结条件才结束
        backtrackList(nums, res, temp, i + 1);
        // 它移除tempList最后一个元素的作用就是返回上一次调用时的数据
        // 也就是希望返回之前的节点再去重新搜索满足条件。这样才能实现回溯
        temp.remove(temp.size() - 1);
    }
}
```

### lc46 全排列

#### 题目

给定一个 **没有重复** 数字的序列，返回其所有可能的全排列。

```
输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

#### 分析

#### 代码

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(res, new ArrayList<>(), nums);
    return res;
}

public void backtrack(List<List<Integer>> res, List<Integer> temp, int[] nums) {
    if (temp.size() == nums.length) {
        res.add(new ArrayList<>(temp));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (temp.contains(nums[i])) {
            continue;
        }
        temp.add(nums[i]);
        backtrack(res, temp, nums);
        temp.remove(temp.size() - 1);
    }
}
```

### lc 解数独

#### 题目

编写一个程序，通过填充空格来解决数独问题。

一个数独的解法需遵循如下规则：

数字 1-9 在每一行只能出现一次。
数字 1-9 在每一列只能出现一次。
数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。
空白格用 '.' 表示。

一个数独。

![](http://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Sudoku-by-L2G-20050714.svg/250px-Sudoku-by-L2G-20050714.svg.png)

答案被标成红色。

![img](http://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Sudoku-by-L2G-20050714_solution.svg/250px-Sudoku-by-L2G-20050714_solution.svg.png)

**提示：**

- 给定的数独序列只包含数字 `1-9` 和字符 `'.'` 。
- 你可以假设给定的数独只有唯一解。
- 给定数独永远是 `9x9` 形式的。

#### 分析

#### 代码

```java
public void solveSudoku(char[][] board) {
    solve(board);
}

public boolean solve(char[][] board) {
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[i].length; j++) {
            if (board[i][j] != '.') {
                continue;
            }
            for (char c = '1'; c <= '9'; c++) {
                if (isValid(board, i, j, c)) {
                    board[i][j] = c;
                    if (solve(board)) {
                        return true;
                    } else {
                        board[i][j] = '.';
                    }
                }
            }
            // 如果递归完都没有找到合适解直接返回 false
            return false;
        }
    }
    // 全部情况多走完就返回 true
    return true;
}

public boolean isValid(char[][] board, int i, int j, char c) {
    // 扫描行
    for (int row = 0; row < 9; row++) {
        if (board[row][j] == c) {
            return false;
        }
    }
    // 扫描列
    for (int col = 0; col < 9; col++) {
        if (board[i][col] == c) {
            return false;
        }
    }
    // 扫描 3 * 3 小矩形
    for (int row = (i / 3) * 3; row < (i / 3) * 3 + 3; row++) {
        for (int col = (j / 3) * 3; col < (j / 3) * 3 + 3; row++) {
            if (board[row][col] == c) {
                return false;
            }
        }
    }
    return true;
}
```

### lc 51 [N 皇后](https://leetcode-cn.com/problems/n-queens/)

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

![](https://assets.leetcode.com/uploads/2020/11/13/queens.jpg)

```
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
解释：如上图所示，4 皇后问题存在两个不同的解法。
```

```
输入：n = 1
输出：[["Q"]]
```

 ```java
public List<List<String>> solveNQueens(int n) {
    char[][] board = new char[n][n];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            board[i][j] = '.';
        }
    }
    List<List<String>> res = new ArrayList<>();
    dfs(board, 0, res);
    return res;
}

public void dfs(char[][] board, int colIndex, List<List<String>> res) {
    if (colIndex == board.length) {
        // 完成一个解
        res.add(construct(board));
    }
    for (int i = 0; i < board.length; i++) {
        if (validate(board, i, colIndex)) {
            board[i][colIndex] = 'Q';
            dfs(board, colIndex + 1, res);
            board[i][colIndex] = '.';
        }
    }

}

public List<String> construct(char[][] board) {
    List<String> res = new ArrayList<>();
    for (int i = 0; i < board.length; i++) {
        res.add(new String(board[i]));
    }
    return res;
}

public boolean validate(char[][] board, int x, int y) {
    // 和排好的皇后进行比较判断有没有重复的
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < y; j++) {
            if (board[i][j] == 'Q' && (x + j == y + i || x + y == i + j || x == i)) {
                return false;
            }
        }
    }
    return true;
}
 ```

