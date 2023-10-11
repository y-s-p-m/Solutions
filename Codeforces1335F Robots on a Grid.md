蒟蒻怕是颓了  $std$

# Description

[link](https://www.luogu.com.cn/problem/CF1335F)

题意：给一个矩阵，上面有$ULRD$四种指向

同时这个矩阵的每个元素还有黑或白两种属性

在该格中则下一秒就到了指向的那个格子

现在在矩阵中放机器人，如果两个机器人在若干秒后相撞了，则该摆放不合法

求最大合法摆放个数和以之为前提最多有多少黑格

# Solution

考场想到了二分图……显而易见地不入流

我们发现机器人的走法可能构成了一个环，长度$\le n \times m$

如果两个机器人在某个时候相遇了，那么它们会接着同步走

考虑到环长，所以如果两个机器人会相遇，那么在$n \times m$ 步后会在一个格子上面

我们用倍增实现求从 $(i,j)$ 开始 $n \times m$ 步会到达的位置即可

最后用桶一下就可以了

最后，这个题坑点还有清空数组，桶需要在求完了就即时清空

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
	const int N=1e6+10;
	int b[N],w[N],col[N]; char s[N];
	int nxt[24][N],n,m;
	inline int id(int x,int y)
	{
		return m*(x-1)+y;
	}
	inline void work()
	{
		n=read(); m=read(); 
		for(int i=1;i<=n;++i) 
		{
			scanf("%s",s+1);
			for(int j=1;j<=m;++j) col[id(i,j)]=s[j]-'0';
		}
		for(int i=1;i<=n;++i)
		{
			scanf("%s",s+1);
			for(int j=1;j<=m;++j)
			{
				int now=id(i,j);
				if(s[j]=='U') nxt[0][now]=now-m;
				else if(s[j]=='D') nxt[0][now]=now+m;
				else if(s[j]=='L') nxt[0][now]=now-1;
				else nxt[0][now]=now+1;
			}
		} 
		n*=m;
		for(int i=1;i<=20;++i) for(int j=1;j<=n;++j) nxt[i][j]=nxt[i-1][nxt[i-1][j]];
		for(int j=1;j<=n;++j) 
		{
			int to=j;
			for(int i=20;i>=0;--i) if((1<<i)&n) to=nxt[i][to];
			if(col[j]) w[to]=1; else b[to]=1;
		}
		int ans=0,maxx=0;
		for(int i=1;i<=n;++i)
		{
			if(b[i]) ans++,maxx++,b[i]=w[i]=0;
			else if(w[i]) ans++,b[i]=w[i]=0; 
		}printf("%lld %lld\n",ans,maxx);
		return ;
	}
	signed main()
	{
		int T=read(); while(T--) work();
		return 0;
	}
}
signed main(){return yspm::main();}
```