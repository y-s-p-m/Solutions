做多网络流之后来个期望水一发

# Description

[link](https://www.luogu.com.cn/problem/CF258D)

有一个 $1 \sim n$ 的排列，会进行 $m$ 次操作，操作为交换 $a,b$。每次操作都有 $50\%$ 的概率进行。 

求进行 $m$ 次操作以后的期望逆序对个数。

$n,m\le 1000$

# Solution

首先定义状态：$f_{i,j}$ 为 $i$ 比 $j$ 大的概率

答案就是

$$ans=\sum_{i=1}^{n-1} \sum_{j=i+1}^n f_{i,j}$$

初态很好维护

然后我们考虑每个修改

这里的修改会对每个位置造成影响

那么

$$f_{x,i}=\frac{f_{x,i}+f_{y,i}}{2}$$

显然吧……

每次交换的时候都这么改，最后统计答案

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
	const int N=1010;
	double f[N][N],ans;
	int a[N],n,m;
	signed main()
	{
		n=read(); m=read(); for(int i=1;i<=n;++i) a[i]=read();
		for(int i=1;i<=n;++i)
		{
			for(int j=i+1;j<=n;++j)
			{
				if(a[i]>a[j]) f[i][j]=1;
				else f[j][i]=1;
			}
		}
		while(m--)
		{
			int x=read(),y=read();
			for(int i=1;i<=n;++i) 
			{
				f[i][x]=f[i][y]=(f[i][x]+f[i][y])/2;
				f[x][i]=f[y][i]=(f[x][i]+f[y][i])/2;
			}f[x][y]=f[y][x]=0.5;
		}
		for(int i=1;i<=n;++i)
		{
			for(int j=i+1;j<=n;++j) ans+=f[i][j];
		} printf("%.10lf\n",ans); 
		return 0;
	}
}
signed main(){return yspm::main();}
```

# Review

定义状态的时候要思维灵活，可以试试本题这种比较妙的定义方法

就是把总答案下放给位置，最后进行统计