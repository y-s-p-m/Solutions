# Description

[link](https://www.luogu.com.cn/problem/P3324)

# Solution

首先我们发现这个时间不太好处理，又加上还有[这个题](https://www.cnblogs.com/yspm/p/12634320.html)

所以把题目转成二分 $+$ 判定存在性问题

这样就好处理了

关于建图：

（其实就是个二分图多重匹配拿网络流做的典范）

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
	const int M=2e5+10,N=2020;
	struct node{
		int to,nxt,lim;
	}e[M<<1];
	int head[N],cnt=1,S,T,dep[N];
	inline void add1(int u,int v,int w)
	{
		e[++cnt].nxt=head[u]; e[cnt].to=v; e[cnt].lim=w;
		return head[u]=cnt,void();
	}
	inline void add(int u,int v,int w){return add1(u,v,w),add1(v,u,0);}
	inline bool bfs()
	{
		queue<int> q; q.push(S); 
		memset(dep,0,sizeof(dep)); dep[S]=1;
		while(q.size())
		{
			int fr=q.front(); q.pop();
			for(int i=head[fr];i;i=e[i].nxt)
			{
				int t=e[i].to;
				if(e[i].lim&&!dep[t]) dep[t]=dep[fr]+1,q.push(t);
			}
		} return dep[T];
	}
	inline int dfs(int now,int in)
	{
		if(now==T) return in; int out=0; 
		for(int i=head[now];i;i=e[i].nxt)
		{
			int t=e[i].to; 
			if(e[i].lim&&dep[now]+1==dep[t])
			{
				int res=dfs(t,min(e[i].lim,in));
				in-=res; out+=res; e[i].lim-=res; e[i^1].lim+=res;
			}
		}if(!out) dep[now]=0;
		return out;
	}
	int fl[N][N],n,m,a[N],b[N],sum;
	inline int dinic()
	{
		int ans=0; while(bfs()) ans+=dfs(S,1e18+10);
		return ans;
	}
	inline void build(int x)
	{
		memset(e,0,sizeof(e)); memset(head,0,sizeof(head)); cnt=1;
		S=n+m+1,T=n+m+2;
		for(int i=1;i<=m;++i) add(S,i,b[i]*x);
		for(int i=1;i<=n;++i) add(i+m,T,a[i]);
		for(int i=1;i<=m;++i) 
		{
			for(int j=1;j<=n;++j)
			{
				if(fl[i][j]) add(i,j+m,1e18);
			}
		} return ;
	}
	signed main()
	{
		n=read(); m=read(); 
		for(int i=1;i<=n;++i) a[i]=read()*10000,sum+=a[i];
		for(int i=1;i<=m;++i) b[i]=read();
		for(int i=1;i<=m;++i) for(int j=1;j<=n;++j) fl[i][j]=read();
		int l=1,r=1e13,ans;
		while(l<=r)
		{
			int mid=(l+r)>>1;
			build(mid); 
			if(dinic()>=sum) r=mid-1,ans=mid;
			else l=mid+1;
 		} printf("%.5lf\n",1.0*ans/10000);
		return 0;
	}
}
signed main(){return yspm::main();}
```