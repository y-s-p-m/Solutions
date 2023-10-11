由于我的代码过不了洛谷的数据，本题头只能叫 $BZOJ1097$

# Description

[link](https://www.luogu.com.cn/problem/P3451)

直接去看题面吧，真的清楚诶

# Solution

声明：本博客只能过掉 $BZOJ$ 的数据，并不是原题的解法

观察数据范围可以发现这是一道状压题

预处理 $k$ 个点到所有点的最短路

定义：$f_{i,s}$ 为关键点的经过状态为 $s$ 的时从 $1$ 到 $i$ 的最短路

转移的时候我们只需要在 $k$ 个点中进行转移即可

关于是否满足当前点的所有前缀点是不是被经过了，可以用这样的方法进行判断：

```cpp
if((s&st[t])!=st[t]) continue;
```

挺好理解的对吧

然后我们对于每个状态中为 $1$ 的点看是不是有哪个新的点可以被更新即可

关于如何把空间压到 $64mb$，留坑

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
	const int N=2e4+10,inf=0x3f3f3f3f3f3f3f3f;
	int head[N],cnt;
	struct node{
		int to,dis,nxt;
	}e[400010];
	inline void add(int u,int v,int w)
	{
		e[++cnt].dis=w; e[cnt].nxt=head[u]; e[cnt].to=v;
		return head[u]=cnt,void();
	}
	int n,m,k,dis[22][N];
	bool fl[N];
	inline void dij(int s)
	{
		memset(fl,0,sizeof(fl)); dis[s][s]=0;
		priority_queue<pair<int,int> > q;
		q.push(make_pair(0,s));
		while(!q.empty())
		{
			int fr=q.top().second; q.pop();
			if(fl[fr]) continue; fl[fr]=1;
			for(int i=head[fr];i;i=e[i].nxt)
			{
				int t=e[i].to,dist=e[i].dis+dis[s][fr];
				if(dist<dis[s][t])
				{
					dis[s][t]=dist;
					q.push(make_pair(-dis[s][t],t));
				}
			}
		}
		return ;
	}
	int r,s,f[21][1<<20],st[30];
	signed main()
	{
		n=read(); m=read(); k=read();
		for(int i=1,x,y,z;i<=m;++i)
		{
			x=read(),y=read(),z=read();
			add(x,y,z); add(y,x,z);
		} 
		memset(dis,0x3f,sizeof(dis));
		if(!k) return dij(1),printf("%lld\n",dis[1][n]),0;
		for(int i=1;i<=k+1;++i) dij(i);
		int g=read();
		for(int i=1;i<=g;++i) r=read()-2,s=read()-2,st[s]|=1<<r;
		for(int i=1;i<(1<<k);++i) for(int j=0;j<k;++j) f[j][i]=inf;
		for(int i=0;i<k;++i) if(!st[i]) f[i][1<<i]=dis[1][i+2];
		for(int s=1;s<(1<<k);++s)
		{
			for(int i=0;i<k;++i)
			{
				if(!((1<<i)&s)) continue;
				if(f[i][s]==inf) continue;
				for(int t=0;t<k;++t) 
				{
					if((1<<t)&s) continue;
					if((s&st[t])!=st[t]) continue;
					f[t][s|(1<<t)]=min(f[t][s|(1<<t)],f[i][s]+dis[i+2][t+2]);
				}
			}
		} 
		int ans=inf;
		for(int i=0;i<k;++i) ans=min(ans,f[i][(1<<k)-1]+dis[i+2][n]);
		printf("%lld\n",ans);
		return 0;
	}
}
signed main(){return yspm::main();}

```

# Review

对于状压和最短路进行结合的时候可以考虑预处理必经点之类的操作

不需要像分层图那样考虑

再一个就是补一下如何压空间的事