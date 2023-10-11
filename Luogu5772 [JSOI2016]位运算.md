# Description

给定一个 $0/1$ 二进制串 $s(|s|\le 50)$，和两个整数 $n,k(n\le 7,k\le 10^5)$ 

从不大于将 $s$ 循环 $k$ 次后得到的二进制串的数中选不同的 $n$ 个，异或和为 $0$ 的方案

# Solution 

题目本质上是要使所有的位置上面满足有偶数个数字相同

同时还要满足所选的 $n$ 个数得互不相同

正难则反

每个位置上面的方案就是 $\sum\limits_{i\in \{Even\}}^n\binom n i$

（暂令 $R>x_1>x_2>\cdots>x_n$）

用 $f_{i,st}$ 表示已经从高到低考虑了 $i$ 位，$st$ 表示对于当前的位置 $i$， $x_i$ 和 $x_{i-1}$ 的相等关系，如果相等就是 $1$ ，反之则反之

然后转移的时候直接枚举这一次 $n$ 个数当前位要填的 $0/1$（这里可以状压）

如果满足原来两项相同但是现在后面的比前面的大，那么就不转移了

更新下来状态直接更新答案

这样复杂度显然是不行的，而且显然忽略了题目中对 $R$ 重复性的描述,又要矩阵快速幂

显然对于每个循环的转移方式都是一致的（因为字符串是一样的）
 
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
	const int mod=1e9+7,N=60,M=2<<7;
	inline int add(int x,int y){return x+y>=mod?x+y-mod:x+y;}
	inline int del(int x,int y){return x-y<0?x-y+mod:x-y;}
	char s[N];
	int n,k,len;
	struct node{
		int a[M][M];
		int* operator [](int x){return a[x];}
		inline void init(){memset(a,0,sizeof(a)); return ;}
		inline void prework(){for(int i=1;i<=(1<<n);++i) a[i][i]=1; return ;}
	};
	node ans,base;
	int S,f[60][60];
	inline node mul(node x,node y)
	{
		node ans; ans.init();
		for(int i=0;i<=S;++i)
		{
			for(int j=0;j<=S;++j) 
			{
				for(int k=0;k<=S;++k) ans[i][j]=add(ans[i][j],x[i][k]*y[k][j]%mod);
			}
		} return ans;
	}
	inline node ksm(node x,int y)
	{
		node res; res.init(); res.prework();
		for(;y;y>>=1,x=mul(x,x)) if(y&1) res=mul(res,x);
		return res;
	}
	int c[M],val[10];
	inline int lowbit(int x){return x&(-x);}
	inline int calc(int x){int cnt=0; for(;x;x-=lowbit(x)) cnt++; return cnt;}
	signed main()
	{
		n=read(); k=read(); scanf("%s",s+1); len=strlen(s+1); S=(1<<n)-1;
		for(int i=0;i<=S;++i) c[i]=calc(i);
		for(int st=0;st<=S;++st)//就是状态st向其它的状态进行转移的时候的预处理（大概的样子应该就是初始那列的吧……）
		{
			memset(f,0,sizeof(f)); f[0][st]=1;
			for(int i=1;i<=len;++i)//从高到低前 i 位
			{
				for(int j=0;j<=S;++j)//上一次到了这种状态
				{
					if(!f[i-1][j]) continue;
					for(int k=0;k<=S;++k)//枚举这一次的几个填数
					{
						if(c[k]%2!=0) continue;
						for(int num=1;num<=n;++num) val[num]=k>>(num-1)&1;
						val[0]=s[i]-'0';
						for(int num=1;num<=n;++num) cout<<val[num]<<" "; puts("");
						int res=0; bool fl=0;
						for(int num=1;num<=n;++num)
						{
							if(j&(1<<(num-1)))
							{
								if(val[num-1]<val[num]){fl=1; break;}
								if(val[num-1]==val[num])res|=1<<(num-1); 
							}
						}if(fl) continue;
						f[i][res]=add(f[i][res],f[i-1][j]); 
					}
				}
			}
			for(int i=0;i<=S;++i) base[st][i]=f[len][i];
		} 
		base=ksm(base,k); printf("%lld\n",base[S][0]);
		return 0;
	}
}
signed main(){return yspm::main();}
```
