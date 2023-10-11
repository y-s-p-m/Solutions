# Description

[link](https://www.luogu.com.cn/problem/P5038)

# Solution

发现这个棋盘和操作构成了一个二分图……

我们把整个棋盘分为两个部分

然后每次操作会改变黑点白点上的两个值

设点的个数和权值和分别为 $num$ 和 $sum$

讨论一发：

$1.$ 个数相同

先判断 $sum$ 是不是一样

考虑到黑点白点个数有一样，如果有一个值 $x$ 是经过操作所有点都可以达到的

就是点的终值为 $x$

那么 $x+1$ 也可以被达到

那么如果 $x$ 不行，$x-1$ 也不行

那么我们求出一个 $x$ 或者证明不存在即可

这个过程因为数大了可能会存在（存在“解的单调性”），所以二分一下

然后类多重匹配用最大流做

$2.$ 个数不同

设终值为 $p$ 

有：

$$x\times num_1-sum_1=x\times num_2-sum_2$$

推一下：

$$x=\frac{sum_1-sum_2}{num_1-num_2}$$

但是这个 $x$ 不一定可行，建个图判断一下吧

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
	const int N=2010,M=2e5+10;
	struct node{
		int to,lim,nxt;
	}e[M<<1];
	inline int max(int x,int y){return x<y?y:x;}
	inline int min(int x,int y){return x<y?x:y;}
	int head[N],cnt=1,dep[N],S,T,n,m,maxx,a[N][N];
	int fx[4]={1,0,-1,0},fy[4]={0,1,0,-1};
	inline void adde(int u,int v,int w)
	{
		e[++cnt].lim=w; e[cnt].nxt=head[u]; e[cnt].to=v;
		return head[u]=cnt,void();
	} 
	queue<int>q; 
	inline void add(int u,int v,int w){return adde(u,v,w),adde(v,u,0);}
	inline bool bfs()
	{
		memset(dep,0,sizeof(dep)); dep[S]=1; 
		while(q.size()) q.pop(); q.push(S);
		while(!q.empty())
		{
			int fr=q.front(); q.pop(); 
			for(int i=head[fr];i;i=e[i].nxt)
			{
				int t=e[i].to;
				if(!dep[t]&&e[i].lim) dep[t]=dep[fr]+1,q.push(t); 
			}
		}
		return dep[T];
	}
	inline int dfs(int now,int in)
	{
		if(now==T) return in; int out=0;
		for(int i=head[now];i&&in;i=e[i].nxt)
		{
			int t=e[i].to;
			if(e[i].lim&&dep[t]==dep[now]+1)
			{
				int res=dfs(t,min(in,e[i].lim));
				in-=res; out+=res; e[i].lim-=res;
				e[i^1].lim+=res;
			}
		}if(!out) dep[now]=0;
		return out;
	 } 
	inline int id(int x,int y){return (x-1)*m+y;}
	inline bool in(int x,int y){return x>=1&&x<=n&&y>=1&&y<=m;}
	inline bool check(int val)
	{
		S=n*m+1; T=n*m+2;
		int sum=0; 
		memset(head,0,sizeof(head)); cnt=1;
		for(int i=1;i<=n;++i)
		{
			for(int j=1;j<=m;++j)
			{
				if((i+j)%2==0)
				{
					add(S,id(i,j),val-a[i][j]); sum+=val-a[i][j];
					for(int k=0;k<4;++k)
					{
						int tx=fx[k]+i,ty=fy[k]+j;
						if(in(tx,ty)) add(id(i,j),id(tx,ty),1e18+10);
					}
				}
				else add(id(i,j),T,val-a[i][j]);
			}
		}
		int res=0;
		while(bfs()) res+=dfs(S,1e18+10);
		return res==sum;
	}
	inline void work()
	{
		int num[2]={0},sum[2]={0};
		n=read(); m=read(); maxx=0;
		for(int i=1;i<=n;++i) 
		{
			for(int j=1;j<=m;++j)
			{
				a[i][j]=read(); 
				maxx=max(maxx,a[i][j]);
				num[(i+j)&1]++; sum[(i+j)&1]+=a[i][j];
			}
		}
		if(num[0]==num[1]) 
		{
			if(sum[1]!=sum[0]) return puts("-1"),void();
			int l=maxx,r=1e18,ans;
			while(l<=r)
			{
				int mid=(l+r)>>1;
				if(check(mid)) ans=mid,r=mid-1;
				else l=mid+1;
			}
			if(ans==1e18) return puts("-1"),void();
			else return printf("%lld\n",ans*num[1]-sum[1]),void();
		}
		else
		{
			int x=(sum[0]-sum[1])/(num[0]-num[1]);
			if(x>=maxx&&check(x)) printf("%lld\n",x*num[1]-sum[1]);
			else return puts("-1"),void();
		} return ;
	}
	signed main()
	{
//		freopen("8.in","r",stdin); 
		int T=read(); while(T--) work();
		return 0;
	}
}
signed main(){return yspm::main();}
```