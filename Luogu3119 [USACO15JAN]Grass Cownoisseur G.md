# Description

[link](https://www.luogu.com.cn/problem/P3119)

给定一个有向图，从 $1$ 号出发，最后要求回到 $1$ 号

其中允许经过重复的点，同时允许走一次反向边

求最多经过多少个点

$n,m \le 10^5$

# Solution

这种题上了缩点没的说（不会缩点？左转 $Luogu$ 模板区）

（下文的图都是指缩完点的图）

然后我们发现在这个 $DAG$ 上面跑最长路 $dp$ 并不可行，反悔并不好处理

分层呀！

图上的所有边我们可以建成这样：

### 第一层的边复制到第二层，然后在第一层的终点向第二层的起点建边

再跑最长路

（本题的唯一思考点……）

正确性？如果重复经过点呢？

请思考我们这个最长路会在哪些点上

如果往出边走，是不会回到 $1$ 的（要不就又是环了）

只会是往前走一次，再回到 $1$ 点

所以正确性是有保证的

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
	const int N=2e5+10;
	int st[N],top;
	vector<int> g[N],vec[N];
	int dfn[N],low[N],tim,vis[N],val[N],tot,id[N],res[N],n,m;
	inline void tarjan(int x)
	{
		dfn[x]=low[x]=++tim; vis[x]=1; st[++top]=x;
		int siz=g[x].size(); 
		for(int i=0;i<siz;++i)
		{
			int t=g[x][i]; 
			if(!dfn[t]) tarjan(t),low[x]=min(low[t],low[x]);
			else if(vis[t]) low[x]=min(low[x],dfn[t]);
		}
		if(dfn[x]==low[x])
		{
			++tot; 
			do{
				int t=st[top]; val[tot]++;
				id[t]=tot;
				vis[t]=0; top--;
			}while(st[top+1]!=x);
		}
		return ;
	}
	inline void spfa(int s)
	{
		memset(vis,0,sizeof(vis));
		queue<int>q; q.push(s); vis[s]=1;
		while(!q.empty())
		{
			int fr=q.front(); vis[fr]=0; q.pop();
			int siz=vec[fr].size();
			for(int i=0;i<siz;++i)
			{
				int t=vec[fr][i]; 
				if(res[t]<res[fr]+val[fr]) 
				{
					res[t]=res[fr]+val[fr];
					if(!vis[t]) vis[t]=1,q.push(t);
				}
			}
		} return ;
	}
	signed main()
	{
		n=read(); m=read();
		for(int i=1;i<=m;++i)
		{
			int x=read(),y=read();
			g[x].push_back(y);
		}
		for(int i=1;i<=n;++i) if(!dfn[i]) tarjan(i);
		for(int i=1;i<=tot;++i) val[i+tot]=val[i];
		for(int i=1;i<=n;++i)
		{
			int sz=g[i].size();
			for(int j=0;j<sz;++j)
			{
				if(id[g[i][j]]!=id[i]) 
				{
					vec[id[i]].push_back(id[g[i][j]]);
					vec[id[i]+tot].push_back(id[g[i][j]]+tot);
					vec[id[g[i][j]]].push_back(id[i]+tot);
				}
			}
		}
		spfa(id[1]);
		printf("%lld\n",res[id[1]+tot]);
		return 0;
	}
}
signed main(){return yspm::main();}
```
