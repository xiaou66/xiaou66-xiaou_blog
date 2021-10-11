---
title: 广度优先搜索
date: 2021/09/20 20:00
math: true
categories:
  - [algorithm]
tags:
  - [java]
---
# BFS(广度优先搜索)

## 主要解决问题

- 遍历树结构
- 遍历图结构
- 遍历二维数据
- 用来搜索最短路径的解

## 模板



## 题目

### lc 二叉树的最小深度

#### 题目

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明：**叶子节点是指没有子节点的节点。

![](https://assets.leetcode.com/uploads/2020/10/12/ex_depth.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：2
```

```
输入：root = [2,null,3,null,4,null,5,null,6]
输出：5
```

**提示：**

- 树中节点数的范围在 `[0, 105]` 内
- `-1000 <= Node.val <= 1000`

#### 分析

#### 代码

```java
public int minDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int depth = 1;
    Queue<TreeNode> q = new LinkedList<TreeNode>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        // 当前队列中所有节点向子节点扩散
        for (int i = 0; i < size; i++) {
            TreeNode cur = q.poll();
            // 既没有左节点又没有右节点那必定是叶子节点
            if (cur.left == null && cur.right == null) {
                return depth;
            }
            // 有左节点就往左边走
            if (cur.left != null) {
                q.offer(cur.left);
            }
            // 有右节点就往右边走
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
        // 增加深度
        depth++;
    }
    return depth;
}
```

### lc 102 二叉树的层序遍历

#### 题目

给你一个二叉树，请你返回其按 **层序遍历** 得到的节点值。 （即逐层地，从左到右访问所有节点）。

**示例：**
二叉树：`[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层序遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

#### 分析

标准模板题

#### 代码

````java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) {
        return res;
    }
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        List<Integer> row = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode cur = q.poll();
            row.add(cur.val);
            if (cur.left != null) {
                q.offer(cur.left);
            }
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
        res.add(row);
    }
    return res;
}
````

### lc 127 单词接龙

#### 题目

字典 `wordList` 中从单词 `beginWord` 和 `endWord` 的 **转换序列** 是一个按下述规格形成的序列：

- 序列中第一个单词是 `beginWord` 。
- 序列中最后一个单词是 `endWord` 。
- 每次转换只能改变一个字母。
- 转换过程中的中间单词必须是字典 `wordList` 中的单词。

给你两个单词 `beginWord` 和 `endWord` 和一个字典 `wordList` ，找到从 `beginWord`到 `endWord` 的 **最短转换序列** 中的 **单词数目** 。如果不存在这样的转换序列，返回 0。

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。
```

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
输出：0
解释：endWord "cog" 不在字典中，所以无法进行转换。
```

#### 分析

![](https://assets.leetcode-cn.com/solution-static/127/2.png)

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1613975776786-1613975776779.png)

#### 代码

