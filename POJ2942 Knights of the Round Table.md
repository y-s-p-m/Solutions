真的是送自闭的一个题，大早上就到大下午了

# Description

[link](http://poj.org/problem?id=2942)

召开圆桌会议要有奇数个人，现在有 $n$ 个人，每个人都有给定的仇恨的对象

这个是一个无向图

相互仇恨的人不能邻座，求有多少个人永远不能参加会议

# Solution

首先建补图，我们发现题目转化成了哪些骑士不包含于任何一个奇环中

引理：若某个点双连通分量中有一个奇环，则在该点双联通分量中，奇环外的点一定也在某一个奇环内。

其实证明很简单，就是这个点连向奇环上的两个点

优弧和劣弧上必然是一个有奇数个点，一个有偶数个点

然后看这个点掐住的的胳膊上有几个点，奇数对偶数，偶数对奇数

就证完了

然后我们 $tarjan$ 求点双，每次求出来进行二分图的奇环染色，最后这题细节真的真的多

到最后我也不知道我一开始的清空方法为啥是错的

# Code

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<cmath>
#include<vector>
#include<queue>
using namespace std;
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=2010;
	bool f[N][N],vis[N];
	int dfn[N],tim,low[N],ans,col[N],n,m,abl[N],in[N],st[N],top;
	inline bool make(int x)
	{
		queue<int> q; q.push(x); memset(col,0,sizeof(col)); col[x]=1;
		while(!q.empty())
		{
			int fr=q.front(); q.pop();
			for(int i=1;i<=n;++i)
			{
				if(i==fr||!in[i]||f[fr][i]) continue;
				else
				{
					if(!col[i]) col[i]=3-col[fr],q.push(i);
					else if(col[i]==col[fr]) return 1; 
				}
			}
		} return 0;
	}
	inline void tarjan(int x,int rt)
	{
		dfn[x]=low[x]=++tim; st[++top]=x; vis[x]=1;
		for(int i=1;i<=n;++i)
		{
			if(i==x||i==rt||f[x][i]) continue;
			if(!vis[i])
			{
				tarjan(i,x); low[x]=min(low[x],low[i]);
				if(low[i]>=dfn[x])
				{
					memset(in,0,sizeof(in));
					while(st[top]!=i) in[st[top]]=1,top--; 
					in[i]=1; top--; in[x]=1; 
					if(make(x)) for(int i=1;i<=n;++i) if(in[i]) abl[i]=1;
				} 
			}
			else low[x]=min(low[x],dfn[i]);
		}
		return ;
	}
	inline void prework()
	{
		memset(dfn,0,sizeof(dfn)); memset(low,0,sizeof(low));
		memset(st,0,sizeof(st)); tim=0; top=0;
		memset(abl,0,sizeof(abl)); memset(vis,0,sizeof(vis));
		memset(f,0,sizeof(f));
		return ;
	}
	inline void work()
	{
		for(int i=1;i<=m;++i){
			int u=read(),v=read();
			f[u][v]=f[v][u]=1;
		} 
		for(int i=1;i<=n;++i){
			if(!vis[i]) tarjan(i,i);
		}
		ans=0; 
		for(int i=1;i<=n;++i) ans+=!abl[i];
		cout<<ans<<endl;
		return ;
	}
	signed main()
	{
		n=read(),m=read();
		while(n+m) work(),n=read(),m=read();
		return 0;
	}
}
signed main(){return yspm::main();}

```

错误代码如上

正确代码如下：

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<cmath>
#include<vector>
#include<queue>
using namespace std;
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=2010;
	bool f[N][N],vis[N];
	int dfn[N],tim,low[N],ans,col[N],n,m,abl[N],in[N],st[N],top;
	inline bool make(int x)
	{
		queue<int> q; q.push(x); memset(col,0,sizeof(col)); col[x]=1;
		while(!q.empty())
		{
			int fr=q.front(); q.pop();
			for(int i=1;i<=n;++i)
			{
				if(i==fr||!in[i]||f[fr][i]) continue;
				else
				{
					if(!col[i]) col[i]=3-col[fr],q.push(i);
					else if(col[i]==col[fr]) return 1; 
				}
			}
		} return 0;
	}
	inline void tarjan(int x,int rt)
	{
		dfn[x]=low[x]=++tim; st[++top]=x; vis[x]=1;
		for(int i=1;i<=n;++i)
		{
			if(i==x||i==rt||f[x][i]) continue;
			if(!vis[i])
			{
				tarjan(i,x); low[x]=min(low[x],low[i]);
				if(low[i]>=dfn[x])
				{
					memset(in,0,sizeof(in));
					while(st[top]!=i) in[st[top]]=1,top--; 
					in[i]=1; top--; in[x]=1; 
					if(make(x)) for(int i=1;i<=n;++i) if(in[i]) abl[i]=1;
				} 
			}
			else low[x]=min(low[x],dfn[i]);
		}
		return ;
	}
	
	inline void work()
	{
		memset(f,0,sizeof(f));
		for(int i=1;i<=m;++i){
			int u=read(),v=read();
			f[u][v]=f[v][u]=1;
		} 
		memset(dfn,0,sizeof(dfn)); memset(low,0,sizeof(low));
		memset(st,0,sizeof(st)); tim=0; top=0;
		memset(abl,0,sizeof(abl)); memset(vis,0,sizeof(vis));
		for(int i=1;i<=n;++i){
			if(!vis[i]) tarjan(i,i);
		}
		ans=0; 
		for(int i=1;i<=n;++i) ans+=!abl[i];
		cout<<ans<<endl;
		return ;
	}
	signed main()
	{
		n=read(),m=read();
		while(n+m) work(),n=read(),m=read();
		return 0;
	}
}
signed main(){return yspm::main();}
```