# Description

[link](https://www.luogu.com.cn/problem/P2150)

给定 $n$ 求将 $2 \to n$ 的所有数分成两份，使得两份中的每一对数都互质的方案数，可以有数不在任何一份中

$n\le 500$ 

# Solution

不互质就是有共同的因子，但直接状压因为素数个数过多而复杂度无法接受

注意到一个数必然的构成要不是单独是一个质数，要不必然有一个 $\sqrt{500}$ 一下的质因子，即 $1\to 19\ \ $ $8$个素数

计算每个数字大于 $\sqrt n$ 的因子并排序，不难发现每个数最多有一个次数为 $1$ 的较大素因子

下面的 ”段“ 指那些较大质因子相同的数字组成的那个段，每个素数或 $1$ 自成一个段

具体地来说……设一个 $ans_{i,j}$ 来统计答案，$f_{x,y}$ 表示在段内的给第一个人统计的方法，$g_{x,y}$ 表示在段内给第二个人统计的方法

在段开始处理的时候把 $ans$ 的值赋给 $f,g$

如果 $S\&T=0$， 那么这种方案合法

对于段内的每个数字， 计算 它包含的小质数集合，能与的就进行转移

处理完每个段之后要 $ans_{S_1,S_2}=f_{S_1,S_2}+g_{S_1,S_2}-ans_{S_1,S_2}$ 

最后把所有的合法状态一统计即可

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
#define For(i,a,b) for(int i=a;i<=b;++i)
namespace yspm{
	inline int read(){
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=510,SZ=1<<8;
	int n,pri[N],cnt,fl[N],f[SZ][SZ],g[SZ][SZ],ans[SZ][SZ],mod;
	inline int add(int x,int y){return x+y>=mod?x+y-mod:x+y;}
	inline int del(int x,int y){return x-y<0?x-y+mod:x-y;}
	struct node{
		int id,val,k;
		#define k(i) num[i].k
		#define val(i) num[i].val
		#define id(i) num[i].id
		bool operator <(const node &a)const{return val<a.val;}
	}num[N];
	inline void calc(int x){
		int t=x; id(x)=x;
		for(int i=1;i<=min(cnt,8ll);++i){
			if(x%pri[i]==0){
				while(x%pri[i]==0) x/=pri[i];
				k(t)|=1<<(i-1);
			}
		} if(x>1) val(t)=x; else val(t)=-t; return ;
	}
	inline void seive(int x){
		For(i,2,x){
			if(!fl[i]) pri[++cnt]=i;
			for(int j=1;j<=cnt&&i*pri[j]<=x;++j){                           
				fl[i*pri[j]]=1; if(i%pri[j]==0) break; 
			}
		} For(i,2,x) calc(i); 
		return ;
	}
	inline int find(int x){
		int res=x; while(val(res+1)==val(res)) ++res; 
		return res;
	}
	int s=0,r;
	inline void work(){
		For(i,0,s) For(j,0,s) if(!(i&j)) ans[i][j]=del(add(f[i][j],g[i][j]),ans[i][j]);
		return ;
	}
	signed main(){
		n=read(); mod=read(); seive(n); s=1;
		for(int i=1;i<=cnt;++i) if(pri[i]<=19) s<<=1; else break;
		--s;  sort(num+2,num+n+1); 
		ans[0][0]=1; 
		for(int i=2;i<=n;++i){
			r=find(i);
			For(j,0,s) For(k,0,s) f[j][k]=ans[j][k],g[j][k]=ans[j][k];
			For(j,i,r){
				for(int s1=s;s1>=0;--s1){
					for(int s2=s;s2>=0;--s2){
						if(s1&s2) continue;
						if((k(j)&s1)==0) f[s1][s2|k(j)]=add(f[s1][s2|k(j)],f[s1][s2]);
						if((k(j)&s2)==0) g[s1|k(j)][s2]=add(g[s1|k(j)][s2],g[s1][s2]);
					}
				} 
			} i=r; work();
		} 	
		int tot=0;
		For(i,0,s) For(j,0,s) if(!(i&j)) tot=add(tot,ans[i][j]);
		printf("%lld\n",tot);
		return 0;
	}
}signed main(){return yspm::main();}
```