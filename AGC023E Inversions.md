# Description

给定一个长度为 $n$ 的序列 $A$，问所有满足 $\forall i,P_i\le A_i$ 的 $1\sim n$ 的排列的逆序数的和为多少

答案对 $10^9+7$ 取模

# Solution

设 $c_i$ 是将 $a_i$ 排序后的结果，$b_i$ 是 $a_i$ 排名，那么总合法排列数是 $S=\prod\limits_{i=1}^nc_i-i+1$

考虑一对 $i,j$ 且 $i<j,a_i<a_j$ 能在多少个排列中形成逆序对

由于 $p_j>a_i$ 时一定不能形成逆序对那么可以将 $a_j$ 对 $a_i$ 取 $\min$ ，此时在一个合法排列中将 $i,j$ 交换依然合法，那么有半数合法排列能形成逆序对

这个方案数列式表达为 $\displaystyle\dfrac 12 S\dfrac{a_j-b_j}{a_i-b_i+1}\prod\limits_{b_i+1}^{k=b_j-1}\dfrac{c_k-k}{c_k-k+1}$

考虑另外一种 $i<j,a_i>a_j$ 的情况，大可计算顺序对方案数再用总方案数减去，表达式类似

优化这个复杂度为 $\Theta(n^2)$ 的做法的唯一出路是找到定量：从小到大加入每个 $a_i$ 那么后面的乘积使用一个线段树来求贡献总和，每个叶子 $j$ 存 $a_j-b_j$

每次一个 $i$ 求完答案后全局附上 $\dfrac{c_i-i}{c_i-i+1}$ 的乘法标记再激活 $i$ 号叶子即可

根据表达式，还需要维护某个区间的被激活叶子数量

时间复杂度 $\Theta(n\log n)$

# Code

```cpp
const int N=5e5+10;
int inv[N],n;
pair<int,int> a[N];
#define ls p<<1
#define rs p<<1|1
#define lson p<<1,l,mid
#define rson p<<1|1,mid+1,r
int sum[N<<1],tag[N<<1],cnt[N<<1];
inline void push_up(int p){
	cnt[p]=cnt[ls]+cnt[rs];
	sum[p]=add(sum[rs],sum[ls]);	
}
inline void push_tag(int p,int v){ckmul(tag[p],v); ckmul(sum[p],v);}
inline void push_down(int p){
	if(tag[p]!=1){
		push_tag(ls,tag[p]);
		push_tag(rs,tag[p]);
		tag[p]=1;
	}
}
inline void build(int p,int l,int r){
	tag[p]=1;
	if(l==r) return ;
	int mid=(l+r)>>1;
	build(lson); build(rson);
}
inline int query_sum(int st,int ed,int p=1,int l=1,int r=n){
	if(ed<st) return 0;
	if(st<=l&&r<=ed) return sum[p];
	int mid=(l+r)>>1,res=0; push_down(p);
	if(st<=mid) ckadd(res,query_sum(st,ed,lson));
	if(ed>mid) ckadd(res,query_sum(st,ed,rson));
	return res;
}
inline int query_cnt(int st,int ed,int p=1,int l=1,int r=n){
	if(ed<st) return 0;
	if(st<=l&&r<=ed) return cnt[p];
	int mid=(l+r)>>1,res=0; push_down(p);
	if(st<=mid) res+=query_cnt(st,ed,lson);
	if(ed>mid) res+=query_cnt(st,ed,rson);
	return res;
}
inline void Modify(int pos,int v,int p=1,int l=1,int r=n){
	if(l==r){
		cnt[p]=1;
		sum[p]=v;
		return ;
	}
	int mid=(l+r)>>1; push_down(p);
	if(pos<=mid) Modify(pos,v,lson); else Modify(pos,v,rson);
	return push_up(p);
}
#undef ls 
#undef rs 
#undef lson
#undef rson
signed main(){
	n=4e5+10; inv[0]=inv[1]=1;
	for(int i=2;i<=n;++i) inv[i]=mod-mul(inv[mod%i],mod/i);
	n=read();
	rep(i,1,n) a[i]={read(),i};
	sort(a+1,a+n+1,[&](const pii a,const pii b){return a.fir<b.fir;});
	int ans=0;
	int U=1;
	for(int i=1;i<=n;++i){
		if(a[i].fir<i) puts("0"),exit(0);
		ckmul(U,a[i].fir-i+1);
	}
	for(int i=1;i<=n;++i){
		int res=0;
		res=del(query_sum(1,a[i].sec-1),query_sum(a[i].sec+1,n));
		ckmul(res,mul(U,inv[2*(a[i].fir-i+1)]));
		ckadd(res,mul(U,query_cnt(a[i].sec+1,n)));
		ckadd(ans,res);	
		
		push_tag(1,mul(a[i].fir-i,inv[a[i].fir-i+1]));
		Modify(a[i].sec,a[i].fir-i);
	}
	print(ans);
	return 0;
}
```