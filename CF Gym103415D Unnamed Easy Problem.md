# Description

计算满足下列条件的 $[m\times n]0/1$ 矩阵的数量：

- 每行后 $n-k$ 列至少有一个 $1$

- 每行互不相同

- $1\sim p$ 列共有奇数个 $1$，$k+1\sim k+q$ 列共有奇数个 $1$

- 行之间无序

$1\le k<n\le 10^9,m\le 10^6$

# Solution

先不考虑最后一个限制，最后除以 $m!$ 来无序化

奇偶性的限制可以使用烂大街的套路：最后一行来抵消前面行的奇偶性需求，前面的行任意选

那么需要处理此时不满足第一和第二条限制的问题，将不合法方案数减去

设 $f_m$ 表示行数为 $m$ 时的答案，$g_m$ 表示行数为 $m$ 且不一定满足限制 $1$ 时的答案

前者转移可以用 $\displaystyle\binom{2^n-2^k}{m-1}(m-1)!$ 减掉出现相同的情况数 $(m-1)(2^n-2^k-(n-2))f_{m-2}$ 再减掉不满足最后一行后 $n-k$ 列至少有 $1$ 个 $1$ 的方案数

那么非常精彩地，现在最后一行的后 $n-k$ 列没有任何 $1$ 了，前 $m-1$ 行的后 $n-k$ 列奇偶性已经满足，所以这是 $g_{m-1}$

后者的转移由于不考虑前 $k$ 列的合法性，所以总方案数要附加最后一行前 $k$ 列乱选的 $2^k$ 的系数，出现相同还是 $(m-1)(2^n-2^k-(n-2))g_{m-2}$ 

不满足有 $1$ 的部分也是符合 $g_{m-1}$ 的定义，但是由于最后一行前 $k$ 列可以乱选，那么也要附上 $2^k$ 作为系数

时间复杂度 $\Theta(m)$

# Code

```cpp
const int N=1e6+10;
int f[N],g[N];
int n,m,k,oee,oez;
signed main(){
	n=read(); k=read(); m=read(); 
	oee=read(),oez=read();
	int s=ksm(2,k),tot=del(ksm(2,n),ksm(2,k));
	int fac=1,P=1;
	f[0]=!oez&&!oee;
	g[0]=(bool)!oez;
	f[1]=(bool)oez;
	g[1]=f[1]*s;
	for(int i=2;i<=m;++i){
		int cho=del(tot,i-2);
		ckmul(P,cho);
		ckmul(fac,i);
		f[i]=del(P,g[i-1]);
		ckdel(f[i],mul(i-1,mul(f[i-2],cho)));
		g[i]=mul(s,del(P,g[i-1]));
		ckdel(g[i],mul(i-1,mul(g[i-2],cho)));
	}
	print(mul(f[m],ksm(fac,mod-2)));
	return 0;
}
```