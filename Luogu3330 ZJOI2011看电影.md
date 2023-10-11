# Description

[link](https://www.luogu.com.cn/problem/P3330)

有 $n$ 个人去一个有 $k$ 个座位的电影院看电影

当前人选座位的方式是随机一个数，如果那个数字已经有人了，那么就往后移一个

如果到第 $k$ 个都有了人，那么就只能站着了

求让所有人都坐着的概率

$n,k\le 200$

# Solution

特判 $k<n$ 的情况，总的情况数是 $k^n$ ，即每个人有 $k$ 种选择，共 $n$ 个人

（下面这里很巧妙！）

我们考虑构造一个环，连接最后一个点和起始点，即如果到了最后一个位置没法坐就到第一个位置，这样座位总是够坐的

这样我们发现还是没法统计

那么我们建立第 $k+1$ 个位置作为虚拟点

所以答案的分子就是没有人坐最后一个虚拟点的方案

我们断环成链，每次找那个最后一个位置没有人的方案

一共有 $(k+1)^{n-1}$ 种总方案，环的重复是有 $k+1$ 种的，每次断环成链最后一个位置没有人就是最后一个位置为空的方案

所以答案就顺理成章地变成了 $\frac{(k+1)^{n-1}\times (k-n+1)}{k^n}$

# Code

```cpp
const int N=3010;
struct node{
	int a[N],len;
	inline void init(int x){memset(a,0,sizeof(a)); len=0; while(x) a[++len]=x%10,x/=10; return ;}
	inline void print(){for(int i=len;i;--i) putchar('0'+a[i]);  printf(" "); return ;}
	inline void mul(int x){
		for(int i=1;i<=len;++i) a[i]=a[i]*x;
		for(int i=1;i<=len;++i) a[i+1]+=a[i]/10,a[i]%=10;
		while(a[len+1]) len++,a[len+1]+=a[len]/10,a[len]%=10;
		return ;
	}
};
int fl[N],f[N],n,k,p[N],cnt;
int d1[N],d2[N];
inline void work(){
	memset(d1,0,sizeof(d1)); memset(d2,0,sizeof(d2));
	int n=read(),k=read();
	if(n>k) return puts("0 1"),void();
	int t=k+1-n; 
	d1[k+1]+=n-1; d2[k]+=n; d1[k+1-n]++; 
	for(int i=k+1;i>=2;--i) 
		if(fl[i]==i) continue; 
		else d1[fl[i]]+=d1[i],d1[i/fl[i]]+=d1[i];
	for(int i=k+1;i>=2;--i) 
		if(fl[i]==i) continue; 
	else d2[fl[i]]+=d2[i],d2[i/fl[i]]+=d2[i]; 
	for(int i=1;i<=k+1;++i) 
	{
		if(fl[i]!=i) continue;
		int p=min(d1[i],d2[i]); d1[i]-=p; d2[i]-=p;
	}
	node a1,a2; a1.init(1); a2.init(1);
	for(int i=1;i<=N-1;++i){
		if(fl[i]==i){
			while(d1[i]--) a1.mul(i); 
			while(d2[i]--) a2.mul(i);
		}
	}
	a1.print(); a2.print(); puts("");
	return ;
}
inline void prework(){
	for(int i=2;i<N;++i){
		if(!fl[i]) fl[i]=i,p[++cnt]=i;
		for(int j=1;j<=cnt&&p[j]*i<N;++j){
			if(p[j]>fl[i]) break;
			fl[p[j]*i]=p[j];
		}
	} return ;
}
signed main(){
	prework();
	int T=read(); while(T--) work();
	return 0;
}
```

# Review

当我们在发现当下的统计不方便的时候，不如考虑新建虚拟点的类似操作

然后把答案转化出来，再进行统计