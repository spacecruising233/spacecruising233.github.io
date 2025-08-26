---
title: 树的直径与两心
tags: [OI, 图论, 树论]
category: OI
description: 关于树的直径，重心与中心
published: 2025-08-27
draft: false
---

:::note

观前提醒：  
本笔记含有大量证明，不想看可选择性跳过

:::

# 树的直径

:::important

以下无特殊说明，所有树的边权（若有）均非负。
见[此处](https://spacecruising233.github.io/posts/tree-diamater/#04-%E6%B1%82%E6%B3%952---%E6%A0%91%E5%BD%A2dp)以求解带负权边的树的直径。

:::

## 01 定义

**树的直径**：树上最长的简单路径。

## 02 性质

1. 直径的两端点一定是叶子节点。

2. 到**任意**一点最远的点一定是直径的某个端点。（证明见下）

3. 在树上加点，每加一点树的直径长度变化量 $\le 1$ 。

4. **树的直径不一定唯一**，所有直径交于直径的中点。

性质1和性质3都很自然，证明比较简单，此处略去不证。

### 证明（性质4）

如图，可说明树的直径不一定唯一。

![](prove2.png)

对于命题的后半部分，考虑反证。

任取两直径，设两直径分别为 $(a, b)$ 和 $(c, d)$ 。（其中 $(a, b)$ 和 $(c, d)$ 不是同一条直径）

#### 情况1：两直径相交，但不交于某一直径的中点

设交点为 $p$ 。

不妨设： $(a, p) > (b, p)$ ， $(c, p) \ge (d, p)$ 。

由于 $(a, b) = (c, d)$ ，我们亦知： $(b, p) < (c, p)$

于是 $(a, c) = (a, p) + (c, p) > (a, p) + (b, p) = (a, b)$ ，与 $(a, c)$ 是直径矛盾。

#### 情况2：两直径不相交

考虑用一条路径 $(m, n)$ 连接两直径，以转成情况1。

> 为了方便，我们定义：   
> $(a, b)$ 到 $(a, c)$ 的岔路口是指同时在两路径上的最后一个节点。

设 $m$ 是 $(a, b)$ 和  $(a, d)$ 的岔路口，$n$ 是 $(d, c)$ 和 $(d, a)$ 的岔路口。

不妨设： $(a, m) \ge (b, m)$ ， $(c, n) \ge (d, n)$ 。

容易发现 $(a, m) \ge (d, n)$ 。

则 $(a, c) = (a, m) + (m, n) + (n, c) > (a, m) + (n, c) \ge (d, n) + (n, c) = (d, c)$ ，与 $(d, c)$ 是直径矛盾。

## 03 求法1 - 两遍DFS

该做法时间复杂度为 $O(n)$ 。

利用性质2，任取一节点为根，第一遍DFS遍历离根最远的点；再以最远的点为根，第二遍DFS再遍历离新根最远的点。

### 正确性证明

#### 直观理解

我们只需证：**第一遍DFS找到的最远的点是直径的一端**。

比如说，起始点不在直径上的情况。（其余情况类似）

我们把树的直径标为红色，假设这种方法找到的不是直径的一端：

![](prove1.png)

那么橙色路径会取代红色路径，成为新的直径。

#### 严谨一点？

设直径两端点分别为 $a, b$ ，从 $s$ 开始搜寻，找到点 $d$ 。需要分类讨论。

- 若 $s$ 是 $(a, b)$ 上一点：那么 $(a, d) = (a, s) + (s, d) \ge (a, s) + (s, b) = (a, b)$ ，与 $(a, b)$ 是树的直径矛盾。

- 若 $s$ 不在 $(a, b)$ 上：
  
  - 若 $(s, d)$ 与 $(a, b)$ 相交于 $m$ ，类比上一点有 $(a, d) \ge (a, b)$ ，矛盾。
  
  - 若 $(s, d)$ 与 $(a, b)$ 无交点，我们希望：能够用一条路径把 $(s, d)$ 和 $(a, b)$ “接上“，转成上一种情况。
    
    于是设 $m$ 是 $(s, d)$ 与 $(s, b)$ 的岔路口，$n$ 是 $(b, a)$ 与 $(b, m)$ 的岔路口，应有：
    $(s, d) \ge (s, b)$
    两边同减 $(s, m)$ 得 $(m, d) \ge (m, b)$ ，
    而 $(n, d) = (m, d) + (m, n)$ ， $(n, b) = (m, b) - (m, n)$ ，
    说明 $(n, d) > (n, b)$，可以看作 $n$ 开始搜到 $d$ ，这就又回到了第一种情况。

于是定理得证。

:::note[为什么求法1不适用负权边]

"而 $(n, d) = (m, d) + (m, n)$ ， $(n, b) = (m, b) - (m, n)$ "  
不再能推出  
" $(n, d) > (n, b)$ "  
因为不能保证 $(m, n) \ge 0$ 。

:::

### 代码实现

```cpp
#include <iostream>
#include <vector>

using namespace std;

vector<int> graph[100005];

void add(int u, int v) {
    graph[u].push_back(v);
    graph[v].push_back(u);
}

int depth[100005];
int dfs(int p, int fa) {
    int res = p;

    for (int ch : graph[p]) {
        if (ch == fa) continue;
        depth[ch] = depth[p] + 1;

        int pt = dfs(ch, p);
        if (depth[pt] > depth[res]) res = pt;
    }

    return res;
}

int main(){
    int n;
    cin >> n;

    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        add(u, v);
    }

    int s = dfs(1, 0);
    depth[s] = 0;
    int d = dfs(s, 0);
    cout << depth[d];
}
```

## 04 求法2 - 树形DP

这种做法时间复杂度 $O(n)$ ，且**可以解决负权边的问题**。

我们先任取一点为根，记为 $1$ 号节点。

既然是树的直径，这个直径对于某棵子树来讲，一定是叶子节点 --> 根节点 --> 叶子节点的一条路径。这条路径可以由“到根最长” + “到根次长”来更新。

这样来看，维护好“到根最长”数组 $d_1$ 和“到根次长”数组 $d_2$ ，答案近在迟尺。

怎么维护？优先更新 $d_1$ ，剩下的给 $d_2$ 。

> [!NOTE]
> 
> 显然， $d_1[i]$ 和 $d_2[i]$ 不能有重边。

### 代码实现

```cpp
#include <iostream>
#include <vector>

using namespace std;

vector<int> graph[100005];

void add(int u, int v) {
    graph[u].push_back(v);
    graph[v].push_back(u);
}

int res = 0;
int d1[100005];
int d2[100005];
void dfs(int p, int fa) {
    for (int ch : graph[p]) {
        if (ch == fa) continue;

        dfs(ch, p);

        int d_new = d1[ch] + 1;
        if (d_new > d1[p]) {
            d2[p] = d1[p];
            d1[p] = d_new;
        } else {
            d2[p] = max(d_new, d2[p]);
        }
    }

    res = max(d1[p] + d2[p], res);
}

int main(){
    int n;
    cin >> n;

    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        add(u, v);
    }

    dfs(1, 0);

    cout << res;
}
```

事实上，还有另一种开一个数组的求法，见[OI Wiki](https://oiwiki.com/graph/tree-diameter/#%E8%BF%87%E7%A8%8B-2)。

# 树的重心

:::important

WIP  
施工中

:::

## 01 定义

**树的重心**：树上一点，使以它为根时，其最大子树最小。

可以理解为：把子树分割得最平均的位置。所谓平均，就是没有哪个子树太大（对应“最大子树最小”）。

## 02 性质

## 03 求法

# 树的中心

## 01 定义

**树的中心**：树上一点，使到任意一点距离的最大值最小。

可以理解为：到各点距离最平均的点。

## 02 性质

## 03 求法

# 小结
