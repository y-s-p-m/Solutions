# Description

[link](https://www.luogu.com.cn/problem/CF1355F)

交互题

要求猜一个正整数 $X(X\le 10^9)$

每次可以给出一个 $Q$ 会从交互库里得到 $gcd(X,Q)$

在不超过 $22$ 次询问里面求出 $X$ 的个数

容错机制：

设输出为 $out$ 则满足一下之一即可

$1.\frac 1 2\le\frac{out}{ans}\le 2$

$2.|ans-out|\le 7$

# Solution

参考了@天下我有的题解，真的很不错的做法

先把 $\sqrt {10^9}$ 之内的质数筛出来

把所有的数扔到一个优先队列里面，队列里面是 $(val,y,base)$ 的三元组

其中 $val$ 是当前的值，$base$是基质数，$y$ 是幂

队列按照幂次为第一关键字，值为第二关键字，第一降序，第二升序

执行二十二次以下操作：

取出队列里面所有的能用的元素，把他们乘起来输出

然后得到答案后有 $gcd$ 的质因子把他们幂次加一扔到队列里面

互质的就不回放了

答案处理相对简单，最后乘 $2$ 输出

正确性？

首先可以试到 $900$ 左右的质数（这里直接考虑质数阶乘就行的……）

所以这部分质因子没问题

然后对于所有没有讨论到的情况分：单个质数，质数平方，两质数乘积

正确性想一下就发现没问题

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
	const int N=4e4+10;
	bool fl[N]; 
	int p[N],T,cnt,ans;
	struct node{
		int base,y,val;
		bool operator< (const node a) const
		{
			return a.y==y?a.val<val:a.y>y;
		}
	}t[N];
	priority_queue<node> q;
	signed main()
	{
		T=read();
		for(int i=2;i<N;++i)
		{
			if(fl[i]) continue; p[++cnt]=i;
			for(int j=i*i;j<N;j+=i) fl[j]=1;
		}
		while(T--)
		{
			while(q.size()) q.pop(); ans=1;
			for(int i=1;i<=cnt;++i) q.push((node){p[i],1,p[i]});
			for(int j=1;j<=22;++j)
			{
				int res=1,num=0;
				while(q.size())
				{
					node fr=q.top(); q.pop();
					if(1.0*res*fr.val>1e18){q.push(fr); break;}
					t[++num]=fr; res=fr.val*res;
				}
				printf("? %lld\n",res);
				fflush(stdout); 
				res=read();
				for(int i=1;i<=num;++i)
				{
					if(res%t[i].val==0) 
					{
						ans/=t[i].y; ++t[i].y; ans*=t[i].y;
						t[i].val*=t[i].base;
						q.push(t[i]);
					}
				}
			} printf("! %lld\n",ans<<1); fflush(stdout);
		}
		
		return 0;
	}
}
signed main(){return yspm::main();}
```