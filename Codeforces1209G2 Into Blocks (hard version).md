# Description

给你 $n,q$，$n$ 表示序列长度，$q$ 表示操作次数。

我们需要达成这么一个目标状态：如果存在 $x$ 这个元素，那么必须满足所有 $x$ 元素都必须在序列中连续。

然后你可以进行这么一种操作，将所有的 $x$ 元素的变为任意你指定的 $y$ 元素，并且花费 $cnt[x]$ 的花费，$cnt[x]$ 代表 $x$ 元素的个数。

现在有 $q$ 次询问，每次询问单点修改一个位置的值，求修改完之后最小花费使得序列满足目标状态。

注意：更新不是独立的，之前的更新会保留。

$n,q\le 2\times 10^5$

# Solution

暴力做法就是将计算出来每个颜色 最早&晚 的出现位置 $s_c,t_c$，并将其视作一个区间

可以将 $n$ 个元素分成若干连续段，每段内的颜色的出现的位置的 $\min,\max$ 都在这个段内，或者说这样的划分是 “极长的”

另外一种刻画方式是将上述两个位置记成区间 $[s_c,t_c)$ 表示该区间中的每个 $i$ 的颜色和 $i+1$ 的颜色相同

对每个颜色的 $[s_c,t_c)$ 包含的 $i$ 加一那么上面说的不交区间就是上个 $0$ 权点以右，当前零权点（包含）以左

这时可以利用区间最小值来表示两个段的分界点：全局最小值一定存在取值点 $n$，因为其不会在右开的情况下被加一，而其它符合全局最小值的点必然是 $0$ 权点

如果实现了将每段内的元素挂到段尾的 $0$ 进行统计即可完成本题

将每种颜色的出现次数作为其第一次出现位置的权值 $w$，线段树上每个节点需要维护每个区间最小值靠左（包含自身）的最大的 $w$，区间最小值靠右（不包含自身）的最大 $w$，以及最小值中间段的和

使用 `std::set` 来维护每个颜色的所有出现次数，时间复杂度 $\Theta(n\log n)$

# Code

```cpp
const int N=2e5+10;
int n,Q,a[N];
int sum[N<<2],Mx[N<<2],Mn[N<<2],tag[N<<2],lef[N<<2],rig[N<<2];
#define ls p<<1
#define rs p<<1|1
#define lson p<<1,l,mid
#define rson p<<1|1,mid+1,r
inline void push_tag(int p,int v){tag[p]+=v; Mn[p]+=v;}
inline void push_down(int p){
	if(tag[p]){
		push_tag(ls,tag[p]); push_tag(rs,tag[p]);
		tag[p]=0;
	} return ;
}
inline void push_up(int p){
	Mx[p]=max(Mx[ls],Mx[rs]);
	if(Mn[ls]<Mn[rs]){
		Mn[p]=Mn[ls];
		lef[p]=lef[ls]; rig[p]=max(Mx[rs],rig[ls]);
		sum[p]=sum[ls];
	}else if(Mn[rs]<Mn[ls]){
		Mn[p]=Mn[rs];
		lef[p]=max(lef[rs],Mx[ls]); rig[p]=rig[rs];
		sum[p]=sum[rs];
	}else{
		Mn[p]=Mn[ls];
		sum[p]=sum[ls]+sum[rs]+max(rig[ls],lef[rs]);
		lef[p]=lef[ls]; rig[p]=rig[rs];
	}
	return ;
}
inline void Plus(int st,int ed,int d,int p=1,int l=1,int r=n){
	if(st<=l&&r<=ed) return push_tag(p,d);
	int mid=(l+r)>>1; push_down(p);
	if(st<=mid) Plus(st,ed,d,lson); 
	if(ed>mid) Plus(st,ed,d,rson);
	return push_up(p);
}
inline void Modify(int pos,int v,int p=1,int l=1,int r=n){
	if(l==r) return lef[p]=Mx[p]=v,void();
	int mid=(l+r)>>1; push_down(p);
	if(pos<=mid) Modify(pos,v,lson);
	else Modify(pos,v,rson);
	return push_up(p);
}
#undef ls
#undef rs
#undef lson
#undef rson
set<int> app[N];
inline void ins(int c){
	if(!app[c].size()) return ;
	Modify(*app[c].begin(),app[c].size());
	if(app[c].size()>1) Plus(*app[c].begin(),*prev(app[c].end())-1,1);
}
inline void erase(int c){
	if(!app[c].size()) return ;
	Modify(*app[c].begin(),0);
	if(app[c].size()>1) Plus(*app[c].begin(),*prev(app[c].end())-1,-1);
}
signed main(){
	n=read(); Q=read();
	for(int i=1;i<=n;++i) app[a[i]=read()].insert(i);
	for(int i=1;i<=200000;++i) ins(i);
	print(n-sum[1]-lef[1]-rig[1]);
	while(Q--){
		int x=read(),y=read();
		erase(a[x]);
		app[a[x]].erase(x);
		ins(a[x]);
		
		erase(y);
		app[y].insert(x);
		ins(y);
		a[x]=y;
		print(n-sum[1]-lef[1]-rig[1]);
	}
	return 0;
}
```