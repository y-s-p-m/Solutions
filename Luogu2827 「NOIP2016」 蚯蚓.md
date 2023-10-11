# Description

[link](https://www.luogu.com.cn/problem/P2827)

# Solution

相当 $naive$ 的 $O(m\log n)$ 的做法还是比较好说的

有个结论：先被砍的蚯蚓的长度的在当前时刻的剩余会比现在砍的剩余大

证明很简单，设数算一下就行了

那么这题目上手维护三个单调队列，最后揉起来算即可

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
#define reg register
namespace yspm{
	inline int read(){
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=1e5+10,M=7e6+10;
	int a[N],n,m,zsy,u,v,t,sum;
	priority_queue<int> ans;
	int q1[M],q2[M],q3[M],h1,h2,h3,t1,t2,t3;
	inline bool cmp(int x,int y){return x>y;}
	signed main(){
		n=read(); m=read(); zsy=read(); double p=read(); p/=read(); t=read();
		for(reg int i=1;i<=n;++i) a[i]=read();
		h1=h2=h3=1; t1=n,t2=t3=0;
		sort(a+1,a+n+1,cmp);
		for(reg int i=1;i<=n;++i) q1[i]=a[i];
		for(reg int i=1,rem1,rem2,mx=0,id=0;i<=m;++i) 
		{
			mx=0; 
			if(t1>=h1){
				if(q1[h1]>=q2[h2]&&q1[h1]>=q3[h3]) mx=q1[h1],++h1;
				else if(q2[h2]>=q1[h1]&&q2[h2]>=q3[h3]) mx=q2[h2],++h2;
				else mx=q3[h3],++h3;
			}else{
				if(q2[h2]>=q3[h3]) mx=q2[h2],++h2;
				else mx=q3[h3],++h3;
			}
			if(i%t==0){
				printf("%lld ",mx+sum);
			} mx+=sum;
			rem1=floor(mx*p); rem2=mx-rem1;
			sum+=zsy; 
			q2[++t2]=rem1-sum; q3[++t3]=rem2-sum;
		} puts("");
		for(reg int i=h1;i<=t1;++i) ans.push(q1[i]);
		for(reg int i=h2;i<=t2;++i) ans.push(q2[i]);
		for(reg int i=h3;i<=t3;++i) ans.push(q3[i]);
		int cnt=0; 
		while(ans.size()){
			++cnt; if(cnt%t==0) printf("%lld ",sum+ans.top()); 
			ans.pop();
		}puts("");
		return 0;
	}
}
signed main(){return yspm::main();}
```