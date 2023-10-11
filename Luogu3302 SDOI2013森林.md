重构的艺术

# Description

[link](https://www.luogu.com.cn/problem/P3302)

有个森林，我们需要支持两种操作：

$1.$ 加边

$2.$ 查询树内两点路径中的第 $k$ 大点权

强制在线

# Solution

没有强制在线好像会简单一点？不是很清楚（会在线做法了就……）

树上 $k$ 大，很明显有主席树的那个模型

然后问题变成了我们怎么维护 $LCA$

这里有个东西叫“启发式合并”

主要思想：小的往大的上面并

我们并不需要合并权值线段树（博主亲写了一部分发现就不会写了）

只是在连边的时候我们比较两个树的大小然后把小的往大的上并

我们求 $LCA$ 的倍增数组可以在我们 $dfs$ 并树的时候一起更新

合并的总复杂度是$n \log n?$


# Code

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
namespace yspm {
inline int read() {
    int res = 0, f = 1;
    char k;
    while (!isdigit(k = getchar()))
        if (k == '-')
            f = -1;
    while (isdigit(k)) res = res * 10 + k - '0', k = getchar();
    return res * f;
}
const int N =2e5 + 10;
struct node {
    int ls, rs, sum;
#define ls(p) t[p].ls
#define rs(p) t[p].rs
#define cnt(p) t[p].cnt
#define sum(p) t[p].sum
} t[N *100];
int rt[N], sz[N], tot, fa[N][20], n, m, T, b[N], num, dep[N], lans, a[N], f[N], vis[N];
vector<int> g[N];
inline void push_up(int p) { return sum(p) = sum(ls(p)) + sum(rs(p)), void(); }
inline int find(int x) { return f[x] == x ? x : f[x] = find(f[x]); }
inline void update(int &p, int pre, int l, int r, int x) {
	p = ++tot;
    t[p] = t[pre];
    if (l == r)
        return sum(p)++, void();
    int mid = (l + r) >> 1;
    if (x <= mid)
        update(ls(p), ls(pre), l, mid, x);
    else
        update(rs(p), rs(pre), mid + 1, r, x);
    return push_up(p);
}
inline int lca(int x, int y) {
    if (dep[x] < dep[y])
        swap(x, y);
    for(int i=19;i>=0;--i) if(dep[fa[x][i]]>=dep[y]) x=fa[x][i]; 
    if (x == y)
        return x;
    for (int i =19; i >= 0; --i)
        if (fa[x][i] != fa[y][i])
            x = fa[x][i], y = fa[y][i];
    return fa[x][0];
}
inline int query(int a, int b, int c, int d, int l, int r, int k) {
    if (l == r)
        return l;
    int mid = (l + r) >> 1, tmp = sum(ls(a)) + sum(ls(b)) - sum(ls(c)) - sum(ls(d));
    if (k <= tmp)
        return query(ls(a), ls(b), ls(c), ls(d), l, mid, k);
    else
        return query(rs(a), rs(b), rs(c), rs(d), mid + 1, r, k - tmp);
}
inline void dfs(int x, int fat, int r) {
    f[x] = fat;
    vis[x] = 1;
    dep[x] = dep[fat] + 1;
    fa[x][0] = fat;
    sz[r]++;
    for (int i = 1; i<=19; ++i) fa[x][i] = fa[fa[x][i - 1]][i - 1];
    update(rt[x],rt[fat],1,num,a[x]);
    int siz = g[x].size();
    for (int i = 0; i < siz; ++i)
        if (g[x][i] != fat)
            dfs(g[x][i], x, r);
    return;
}
char s[4];
int u, v;
signed main() {
    read(); n=read(); m=read(); T=read();
    for (int i = 1; i <= n; ++i) a[i] = read(), b[++num] = a[i], f[i] = i;
    sort(b + 1, b + num + 1);
    num = unique(b + 1, b + num + 1) - b - 1;
    for (int i = 1; i <= n; ++i) a[i] = lower_bound(b + 1, b + num + 1, a[i]) - b;
    while (m--) u = read(), v = read(), g[u].push_back(v), g[v].push_back(u);
    for (int i = 1; i <= n; ++i) {
        if (vis[i])
            continue;
        dfs(i, 0, i);
        f[i] = i;
    }
    while (T--) {
        scanf("%s", s);
        if (s[0] == 'Q') {
            int x = read() ^ lans, y = read() ^ lans, k = read() ^ lans;
            int l = lca(x, y);
            lans = query(rt[x], rt[y], rt[l], rt[fa[l][0]], 1, num, k);
            printf("%lld\n", b[lans]); lans=b[lans];
        } else {
            int x = read() ^ lans, y = read() ^ lans;
            g[x].push_back(y);
            g[y].push_back(x);
            int ax = find(x), ay = find(y);
            if (sz[ax] < sz[ay])
                swap(ax, ay), swap(x, y);
			dfs(y, x, ax);
            
        }
    }
    return 0;
}
}  // namespace yspm
signed main() { return yspm::main(); }
```

# Review

在发现有连边操作或者合并信息的时候可以考虑启发式合并

还有可以部分暴力重构