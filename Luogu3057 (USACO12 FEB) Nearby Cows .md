第一次做容斥题，结果发现了自己不会容斥的本质

# Description

[link](https://www.luogu.com.cn/problem/P3047)

给你一棵 $n$个点的树，点带权，对于每个节点求出距离它不超过 $k$ 的所有节点权值和 $p_i$

$n \leq 10^5,k \le 20$

# Solution

一道统计答案的题，感觉有点像点分治（然而并不知道点分治能不能做）

定义$f_{i,j}$为距离点 $i$ 长度 $j$ 的点权和

首先可以一边树形 $dp$ 出每个点的子树的 $ans$ 值的对吧

然后我们看看怎么从下往上走一次

直接加上父亲节点的距离减一是不行的，因为如果父亲节点往这个子树走是会出现重复答案的

那我们就考虑在这里搞容斥（博主一开始想复杂了，然后发现其实不是很难）

就是在加上父亲节点的答案的时候先减去距离减 $2$ 就好了

形式化的说：

```cpp
for(int j=k;j>=2;--j) f[t][j]-=f[t][j-2];
for(int j=1;j<=k;++j) f[t][j]+=f[x][j-1];
```

$x$ 为父节点， $t$ 是子节点

这里是在搜父亲的时候进行的过程

# Code 

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=1e5+10;
	struct node{
		int to,nxt;
	}e[N<<1];
	int head[N],cnt;
	inline void add(int u,int v)
	{
		e[++cnt].nxt=head[u]; e[cnt].to=v;
		return head[u]=cnt,void();
	}
	int fa[N],f[N][30],n,k; 
	inline void dp(int x,int fat)
	{
		fa[x]=fat;
		for(int i=head[x];i;i=e[i].nxt)
		{
			int t=e[i].to; if(t==fat) continue;
			dp(t,x);
			for(int j=1;j<=k;++j) f[x][j]+=f[t][j-1];
		} return ;
	}
	inline void dfs(int x)
	{
		for(int i=head[x];i;i=e[i].nxt)
		{
			int t=e[i].to; if(t==fa[x]) continue;
			for(int j=k;j>=2;--j) f[t][j]-=f[t][j-2];
			for(int j=1;j<=k;++j) f[t][j]+=f[x][j-1];
			dfs(t);
		}return ;
	}
	signed main()
	{
		n=read(); k=read(); 
		for(int i=1,u,v;i<n;++i) u=read(),v=read(),add(u,v),add(v,u);
		for(int i=1;i<=n;++i) f[i][0]=read();
		dp(1,0); dfs(1); 
		for(int i=1;i<=n;++i) 
		{
			for(int j=1;j<=k;++j) f[i][j]+=f[i][j-1];
			printf("%lld\n",f[i][k]);
		}
		return 0;
	}
}
signed main(){return yspm::main();}
```