**暴力解法**

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    // 字典 set
    Set<String> set = new HashSet<>(wordList);
    Queue<String> q = new LinkedList<>();
    q.offer(beginWord);
    int step = 1, N = beginWord.length();
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            String cur = q.poll();
            char[] chs = cur.toCharArray();
            // 如果已经找到答案
            if (cur.equals(endWord)) {
                return step;
            }
            // 对字符串每一个字符都尝试替换如果替换
            // 如果在字典中那么就将替换后字符串加入到队列中
            // 如果替换后等于最后最后答案就直接 step + 1 返回
            for (int j = 0; j < N; j++) {
                for (char k = 'a'; k <= 'z'; k++) {
                    char pre = chs[j];
                    chs[j] = k;
                    String next = String.valueOf(chs);
                    if (set.contains(next)) {
                        if (next.equals(endWord)) {
                            return step + 1;
                        }
                        set.remove(next);
                        q.offer(next);
                    }
                    chs[j] = pre;
                }
            }
        }
        step++;
    }
    return 0;
}
```

双向BFS

```java
public int ladderLength2(String beginWord, String endWord, List<String> wordList) {
    Set<String> beginSet = new HashSet<String>();
    Set<String> endSet = new HashSet<String>();
    Set<String> wordSet = new HashSet<String>(wordList);
    Set<String> visited = new HashSet<String>();
    if (!wordSet.contains(endWord)) {
        return 0;
    }
    int step = 1, N = endWord.length();
    beginSet.add(beginWord);
    endSet.add(endWord);
    while (!beginSet.isEmpty() && !endSet.isEmpty()) {
        Set<String> nextSet = new HashSet<String>();
        // 遍历所有 beginSet 确定是否到达了 endSet
        for (String word : beginSet) {
            char[] chs = word.toCharArray();
            for (int i = 0; i < N; i++) {
                for (char c = 'a'; c <= 'z'; c++) {
                    char pre = chs[i];
                    chs[i] = c;
                    String next = new String(chs);
                    // 当 next 字符出现 endSet 证明 只要在做一步就完成单词接龙
                    if (endSet.contains(next)) {
                        return step + 1;
                    }
                    // 没有访问过了字符串并包含在字典中的单词才可以加入 nextSet
                    if (visited.add(next) && wordSet.contains(next)) {
                        nextSet.add(next);
                    }
                    chs[i] = pre;
                }
            }
        }
        // 每一次优先遍历 set 较小的这样 beginSet 和 endSet 将往中间靠拢
        if (endSet.size() < nextSet.size()) {
            beginSet = endSet;
            endSet = nextSet;
        } else {
            beginSet = nextSet;
        }

        step++;
    }
    return 0;
}
```

### lc 迷宫

#### 题目

由空地和墙组成的迷宫中有一个**球**。球可以向**上下左右**四个方向滚动，但在遇到墙壁前不会停止滚动。当球停下时，可以选择下一个方向。

给定球的**起始位置，目的地**和**迷宫**，判断球能否在目的地停下。

迷宫由一个0和1的二维数组表示。 1表示墙壁，0表示空地。你可以假定迷宫的边缘都是墙壁。起始位置和目的地的坐标通过行号和列号给出。

```
输入 1: 迷宫由以下二维数组表示
0 0 1 0 0
0 0 0 0 0
0 0 0 1 0
1 1 0 1 1
0 0 0 0 0
输入 2: 起始位置坐标 (rowStart, colStart) = (0, 4)
输入 3: 目的地坐标 (rowDest, colDest) = (4, 4)
输出: true
解析: 一个可能的路径是 : 左 -> 下 -> 左 -> 下 -> 右 -> 下 -> 右。
```

![](https://assets.leetcode.com/uploads/2018/10/12/maze_1_example_1.png)

```
输入 1: 迷宫由以下二维数组表示

0 0 1 0 0
0 0 0 0 0
0 0 0 1 0
1 1 0 1 1
0 0 0 0 0
输入 2: 起始位置坐标 (rowStart, colStart) = (0, 4)
输入 3: 目的地坐标 (rowDest, colDest) = (3, 2)
输出: false
解析: 没有能够使球停在目的地的路径。
```

![](https://assets.leetcode.com/uploads/2018/10/13/maze_1_example_2.png)

**注意:**

1. 迷宫中只有一个球和一个目的地。
2. 球和目的地都在空地上，且初始时它们不在同一位置。
3. 给定的迷宫不包括边界 (如图中的红色矩形), 但你可以假设迷宫的边缘都是墙壁。
4. 迷宫至少包括2块空地，行数和列数均不超过100。

#### 分析

#### 代码

```java
public boolean hasPath(int[][] maze, int[] start, int[] destination) {
    int[][] dirs = { { 0, 1 }, { 0, -1 }, { -1, 0 }, { 1, 0 } };
    boolean[][] visited = new boolean[maze.length][maze[0].length];
    Queue<int[]> q = new LinkedList<>();
    q.add(start);
    // 将起始点设置已经被访问
    visited[start[0]][start[1]] = true;
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        if (cur[0] == destination[0] && cur[1] == destination[1]) {
            return true;
        }
        for (int[] dir : dirs) {
            int x = cur[0] + dir[0];
            int y = cur[1] + dir[1];
            // 按题目小球一旦移动要撞到墙才会停止
            while (x >= 0 && y >= 0 && x < maze.length &&
                   y < maze[0].length && maze[x][y] == 0) {
                x += dir[0];
                y += dir[1];
            }
            // 上面循环走完这时球应该在墙中,要减掉一个位置
            x -= dir[0];
            y -= dir[1];
            // 判断当前位置是否已经访问过
            if (!visited[x][y]) {
                q.add(new int[] { x, y });
                visited[x][y] = true;
            }
        }
    }
    return false;
}
```

### lc 505 迷宫II

#### 题目

由空地和墙组成的迷宫中有一个**球**。球可以向**上下左右**四个方向滚动，但在遇到墙壁前不会停止滚动。当球停下时，可以选择下一个方向。

给定球的**起始位置，目的地**和**迷宫，**找出让球停在目的地的最短距离。距离的定义是球从起始位置（不包括）到目的地（包括）经过的**空地**个数。如果球无法停在目的地，返回 -1。

迷宫由一个0和1的二维数组表示。 1表示墙壁，0表示空地。你可以假定迷宫的边缘都是墙壁。起始位置和目的地的坐标通过行号和列号给出。
```
输入 1: 迷宫由以下二维数组表示
0 0 1 0 0
0 0 0 0 0
0 0 0 1 0
1 1 0 1 1
0 0 0 0 0
输入 2: 起始位置坐标 (rowStart, colStart) = (0, 4)
输入 3: 目的地坐标 (rowDest, colDest) = (4, 4)
输出: 12
解析: 一条最短路径 : left -> down -> left -> down -> right -> down -> right。
             总距离为 1 + 1 + 3 + 1 + 2 + 2 + 2 = 12。
