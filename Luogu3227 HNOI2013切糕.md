## [My blog](https://www.cnblogs.com/yspm/)

# Description

[link](https://www.luogu.com.cn/problem/P3227)

给定一个 $n,m,h$ 的立方体，每个点有代价，现在在每个纵轴上面找一个点，使得所有的代价的和最小

限制：相邻的纵轴上找的点的位置距离差不能超过一个给定的值 $d$

# Solution

最小割，割掉的边是要切掉的点

从源点向第一层的点连 $inf$ 边

这里还有一个点化边，然后新建一层，接着从上面一层的每一个点向下面的点连上一条边权为点的代价的边

最后一层（$h+1$ 层）往汇点连一条边权为 $inf$ 的边

然后我们考虑怎么对付这个限制

其实这个限制在图上就变成了我们在考虑一个 $(x,y,k)$ 之后必须选 $(x+dx,y+dy,k\ \pm\  d)$

啊这里 $dx$ 和 $dy$ 表示四向相邻点

那么就是必经边呗，连上个 $inf$ 就成了

然后跑个最小割就好了

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
		int to,lim,nxt;
	}e[N<<1];
	int head[N],cnt=1,dep[N],S,T;
	queue<int> Q;
	inline void adde(int u,int v,int w)
	{
		e[++cnt].lim=w; e[cnt].nxt=head[u]; e[cnt].to=v;
		return head[u]=cnt,void();
	}
	inline void add(int u,int v,int w){return adde(u,v,w),adde(v,u,0);}
	inline bool bfs()
	{
		memset(dep,0,sizeof(dep)); dep[S]=1; 
		while(Q.size()) Q.pop(); Q.push(S);
		while(Q.size())
		{
			int fr=Q.front(); Q.pop();
			for(int i=head[fr];i;i=e[i].nxt)
			{
				int t=e[i].to; 
				if(e[i].lim&&!dep[t]) dep[t]=dep[fr]+1,Q.push(t);
			}
		} return dep[T];
	}
	inline int dfs(int now,int in)
	{
		int out=0; if(now==T) return in;
		for(int i=head[now];in&&i;i=e[i].nxt)
		{
			int t=e[i].to; 
			if(e[i].lim&&dep[now]==dep[t]-1) 
			{
				int res=dfs(t,min(e[i].lim,in));
				in-=res; out+=res; e[i].lim-=res; e[i^1].lim+=res;
			}
		} if(!out) dep[now]=0;
		return out;	
	}
	int v[110][110][100],p,q,r,d;
	inline int id(int x,int y,int z){return p*q*(z-1)+y+(x-1)*q;}
	int fx[4]={0,1,0,-1};
	int fy[4]={1,0,-1,0};
	inline bool in(int x,int y){return x>=1&&x<=p&&y>=1&&y<=q;}
	signed main()
	{
		p=read(),q=read(); r=read(); d=read();
		S=p*q*(r+1)+1,T=p*q*(r+1)+2; 
		for(int k=1;k<=r;++k)
		{
			 for(int i=1;i<=p;++i)
			{
				for(int j=1;j<=q;++j) v[i][j][k]=read();
			}
		}
		for(int i=1;i<=p;++i)
		{
			for(int j=1;j<=q;++j)
			{
				add(S,id(i,j,1),1e18+10);
				for(int k=1;k<=r;++k) add(id(i,j,k),id(i,j,k+1),v[i][j][k]);
				add(id(i,j,r+1),T,1e18+10); 
			}
		}
		for(int i=1;i<=p;++i)
		{
			for(int j=1;j<=q;++j)
			{
				for(int k=0;k<4;++k)
				{
					int x=i+fx[k],y=j+fy[k];
					if(!in(x,y)) continue;
					for(int l=d+1;l<=r+1;++l) add(id(i,j,l),id(x,y,l-d),1e18+10);
				}
			}
		}
		int ans=0;
		while(bfs()) ans+=dfs(S,1e18+10);
		printf("%lld\n",ans);
		return 0;
	}
}
signed main(){return yspm::main();}
```

# Review

限制可以转化成网络图上面的必经边，然后直接连 $inf$ 边就好了