---
title: 势能线段树
published: 2026-10-05
description: '神秘势能分析，使我的大脑旋转'
image: 'koishi-komeiji-milktea.jpg'
tags: ['OI', '数据结构', '线段树']
category: 'OI'
draft: false 
lang: ''
---

## 01 引入

来看这样一个问题： $1 \le n, m \le 10^5, 1 \le a_i \le 10^{12}$ ，要求维护区间开方下取整操作、区间和查询。

[[洛谷P4145]上帝造题的七分钟 2 / 花神游历各国](https://www.luogu.com.cn/problem/P4145)

~~是不是觉得花神很可爱？~~ 乍一看，这道题真是无从下手。我们实在是想不到，什么样的数据结构能够维护“开方”这种奇葩操作。普通的线段树期望能快速对一个区间进行整体操作，但是显然，知道一个区间的和并不能推导出开方后的和。走投无路的时候，我们可以选择*从特例入手*，尝试先找到一些什么，不管有什么用。

我们考查一些特例：如果一段数字全为 $1$ ，那么无论如何开方，这段数字还是 $1$ ，总和不会改变。扩展出去，如果一段数字有大于 $1$ 的数字，开方之后，这个数字一定会变小。*如果一段数字不断开方变小，它最后会变成多少？* 我们不难发现，所有数字的归宿，竟然都是最好处理的 $1$ ！而且我们知道，开方让数字下降的程度很“狠”。也就是说，如果我们暴力递归线段树上的节点，那么在经过很少次暴力之后，它马上就会变成 $1$ ；只要我们记录哪些区间变成了全 $1$ ，对这样的区间直出答案，就可以高效解决问题。

为了刻画这种性质，下面引入“势能”的概念。

## 02 势能分析

我们不妨把一个数字不断开根号变成 $1$ 所需要的次数认为是这个数字的势能。定义如下函数：

$$
f(x) = \left\{ \begin{array}{cl}
0 & : \ x = 1 \\
1+f(\left\lfloor \sqrt{x} \right\rfloor) & : \ x > 1
\end{array} \right.
$$

我们把一个区间 $[l,r]$ 的势能定义为其中所有数字的势能总和：

$$
\Phi = \sum^{r}_{i=l}{f(a_i)}
$$

显然，随着操作的进行，势能单调不增。**我们只要找出：每一次势能减小时，暴力往下递归修改的时间复杂度总和**，就能证明这种做法的正确性。

### 到底要多少次开方，才能降为1？

根据定义，一次“有效开方”（也就是对某一个 $a_i>1$ 执行了一次开方操作），势能 $\Phi$ 恰好减小$1$。每次开方使得一个数位数大致减半，我们有：

$$
f(x) = O(\log \log x)
$$

因此，初始的总势能为

$$
\Phi_0 = \sum f(x) = O(n \log \log V)
$$

由于有效开方要让 $\Phi$ 严格减，所以有效开方次数存在上界：$O(n \log \log V)$。

### 单次操作代价

假设一次开方操作 $k$ 个叶子节点发生了有效开方。

在这个区间内：

- $k$ 个节点需要暴力递归到叶子，路径长度为 $O(\log n)$ ，总复杂度 $O((k+1) \log n)$

- $k$ 个节点把大区间分割成了至多 $O(k+1)$ 个全为 $1$ 的小区间，每一个小区间在线段树上被分解成 $O(\log n)$ 个线段树节点，总复杂度 $O((k+1) \log n)$

因此单次区间开方，总复杂度上界 $O((k+1) \log n)$ 。（当 $k=0$ 时，复杂度变为普通线段树的 $O (\log n)$）

### 加起来？

$$
\sum k = O(n \log \log V)
$$

算上查询的复杂度和后续数组变为全 $1$ 时的修改，总代价：

$$
O(n \log n \log \log V + m \log n)
$$

这足以通过 $10^5$ 的数据。

## 03 代码

势能线段树的代码和普通线段树区别很小，只要额外维护区间最大值来标记哪些区间变为了全 $1$ 即可。

:::tip[小技巧（快速开平方）]
浮点数开方存在误差，但误差很小，可以在后面“修正”一下，就得到了高效的开方计算函数。
:::

```cpp
#include <bits/stdc++.h>

using namespace std;
typedef long long ll;

ll root(ll x) {
    ll r = sqrt(double(x));
    while (r * r > x) r--;
    while ((r + 1) * (r + 1) <= x) r++;
    return r;
}

ll a[100005];
struct Node {
    ll sum;
    ll mx;
    Node operator + (Node x) {
        return Node{
            sum + x.sum,
            max(mx, x.mx)
        };
    }
} seg[400005];

void build(int p, int l, int r) {
    if (l == r) {
        seg[p] = Node{a[l], a[l]};
        return;
    }

    int mid = (l + r) >> 1;

    build(2 * p, l, mid);
    build(2 * p + 1, mid + 1, r);

    seg[p] = seg[2 * p] + seg[2 * p + 1];
}

void upd(int p, int x, int y, int l, int r) {
    if (y < l || r < x) {
        return;
    } else if (x <= l && r <= y) {
        if (seg[p].mx == 1) return;
        if (l == r) {
            seg[p].sum = root(seg[p].sum);
            seg[p].mx = seg[p].sum;
            return;
        }

        int mid = (l + r) >> 1;

        upd(2 * p, x, y, l, mid);
        upd(2 * p + 1, x, y, mid + 1, r);

        seg[p] = seg[2 * p] + seg[2 * p + 1];
    } else {
        int mid = (l + r) >> 1;

        upd(2 * p, x, y, l, mid);
        upd(2 * p + 1, x, y, mid + 1, r);

        seg[p] = seg[2 * p] + seg[2 * p + 1];
    }
}

ll query(int p, int x, int y, int l, int r) {
    if (y < l || r < x) {
        return 0;
    } else if (x <= l && r <= y) {
        return seg[p].sum;
    } else {
        int mid = (l + r) >> 1;
        return query(2 * p, x, y, l, mid) + query(2 * p + 1, x, y, mid + 1, r);
    }
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);

    int n;
    cin >> n;

    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }

    build(1, 1, n);

    int m;
    cin >> m;

    for (int i = 0; i < m; i++) {
        int k, l, r;
        cin >> k >> l >> r;

        if (l > r) swap(l, r);

        if (k == 0) {
            upd(1, l, r, 1, n);
        } else if (k == 1) {
            cout << query(1, l, r, 1, n) << '\n';
        }
    }
}
```

## 04 拓展一点？

势能的思想不只有开方操作。像区间 $\min$ 、区间 $\gcd$ 、区间按位与之类单调的操作，都能应用势能线段树。

再看下面这一道题：

> 给定一颗树，支持如下操作：
> 
> - $1 \space u \space v \space d$ 将路径 $(u,v)$ 上的全部数字加上 $d$ 。（$d > 0$）
> 
> - $2 \space u \space v$ 查询路径 $(u,v)$ 上的全部数字的**绝对值**之和。
>
> （$0 \le |a_i|, d \le 10^8$）

“绝对值”可以看成正数和减去负数和，但加法操作会让负数变成正数，看起来不好处理。但，当我们有了“势能”的武器，相信你一定能看出来：我们**动用敏锐的注意力大法**，发现，每一个负数，**最多只会变成正数一次！** 所以，我们只要在“负数变正数”的临界点，线段树上暴力向下遍历一下，套上树链剖分，就能轻轻松松达到 $O(m \log ^2 n + n \log n)$ 的复杂度。

~~我也不知道为什么出题人要硬把区间问题放到树上，除了多套一层剖分以外有什么意义吗~~

有兴趣的读者可以自行尝试实现。

这里也放出代码：

```cpp collapse={1-50,51-180,181-244} collapseStyle=collapsible-start
#include <bits/stdc++.h>

using namespace std;
typedef long long ll;

int n, m;
const ll INF = 2.5e13;
vector<int> t[100005];

void add(int u, int v) {
    t[u].push_back(v);
    t[v].push_back(u);
}

ll a[100005];

int hd[100005];
int dfn[100005];
int rnk[100005];
int hv[100005];
int sz[100005];
int fa[100005];
int dep[100005];
int cnt = 0;
void dfs1(int v, int f) {
    sz[v] = 1;
    fa[v] = f;
    dep[v] = dep[f] + 1;
    for (int c : t[v]) {
        if (c == f) continue;
        dfs1(c, v);
        sz[v] += sz[c];
        if (sz[hv[v]] < sz[c]) hv[v] = c;
    }
}

void dfs2(int v, int h) {
    rnk[++cnt] = v;
    dfn[v] = cnt;
    hd[v] = h;

    if (hv[v]) dfs2(hv[v], h);

    for (int c : t[v]) {
        if (dfn[c]) continue;
        if (c == hv[v]) continue;
        dfs2(c, c);
    }
}

namespace Seg {

ll lazy[400005];

struct Node {
    ll sumn;
    ll sump;
    int cntn;
    int cntp;
    ll maxn;
    Node operator + (Node x) {
        return Node{
            sumn + x.sumn,
            sump + x.sump,
            cntn + x.cntn,
            cntp + x.cntp,
            max(maxn, x.maxn)
        };
    }
} seg[400005];

void push_down(int p, int l, int r) {
    if (l == r) return;

    seg[2 * p] = seg[2 * p] + Node{
        seg[2 * p].cntn * lazy[p],
        seg[2 * p].cntp * lazy[p],
        0,
        0,
        -INF
    };
    seg[2 * p + 1] = seg[2 * p + 1] + Node{
        seg[2 * p + 1].cntn * lazy[p],
        seg[2 * p + 1].cntp * lazy[p],
        0,
        0,
        -INF
    };

    seg[2 * p].maxn += lazy[p];
    seg[2 * p + 1].maxn += lazy[p];

    lazy[2 * p] += lazy[p];
    lazy[2 * p + 1] += lazy[p];

    lazy[p] = 0;
}

void build(int p, int l, int r) {
    if (l == r) {
        seg[p].sumn = min(a[rnk[l]], 0ll);
        seg[p].sump = max(a[rnk[l]], 0ll);
        seg[p].cntn = a[rnk[l]] < 0 ? 1 : 0;
        seg[p].cntp = a[rnk[l]] >= 0 ? 1 : 0;
        seg[p].maxn = -INF;
        if (seg[p].cntn) seg[p].maxn = a[rnk[l]];
        return;
    }

    int mid = (l + r) >> 1;

    build(2 * p, l, mid);
    build(2 * p + 1, mid + 1, r);

    seg[p] = seg[2 * p] + seg[2 * p + 1];
}

void upd(int p, ll d, int x, int y, int l, int r) {
    if (r < x || y < l) {
        return;
    } else if (x <= l && r <= y) {
        if (seg[p].maxn < -d) {
            lazy[p] += d;
            seg[p] = seg[p] + Node{
                seg[p].cntn * d,
                seg[p].cntp * d,
                0,
                0,
                -INF
            };
            seg[p].maxn += d;
        } else if (seg[p].maxn >= -d) {
            if (l == r) {
                seg[p].maxn = -INF;
                seg[p] = Node{
                    0,
                    seg[p].sumn + d,
                    0,
                    1,
                    -INF
                };
                return;
            }

            push_down(p, l, r);

            int mid = (l + r) >> 1;

            upd(2 * p, d, x, y, l, mid);
            upd(2 * p + 1, d, x, y, mid + 1, r);

            seg[p] = seg[2 * p] + seg[2 * p + 1];
        }
    } else {
        push_down(p, l, r);

        int mid = (l + r) >> 1;

        upd(2 * p, d, x, y, l, mid);
        upd(2 * p + 1, d, x, y, mid + 1, r);

        seg[p] = seg[2 * p] + seg[2 * p + 1];
    }
}

Node query(int p, int x, int y, int l, int r) {
    if (r < x || y < l) {
        return Node{0, 0, 0, 0, -INF};
    } else if (x <= l && r <= y) {
        return seg[p];
    } else {
        int mid = (l + r) >> 1;

        push_down(p, l, r);
        return query(2 * p, x, y, l, mid) + query(2 * p + 1, x, y, mid + 1, r);
    }
}

}

ll query(int u, int v) {
    ll res = 0;
    while (hd[u] != hd[v]) {
        if (dep[hd[u]] > dep[hd[v]]) swap(u, v);
        Seg::Node nd = Seg::query(1, dfn[hd[v]], dfn[v], 1, n);
        res += nd.sump - nd.sumn;
        v = fa[hd[v]];
    }

    if (dep[u] > dep[v]) swap(u, v);
    Seg::Node nd = Seg::query(1, dfn[u], dfn[v], 1, n);
    res += nd.sump - nd.sumn;

    return res;
}

void upd(int u, int v, ll d) {
    while (hd[u] != hd[v]) {
        if (dep[hd[u]] > dep[hd[v]]) swap(u, v);
        Seg::upd(1, d, dfn[hd[v]], dfn[v], 1, n);
        v = fa[hd[v]];
    }

    if (dep[u] > dep[v]) swap(u, v);
    Seg::upd(1, d, dfn[u], dfn[v], 1, n);
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);

    cin >> n >> m;

    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }

    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        add(u, v);
    }

    dep[1] = 1;
    dfs1(1, 0);
    dfs2(1, 1);

    Seg::build(1, 1, n);

    for (int i = 0; i < m; i++) {
        int op;
        cin >> op;

        if (op == 1) {
            int u, v, d;
            cin >> u >> v >> d;
            upd(u, v, d);
        } else if (op == 2) {
            int u, v;
            cin >> u >> v;
            cout << query(u, v) << '\n';
        }
    }
}
```

## 05 总结

“势能”在OI中是一个很重要，也很有趣的概念，许多看似暴力的解法经过了势能分析，竟然就变成了正解，这也是为什么我认为势能线段树是一种比较优雅的数据结构之一。因此，仔细观察题目当中隐藏的范围，往往是OI中十分重要的思想方法。

*封面图片作者：[Nanaku_](https://space.bilibili.com/121042172)*