```

![](https://assets.leetcode.com/uploads/2018/10/12/maze_1_example_1.png)

```
输入 1: 迷宫由以下二维数组表示

0 0 1 0 0
0 0 0 0 0
0 0 0 1 0
1 1 0 1 1
0 0 0 0 0
输入 2: 起始位置坐标 (rowStart, colStart) = (0, 4)
输入 3: 目的地坐标 (rowDest, colDest) = (3, 2)
输出: -1
解析: 没有能够使球停在目的地的路径。
```

![](https://assets.leetcode.com/uploads/2018/10/13/maze_1_example_2.png)
#### 分析

#### 代码

 ```java
public int shortestDistance(int[][] maze, int[] start, int[] destination) {
    int[][] distance = new int[maze.length][maze[0].length];
    for (int[] row : distance) {
        Arrays.fill(row, Integer.MAX_VALUE);
    }
    // 保存距离
    distance[start[0]][start[1]] = 0;
    dijkstra(maze, start, distance);
    return distance[destination[0]][destination[1]] == Integer.MAX_VALUE ? -1
        : distance[destination[0]][destination[1]];
}

public void dijkstra(int[][] maze, int[] start, int[][] distance) {
    int[][] dirs = { { 0, 1 }, { 0, -1 }, { -1, 0 }, { 1, 0 } };
    PriorityQueue<int[]> q = new PriorityQueue<>((a, b) -> a[2] - b[2]);
    q.offer(new int[] { start[0], start[1], 0 });
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        for (int[] dir : dirs) {
            int x = cur[0] + dir[0];
            int y = cur[1] + dir[1];
            int count = 0;
            while (x >= 0 && y >= 0 && x < maze.length &&
                   y < maze[0].length && maze[x][y] == 0) {
                x += dir[0];
                y += dir[1];
                count++;
            }
            x -= dir[0];
            y -= dir[1];
            // 起点到达该点的距离 + 当前点到达的距离 < 当前点存在的距离
            if (distance[cur[0]][cur[1]] + count < distance[x][y]) {
                distance[x][y] = distance[cur[0]][cur[1]] + count;
                q.add(new int[] { x, y, distance[x][y] });
            }
        }
    }
}
 ```



```java
//构造路径记录和路径长度数组
        int n = maze.length;
        int m = maze[0].length;
        String[][] way = new String[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                way[i][j] = "";
            }
        }

        int[][] dp = new int[n][m];
        int[] dx = new int[]{-1,1,0,0};
        int[] dy = new int[]{0,0,-1,1};

        String[] dircectStr = new String[]{"u", "d", "l", "r"};
        //构造队列
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(ball);

        while (!queue.isEmpty()){
            int[] cur = queue.poll();
            for (int direction = 0; direction < 4; direction++) {
                int nx = cur[0], ny = cur[1];
                while (nx >= 0 && nx < n && ny >= 0 && ny < m && maze[nx][ny] == 0){
                    if (nx == hole[0] && ny == hole[1]){
                        nx += dx[direction];
                        ny += dy[direction];
                        break;
                    }
                    nx += dx[direction];
                    ny += dy[direction];
                }
                nx -= dx[direction];
                ny -= dy[direction];
                int steps = dp[cur[0]][cur[1]] + Math.abs(nx - cur[0]) + Math.abs(ny - cur[1]);
                //非当前位置，未初始化，路径长度小于当前位置的原来的值，路径长度相等，字典序更小
                if (!(nx == cur[0] && ny == cur[1]) && (dp[nx][ny] == 0 || (dp[nx][ny] > steps || (dp[nx][ny] == steps && (way[cur[0]][cur[1]] + dircectStr[direction]).compareTo(way[nx][ny]) < 0)))){
                    dp[nx][ny] = steps;
                    way[nx][ny] = way[cur[0]][cur[1]] + dircectStr[direction];
                    if (!(nx == hole[0] && ny == hole[1])){
                        queue.offer(new int[]{nx, ny});
                    }
                }
            }
        }

        return way[hole[0]][hole[1]].equals("") ? "impossible" : way[hole[0]][hole[1]];

