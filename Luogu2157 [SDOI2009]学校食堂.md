# Description

[link](https://www.luogu.com.cn/problem/P2157)

给定 $n$ 个学生的口味和忍耐度，若前一道菜的对应的口味是a，这一道为b，则做这道菜所需的时间为 $(a| b)-(a\&b)$，而做第一道菜是不需要计算时间的.

每个学生可以忍耐忍耐度以下的人在他前面插队买饭

求最小的做饭时间

$n\le 1000,b_i\le 8$ 

# Solution

令 $f_{i,S}$ 表示第 $i$ 个人前面的状态是 $S$ 的最小花费

令 $f_{i,S}$ 表示第 $i$ 个人后面的状态为 $S$

其实状态有点等效，从前还是从后转移而已……

但是都不会转移……所以去抄题解……

设 $f_{i,st,k}$ 为第 $1$ 到 $i-1$ 个人已经打完饭，然后第 $i$ 个人和后面的 $7$ 个人的打完饭与否的状态为 $j$ ，当前打完饭的最后一个人是 $k+i$ 的最小时间

不能转移的都设置成 $inf$

分类讨论转移

---

$1.$ 不符合状态的定义

无意义的状态要避免循环，就是 $f_{i,j,st}=inf$

$2.$ 转移状态 

$(1)$ 在 $st\&1==1$ 时

$$f_{i+1,st>>1,k}=min(f_{i+1,st>>1,k-1},f_{i,st,k})$$

（这俩状态一致，所以直接转移）

$(2)$ 

第 $i$ 个人吃饭，往下转移（这里要注意的是否满足 $j\&1==0$）

第 $i$ 个人后面的第几个人吃饭，这里还得判断是不是让人愤怒了，这里做个最大可行位置就成的了

判断是不是第一个吃饭的时候直接考虑 $i+k==0$ 就好


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
	const int inf=0x3f3f3f3f3f3f3f3f;
	int n,b[N],t[N],f[N][256][17];
	inline int id(int x){return x+8;}
	inline void work()
	{
		memset(f,0x3f,sizeof(f));
		n=read(); for(int i=1;i<=n;++i) t[i]=read(),b[i]=read();
		int s=(1<<min(n,8ll))-1; f[1][0][id(-1)]=0; 
		for(int i=1;i<=n;++i)
		{
			for(int j=0;j<=s;++j)
			{
				for(int k=-8;k<=b[i]&&i+k<=n;++k)
				{
					if(f[i][j][id(k)]==inf) continue;
					if(j&1){f[i+1][j>>1][id(k-1)]=min(f[i+1][j>>1][id(k-1)],f[i][j][id(k)]); continue;}
					int rpos=n;
					for(int l=0;l<=b[i]&&i+l<=rpos;++l)
					{
						if(j&(1<<l)) continue;
						rpos=min(i+l+b[i+l],rpos);	
						if(i+k==0) f[i][j|(1<<l)][id(l)]=f[i][j][id(k)]+0;
						else f[i][j|(1<<l)][id(l)]=min(f[i][j|(1<<l)][id(l)],f[i][j][id(k)]+(t[i+k]|t[i+l])-(t[i+k]&t[i+l]));
					}
				}
			} 
		} 
		int ans=1e18+10;
		for(int i=-8;i<=0;++i) ans=min(ans,f[n+1][0][id(i)]); cout<<ans<<endl; 
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

# Review

大概学到了这种加一维控制转移的题目

其实这题目和“奇怪的道路”确实很类似