# Description

有 $n$ 个人一起做题，每个人有 $a_i$ 个题目需要 $1$ 分钟完成，$b_i$ 道两分钟题，$c_i$ 道三分钟题。判定在每个人任意选择做题顺序的情况下，是否可能 第 $i$ 个人是完成前 $i$ 到题目最快的人（之一）。

$n\le 2\times 10^5$

# Solution

如果确定每个人前 $i$ 个数字的 $1,2,3$ 分配 $(x,y,z)$，那么显然在前 $i$ 题以及后 $n-i$ 题都会是按照耗时倒序做

考虑维护序列 $\{T\}$ ，其中 $T_i$ 表示对第 $i$ 个人前 $i$ 道题目完成时间的限制，初始化为每个人都按耗时倒序做题时所需要花费的时间的最小值。

倒序处理每个人调整策略后对前缀的影响，即处理到 $i$ 时 $T_i$ 为 $(i,n]$ 中每个人调整策略后做前 $i$ 题的时间以及 $[1,i]$ 中每个人完全倒序做题的时间中的最小值。此时需要调整 $i$ 倒序做题的策略来使得用时 $\le T_i$

设 $i$ 完成的前 $i$ 个题目中 $1,2,3$ 分钟题的数量分别为 $x,y,z$，于是有 $x+2y+3z\le T_i,x+y+z=i$ 。那么要在最大化 $x+2y+3z$ 的基础上最大化 $x$。无解判据即找不到这样的 $x,y,z$

得到合法方案之后再去更新前缀 $[1,i)$ 的 $T_i$。

一个比较明显的问题没有处理 $i<j$ 的对子 $(i,j)$ 中 $i$ 对 $j$ 的影响。官方题解中的证明，大意是如果经过 $i$ 的策略调整后 $T_{i-1}+2\ge T_i$ 于是后面 $T$ 的折线斜率不超过 $2$，于是如果抛去 $x,y,z$ 后用时为 $1$ 的题目过多一定会在处理 $j$ 的时候影响到。

这个做法看起来是 $\Theta(n^2)$ 的，但是每个人做题的耗时本质上是折线，而更新前缀的过程中也是让折线和折线取 $\min$。这条折线也非常特殊，斜率只有 $1,2,3$ 三种，那么可以维护每个 $x+b_1,2x+b_2,3x+b_3$ 中的 $b_1,b_2,b_3$ 而询问折线某处取值就是代入表达式取 $\min$ 。上面提到的取 $\min$ 操作即将两条折线的 $b_1,b_2,b_3$ 对位取 $\min$

寻找 $x+2y+3z$ 的合法值时可以从 $T_i$ 向下暴力枚举取值，总枚举量是 $\le 3n$ 的。合法性等价于是否存在 $x$ 满足 $0\le x\le a_i$ 另外的两条限制加在 $0\le y\le b_i,0\le z\le c_i$ 上，可以根据 $y+z=i-x,2y+3z=t-x$ 而 $i,t$ 是定值得到 $y,z$ 关于 $x$ 的表达式。

前缀折线的表达式据 $(t,x,i)$ 可以得到。

复杂度为 $\Theta(n)$

# Code

```cpp
const int N=2e5+10,inf=0x3f3f3f3f;
int n,a[N],b[N],c[N];
struct Line{
	int a,b,c;
	//K=1,2,3 时对应的截距
	Line(){a=b=c=inf;}
	Line(int x,int y,int z){a=x; b=y; c=z; return ;}
	inline int at(int x){return min({a+x,b+2*x,c+3*x});}
};
inline Line Merge(Line a,Line b){return Line(min(a.a,b.a),min(a.b,b.b),min(a.c,b.c));}
int main(){
	int T=read();
	while(T--){
		n=read();
		Line A;
		rep(i,1,n){
			a[i]=read(),b[i]=read(),c[i]=read();
			A=Merge(A,Line(b[i]+2*c[i],c[i],0));
		}
		bool illegal=0;
		for(int i=n;i>=1;--i){
			int v=A.at(i);
			while(v>=0){
				int lhs=min(a[i],min((3*i-v)/2,c[i]+2*i-v));
				int rhs=max(0,max(2*i-v,(3*i-v-b[i]+1)/2));
				if(lhs>=rhs) break;
				--v;
			}
			if(v<0){
				illegal=1;
				break;
			}
			int x=min(a[i],min((3*i-v)/2,c[i]+2*i-v));
			int B=3*i-v-2*x,C=v+x-2*i;
			A=Merge(A,Line(B+2*C,C,0));
		}
		puts(illegal?"No":"Yes");
	}
	return 0;
}
```