```

###  lc 207 课程表

#### 题目

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。
```

```
输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前，你需要先完成​课程 0 ；并且学习课程 0 之前，你还应先完成课程 1 。这是不可能的。
```

#### 分析

#### 代码

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    // 入度
    int[] indegree = new int[numCourses];
    // 建图
    for (int i = 0; i < prerequisites.length; i++) {
        int end = prerequisites[i][0];
        int start = prerequisites[i][1];
        graph.computeIfAbsent(start, x -> new ArrayList<>()).add(end);
        indegree[end]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < indegree.length; i++) {
        // 将入度为 0 节点加入到队列中
        // 因为入度0的节点为头节点
        if (indegree[i] == 0) {
            q.add(i);
        }
    }
    int count = 0;
    while (!q.isEmpty()) {
        int cur = q.poll();
        count++;
        for (int nei : graph.getOrDefault(cur, new ArrayList<>())) {
            // 当入度减到0时为开始节点
            if (--indegree[nei] == 0) {
                q.offer(nei);
            }
        }

    }
    return count == numCourses;
}
```



### lc 210 课程表 II

#### 题目

现在你总共有 *n* 门课需要选，记为 `0` 到 `n-1`。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: `[0,1]`

给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。

可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。

```
输入: 2, [[1,0]] 
输出: [0,1]
解释: 总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1] 。
```

```
输入: 4, [[1,0],[2,0],[3,1],[3,2]]
输出: [0,1,2,3] or [0,2,1,3]
解释: 总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
     因此，一个正确的课程顺序是 [0,1,2,3] 。另一个正确的排序是 [0,2,1,3] 。
```

#### 分析

#### 代码

```JAVA
public int[] findOrder(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    List<Integer> result = new ArrayList<>();
    int[] indegree = new int[numCourses];
    for (int i = 0; i < prerequisites.length; i++) {
        int end = prerequisites[i][0];
        int start = prerequisites[i][1];
        graph.computeIfAbsent(start, x -> new ArrayList<>()).add(end);
        indegree[end]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < indegree.length; i++) {
        // 将入度为 0 节点加入到队列中
        // 因为入度0的节点为头节点
        if (indegree[i] == 0) {
            q.offer(i);
        }
    }
    while (!q.isEmpty()) {
        int cur = q.poll();
        result.add(cur);
        for (int nei : graph.getOrDefault(cur, new ArrayList<>())) {
            // 当入度减到0时为开始节点
            if (--indegree[nei] == 0) {
                q.offer(nei);
            }
        }
    }
    return result.size() == numCourses ? new int[] {} : 	result.stream().mapToInt(Integer::valueOf).toArray();
}
```



