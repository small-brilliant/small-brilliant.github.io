---
title: BFS，DFS的区别及应用场景
tags: 
- 算法
- 数据结构
categories: 
- 学习记录
- 数据结构
cover: https://images.pexels.com/photos/9754/mountains-clouds-forest-fog.jpg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2022-05-07 11:10:50
updated: 2022-05-07 11:10:53
keywords: 算法
---
**1.**如果只是要找到某一个结果是否存在，那么DFS会更高效。因为DFS会首先把一种可能的情况尝试到底，才会回溯去尝试下一种情况，只要找到一种情况，就可以返回了。但是BFS必须所有可能的情况同时尝试，在找到一种满足条件的结果的同时，也尝试了很多不必要的路径；

**2.**如果是要找所有可能结果中最短的，那么BFS回更高效。因为DFS是一种一种的尝试，在把所有可能情况尝试完之前，无法确定哪个是最短，所以DFS必须把所有情况都找一遍，才能确定最终答案（DFS的优化就是剪枝，不剪枝很容易超时）。而BFS从一开始就是尝试所有情况，所以只要找到第一个达到的那个点，那就是最短的路径，可以直接返回了，其他情况都可以省略了，所以这种情况下，BFS更高效。

#### [1091. 二进制矩阵中的最短路径](https://leetcode-cn.com/problems/shortest-path-in-binary-matrix/)

```go
func shortestPathBinaryMatrix(grid [][]int) int {
    n := len(grid)
    dircts := [][]int{{-1,0},{-1,1},{0,1},{1,1},{1,0},{1,-1},{0,-1},{-1,-1}}
    visi := make([][]bool,n)
    if grid[0][0] == 1{return -1}
    for i:=0;i<n;i++{
        visi[i]=make([]bool,n)
    }
    queue := make([][]int,0)
    queue = append(queue,[]int{0,0})
    count := 0
    for len(queue)!=0{
        size := len(queue)
        count ++;
        for i:=0 ; i< size;i++{
            node := queue[0]
            queue = queue[1:]
            if node[0]==n-1 && node[1]==n-1{return count}
            for _,dirct := range dircts{
                x,y := node[0]+dirct[0],node[1]+dirct[1]
                if x>=0 && x<n && y>=0 && y<n && grid[x][y]!=1 && !visi[x][y]{
                    queue = append(queue,[]int{x,y})
                    visi[x][y]=true
                }
            }
        }
    }
    return -1
}
```

