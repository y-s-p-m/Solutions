# Description

在一棵大小为 $n$ 的树上面进行错误的点分治，即每次随机选点，然后代价是所有子树的大小和

求这种选点方式的代价和的期望

$n\le 30000$

# Solution

知道这题是分治 $+FFT$ 之后就想着推 $dp$ 式子，期望线性性根本没想到

所以这题第一步，期望线性性，全体的答案就是每个点点分树上面的子树期望大小和的答案

然后引入点分树的概念：点分治中选重心，然后上一次选的重心是下一次选的重心的父亲

然后就相对可做一点了

然后这题就是点分树的所有子树大小的期望和（这样感觉可做多了）

这个期望和就是其它点在当前点的子树内的概率

直接考虑两个点 $i$ 和 $j$ 

如果 $j$ 在点分树里面是 $i$ 的祖先，那么这种情况的概率就是$\frac 1 {dis(i,j)+1}$

所以我们要做的是统计每种距离下的点对的个数，然后求和

额……所以答案直接表示为：

$$\sum _ {i=1}^n \sum _ {j=1}^n \frac 1{dis(i,j)+1}$$

所以我们需要一种可以处理两点间距离的东西，它叫做点分治……

然后我们在点分治的过程中会做对距离做加法的那个部分发现那是一个卷积的形式（因为我们要统计的是方案）

所以我们发现点分治在做加法的时候是一个卷积的过程

所以 $fft$ 做掉

# Code

```cpp
const int N=30010;
struct node{
	double x,y;
	node(){}
	node(double xx,double yy){x=xx; y=yy; return ;}
	node operator+(node a){return node(x+a.x,y+a.y);}
	node operator-(node a){return node(x-a.x,y-a.y);}
	node operator*(node a){return node(x*a.x-y*a.y,x*a.y+y*a.x);}
	inline void clear(){x=y=0; return ;}
}A[N<<2];
const double pi=acos(-1);
int n,r[N<<2];
inline void FFT(node *f,int n,int opt){
	for(int i=0;i<n;++i) if(i<r[i]) swap(f[i],f[r[i]]);
	for(int p=2;p<=n;p<<=1){
		int len=p>>1; node tmp(cos(pi/len),opt*sin(pi/len));
		for(int k=0;k<n;k+=p){
			node buf(1,0);
			for(int l=k;l<k+len;++l){
				node tt=buf*f[l+len]; 
				f[len+l]=f[l]-tt; f[l]=f[l]+tt; 
				buf=buf*tmp;
			}
		}
	} if(opt==-1) for(int i=0;i<n;++i) f[i].x/=n;
	return;
}
struct edg{int to,nxt;}e[N<<1];
int head[N],cnt,app[N<<2],val[N],tot;
inline void add(int u,int v){e[++cnt].to=v; e[cnt].nxt=head[u]; return head[u]=cnt,void();}
inline void calc(int v){
	int la=0;
	for(int i=1;i<=tot;++i) la=max(la,val[i]);
	int n=1,m=la<<1; while(n<=m) n<<=1;
	for(int i=0;i<n;++i) r[i]=r[i>>1]>>1|((i&1)?(n>>1):0);
	for(int i=0;i<n;++i) A[i].clear();
	for(int i=1;i<=tot;++i) ++A[val[i]].x; 
	FFT(A,n,1); for(int i=0;i<n;++i) A[i]=A[i]*A[i]; FFT(A,n,-1);
	for(int i=0;i<n;++i) app[i]+=v*(int)(A[i].x+0.5);
	return ;
}
bool vis[N];
int rt,siz,minn,sz[N],num[N];
inline void getrt(int x,int fa){
	sz[x]=1; int tmp=0;
	for(int i=head[x];i;i=e[i].nxt){
		int t=e[i].to; if(vis[t]||t==fa) continue;
		getrt(t,x); sz[x]+=sz[t]; 
		tmp=max(tmp,sz[t]);
	}tmp=max(tmp,siz-sz[x]);
	if(tmp<minn) rt=x,minn=tmp;
	return ;
}
inline void getdis(int x,int dis,int fat){
	val[++tot]=dis;
	for(int i=head[x];i;i=e[i].nxt){
		int t=e[i].to;
		if(vis[t]||t==fat) continue;
		getdis(t,dis+1,x);
	} return ;
}
inline void dfs(int x){
	tot=0; vis[x]=1; getdis(x,0,0); calc(1); 
	for(int i=head[x];i;i=e[i].nxt){
		int t=e[i].to; if(vis[t]) continue;	
		tot=0; getdis(t,1,0); calc(-1); 
		minn=1e18+10; siz=sz[t]; getrt(t,0); dfs(rt);
	} return ;
}
signed main(){
	double ans=0;
	n=read();
	for(int i=1,u,v;i<n;++i) u=read()+1,v=read()+1,add(u,v),add(v,u);
	siz=n; minn=1e18+10; getrt(1,0); dfs(rt);
	for(int i=0;i<n;++i) ans+=(double)app[i]/(i+1);  
	printf("%.4Lf\n",ans);	
	return 0;
}
```