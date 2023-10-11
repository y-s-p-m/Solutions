# Description

区间加，区间去 $\rm AND$ ，区间求和

$n\le 3\times 10^5,Q\le 2\times 10^5,a_i,v\le 2^{128}$

# Solution

特判除数 $v=1$ 的情况，那么暴力进行除法操作复杂度就是 $\Theta(n\log V)$ ，可以接受

常规思路维护 $\log V$ 个二进制数表示区间里面 $a_i$ 在第 $1\sim \log V$ 位中为 $1$ 的数量是多少，但是复杂度 $\Theta(n\log^2V+q\log V\log n)$ ，不能通过

注意到在常规思路中线段树每个节点本质上维护的是一个 $\log V\times \log n$ 的数表

第二维大小是 $\log n$ 的原因是数字数量不会超过 $n$，更严格的，应当为 $\log len$

**那么可以将这个数表进行翻转！**现在变成维护 $\log n$ 个长度为 $\log V$ 的二进制数

区间求和就是 $val_i\times2^{i}$ ，除法计算时直接使用 $val_0$ 就是真实的节点权值，甚至不用重构叶子处的数表，取 $\rm AND$ 就是对所有 $val_i$ 进行操作

剩余的唯一问题就是 `push_up` 操作中数表本身的维护，注意到每位的权值是独立的，那么维护第 $i-1$ 位两个子区间向量的进位情况 $\rm carry$（压缩成一个长度为 $128$ 的 $0/1$ 串）

不难发现 $val_i$ 就是 $\displaystyle val_{\rm lson}\oplus val_{\rm rson}\oplus \rm carry$ ，而在两者加法中完成进位的不会再进一次，或起来即可

时间复杂度降至 $\Theta(n\log V+q\log^2n)$ ，需要加入的常数优化有二：

- 数表的大小是 $\log len$ ，所以需要使用给每个节点在内存池中分配空间而不是开到 $N\times \log n$ ，注意总空间是 $\sum 2^i\log\lfloor \frac{n}{2^i}\rfloor\le n\sum\frac{i}{2^i}\approx n$

- 区间全零后在区间取 $\rm AND$ 和区间求和也可以进行剪枝

# Code

```cpp
const int N=3e5+10;
int n,Q,num[N<<2];
u128 Memory_pool[N*5+1000],*indic;
u128 a[N],tag[N<<2],*val[N<<2];
bool zero[N<<2];
#define ls p<<1
#define rs p<<1|1
#define lson p<<1,l,mid
#define rson p<<1|1,mid+1,r
inline void push_up(int p){
	u128 carry=0;
	for(int i=0;i<=num[p];++i){
		u128 x=i<=num[ls]?val[ls][i]:0;
		u128 y=i<=num[rs]?val[rs][i]:0;
		val[p][i]=x^y^carry;
		carry=((x^y)&carry)|(x&y);
	}
	zero[p]=zero[ls]&zero[rs];
	return ;
}
inline void push_tag(int p,u128 v){
	if(zero[p]) return ;
	tag[p]&=v;
	rep(i,0,num[p]) val[p][i]&=v;
	return ;
}
inline void push_down(int p){
	if(~tag[p]){
		push_tag(ls,tag[p]);
		push_tag(rs,tag[p]);
		tag[p]=-1;
	} return ;
}
inline void build(int p,int l,int r){
	tag[p]=-1; num[p]=31-__builtin_clz(r-l+1);
	val[p]=indic; indic+=num[p]+1;
	if(l==r){
		if(!(val[p][0]=a[l])) zero[p]=1;
		return ;
	}
	int mid=(l+r)>>1;
	build(p<<1,l,mid); build(p<<1|1,mid+1,r);
	return push_up(p);
}
inline u128 Query(int st,int ed,int p=1,int l=1,int r=n){
	if(zero[p]) return 0;
	if(st<=l&&r<=ed){
		u128 res=0;
		rep(i,0,num[p]) res+=val[p][i]<<i;
		return res;
	}
	int mid=(l+r)>>1; push_down(p);
	u128 res=0;
	if(st<=mid) res+=Query(st,ed,lson);
	if(ed>mid) res+=Query(st,ed,rson);
	return res;
}
inline void Div(int st,int ed,u128 v,int p=1,int l=1,int r=n){
	if(zero[p]) return ;
	if(l==r){
		zero[p]=!(val[p][0]/=v);
		return ;
	}
	int mid=(l+r)>>1; push_down(p);
	if(st<=mid) Div(st,ed,v,lson);
	if(ed>mid) Div(st,ed,v,rson);
	return push_up(p);
}
inline void And(int st,int ed,u128 v,int p=1,int l=1,int r=n){
	if(zero[p]) return ;
	if(st<=l&&r<=ed) return push_tag(p,v);
	int mid=(l+r)>>1; push_down(p);
	if(st<=mid) And(st,ed,v,lson);
	if(ed>mid) And(st,ed,v,rson);
	return push_up(p);
}
#undef ls
#undef rs
#undef lson
#undef rson
signed main(){
	n=read(); Q=read();	
	for(int i=1;i<=n;++i) a[i]=read_16();
	indic=Memory_pool;
	build(1,1,n);
	while(Q--){
		int opt=read(),l=read(),r=read();
		if(opt==1){
			u128 v=read_16();
			if(v!=1) Div(l,r,v);	
		}
		if(opt==2){
			u128 v=read_16();
			And(l,r,v);	
		}
		if(opt==3) output(Query(l,r)),putchar('\n');
	}
	return 0;
}
```