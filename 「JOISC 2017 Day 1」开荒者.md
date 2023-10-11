# Description

开荒者要让一片长方形的平原长满草。这个长方形可视为 $R$ 行 $C$ 列的正方形网格，底平行于南北方向，高平行于东西方向。我们用 $(1,1)$ 表示网格的西北角，$(R,C)$ 表示网格的东南角。开始时有且只有 $n$ 个网格内长有野草，其余网格内既没有草也没有草籽。开荒者们可以控制这片平原上的风往东、西、南、北四个方向中的一个方向吹。

每年夏天，草会制造草籽。开荒者们会选定当年夏天的风向。所有草籽会被风扬起，并根据风向移动一格。来年春天，有草籽降落的网格就会有草萌发。假设草不会枯萎。

我们假设不会有草籽从平原外飘进平原内，飘出网格的草籽不会发芽。试求：让这片平原长满草最少需要几年。

$n\le 300,R,C\le 10^9$

# Solution

如果钦定了向上向下的风吹的次数，那么变成了每行求最多要左右吹几次

最靠左的野草以左只能是使用吹向左的风来覆盖，最靠右的野草以右只能通过吹向右的风来覆盖，其它的草之间的间隔所吹的左右风总数是一定的，那么一行上的所需次数是 $\max\{C-x_n+x_1-1,\max x_k-x_{k-1}-1\}$（设野草的横坐标为 $x_1,\dots x_m$）

那么存在一种暴力就是枚举向上以及向下的风吹的次数并对本质不同的行计算上面的表达式的值，但是冗余多得离谱

有两行因为列变换相遇且有一行撞到边界 以及 有两行撞到边界 的情况会出现本质不同行的集合不同的情况，那么可以对这 $\Theta(n^3)$ 中上下风进行枚举

这时又可以发现可以将所有风都变成向下吹并忽略边界，设吹了 $s$ 次，并在这 $R+s$ 行中选 $R$ 行求最小值来更新答案，进一步减少了冗余

对于任选 $R$ 行求最小值的一部分比较复杂：

先将所有野草从小到大排序，预处理 三元组 $f_{l,r}$ 表示在排好序的坐标中从第 $l$ 个到第 $r$ 个野草的所有列坐标长到一行上时最左边剩 $lef$ 个单位，右边剩 $rig$ 个单位，中间最大的间隔是 $Mid$ 个单位的 $(lef,Mid,rig)$ 

这些三元组可以简单使用 `std::multiset` 记录差，再用记录所有元素位置做到 $\Theta(n^2\log n)$ 的复杂度

列上草坐标发生变化的行标号是所有 $a_i+s+1$ 和 $a_i$ ，将其离散化即可得到所有本质不同的行坐标，根据上面的 $f_{l,r}$ 可以得到每段每段中的代价

尝试枚举每个本质不同的上界（也就是开始于一个特定的行）$u$，找到它到 $u+R-1$ 中所有行的代价的 $\max\{lef+rig,\max \Delta X\}$ 在得到答案

当然现在复杂度 $\Theta(n^4)$，不过这部分可以对 $lef,rig,\max \delta X$ 分别做滑动窗口来求最大值，时间复杂度降到了可过的 $\Theta(n^3)$




# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
#define pii pair<int,int> 
#define fir first
#define sec second
#define rep(i,a,b) for(int i=a;i<=b;++i)
template<typename T=int>inline T read(){
	T res=0,f=1; char k;
	while(!isdigit(k=getchar())) if(k=='-') f=-1;
	while(isdigit(k)) res=res*10+k-'0',k=getchar();
	return res*f;
}
template<typename T>inline void print(T x,int fl=1){
	if(x<0) putchar('-'),x=-x;
	if(x>=10) print(x/10,0);
	putchar(x%10+'0');
	if(fl) putchar('\n');
}
template<typename T>inline void ckmax(T &x,T y){x=x>y?x:y;}
template<typename T>inline void ckmin(T &x,T y){x=x<y?x:y;}
const int N=610;
int R,C,n,ans;
pair<int,int> a[N];
struct node{int x,y,z;}f[N][N],g[N];
int vals[N];
struct Deque{
	int id[N],val[N],head,tail;
	Deque(){
		head=1; tail=0;
		memset(id,0,sizeof(id));
		memset(val,0,sizeof(val));
	}
	inline void push(int cur,int v){
		while(head<=tail&&val[tail]<=v) --tail;
		id[++tail]=cur;
		val[tail]=v; 
	}
	inline void pop(int tar){
		while(head<=tail&&id[head]<=tar) ++head;
		return ;
	}
	inline int front_val(){return val[head];}
};

inline int calc(int L){
	int m=0;
	for(int i=1;i<=n;++i){
		vals[++m]=a[i].fir+L+1;
		vals[++m]=a[i].fir;
	}
	sort(vals+1,vals+m+1);
	m=unique(vals+1,vals+m+1)-vals-1;
	int l=1,r=1;
	for(int i=1;i<=m;++i){
		while(l<n&&a[l].fir+L<vals[i]) ++l;
		while(r<n&&a[r+1].fir<=vals[i]) ++r;
		g[i]=f[l][r];
	}
	Deque Lef,Rig,Mid;
	int Mn=C<<1,indic=1;
	for(int i=1;i<=m;++i){
		while(indic<=m&&vals[i]+R>vals[indic]){
			Lef.push(indic,g[indic].x);
			Mid.push(indic,g[indic].y);
			Rig.push(indic,g[indic].z);
			++indic;
		}
		if(indic>m) break;
		int now=max({Lef.front_val()+Rig.front_val(),Mid.front_val()});
		ckmin(Mn,now);
		Lef.pop(i); Mid.pop(i); Rig.pop(i);
	}
	return Mn;
}
signed main(){
	R=read(); C=read(); n=read();
	ans=R+C;
	for(int i=1;i<=n;++i) a[i].fir=read(),a[i].sec=read();
	sort(a+1,a+n+1);
	for(int i=1;i<=n;++i){
		set<int> s;
		for(int j=i;j<=n;++j){
			s.insert(a[j].sec);
			int cnt=0;
			for(auto t:s) vals[++cnt]=t;
			f[i][j].x=vals[1]-1;
			f[i][j].z=C-vals[cnt];
			for(int k=1;k<cnt;++k) ckmax(f[i][j].y,vals[k+1]-vals[k]-1);
		}//当然可以写个 multiset 避免每次搞，但是我显然懒得写了！
	}
	int lim=a[1].fir-1+R-a[n].fir;
	for(int i=1;i<n;++i) ckmax(lim,a[i+1].fir-a[i].fir-1);
	ckmin(ans,lim+calc(lim));
	for(int i=1;i<=n;++i){
		for(int j=1;j<=n;++j){
			int cur=R-a[i].fir+a[j].fir-1;
			if(cur>lim) ckmin(ans,cur+calc(cur));
			if(i>j){
				cur=a[i].fir-a[j].fir-1;
				if(cur>lim) ckmin(ans,cur+calc(cur));
			}
		}
	}
	print(ans);
	return 0;
}
```