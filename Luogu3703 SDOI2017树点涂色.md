# Description

[Luogu3703](https://www.luogu.com.cn/problem/P3703)

有一棵树，点带颜色，初始颜色各不相同

定义路径价值为路径上面不同颜色的个数

请写出一种数据结构，支持以下几种操作：

$1.$ 更新根到某点的链上的颜色为一种新的

$2.$ 求一个点子树到中到根路径价值最大的点（求最大价值）

$3.$ 求两点之间的路径价值

# Solution

尝试使用强力数据结构 $\rm LCT$ 来模拟操作 $1$，那么不难得到每个 $\rm splay$ 上的点的颜色相同

因为有子树和树上路径操作，那么维护 $dfn$ 序是必要的，而子树操作可以直接进行区间加法，这时路径价值就需要减掉 $lca$ 后得到了

非常经典的题目，$\rm LCT$ 配合维护 $\rm dfn$ 序的数据结构时候展现出来了解决树上路径问题的巨大威力

博主在 2021-12-23 一定程度上修改了该文，表达对出题人的景仰并对初三时候不深度思考的学习生活的简单回忆

当我做到类似【ZJOI2018 历史】等 $\rm LCT$ 维护树链信息题目时总能回想起来这道题目，本题也总启发着我研究“究竟哪些贡献应当被删除，加入的贡献究竟属于谁”等细节

从各种意义上来说，本题是一道模板题，或许这样一个开始，是我奔向对细节极致追求并与天才一战的旅途吧

# Code

```cpp
const int N=1e5+10;
int fa[N],ls[N],rs[N],l[N],r[N],dfn[N],tot,anc[N][20];
int head[N],cnt,dep[N],n,m;
struct node{int to,nxt;}e[N<<1];
inline void add(int u,int v){
    e[++cnt].nxt=head[u]; e[cnt].to=v;
    return head[u]=cnt,void();
}
inline void dfs(int x,int fat){
    dep[x]=dep[fat]+1; dfn[++tot]=x; l[x]=tot; anc[x][0]=fat; fa[x]=fat;
    for(int i=1;anc[x][i-1];++i) anc[x][i]=anc[anc[x][i-1]][i-1];
    for(int i=head[x];i;i=e[i].nxt) if(e[i].to!=fat) dfs(e[i].to,x);
    r[x]=tot;
    return ;
}
inline int lca(int x,int y){
    if(dep[x]<dep[y]) swap(x,y);
    for(int i=19;i>=0;--i) if(dep[anc[x][i]]>=dep[y]) x=anc[x][i];
    if(x==y) return x;
    for(int i=19;i>=0;--i) if(anc[x][i]!=anc[y][i]) x=anc[x][i],y=anc[y][i];
    return anc[x][0];
}
struct tree{
    int maxx,fl,l,r;
    #define maxx(p) t[p].maxx
    #define l(p) t[p].l
    #define r(p) t[p].r
    #define fl(p) t[p].fl
}t[N<<2];
inline void push_up(int x){
    maxx(x)=max(maxx(x<<1),maxx(x<<1|1));
    return ;
}
inline void build(int p,int st,int ed){
    l(p)=st; r(p)=ed; if(st==ed) return maxx(p)=dep[dfn[st]],void();
    int mid=(st+ed)>>1; 
    build(p<<1,st,mid); build(p<<1|1,mid+1,ed); 
    return push_up(p);
}
inline void push_down(int p){
    if(fl(p)){
        fl(p<<1)+=fl(p); maxx(p<<1)+=fl(p);
        maxx(p<<1|1)+=fl(p); fl(p<<1|1)+=fl(p);
        t[p].fl=0;
    } 
    return ;
}
inline void update(int p,int st,int ed,int val){
    if(st<=l(p)&&r(p)<=ed) return fl(p)+=val,maxx(p)+=val,void();
    int mid=(l(p)+r(p))>>1; push_down(p);
    if(st<=mid) update(p<<1,st,ed,val);
    if(ed>mid) update(p<<1|1,st,ed,val);
    return push_up(p);
}
inline int query(int p,int st,int ed){
    if(st<=l(p)&&r(p)<=ed) return maxx(p);
    int mid=(l(p)+r(p))>>1; push_down(p);
    if(ed<=mid) return query(p<<1,st,ed);
    if(st>mid) return query(p<<1|1,st,ed);
    return max(query(p<<1|1,st,ed),query(p<<1,st,ed));
}
inline bool isroot(int x){return ls[fa[x]]!=x&&rs[fa[x]]!=x;}
inline void rotate(int x){
    int y=fa[x],z=fa[y];
    if(!isroot(y)) if(ls[z]==y) ls[z]=x; else rs[z]=x;
    if(ls[y]==x) ls[y]=rs[x],fa[rs[x]]=y,rs[x]=y;
    else rs[y]=ls[x],fa[ls[x]]=y,ls[x]=y;
    fa[x]=z; fa[y]=x;
    return ;
}
inline void splay(int x){
    // 没有下传标记操作自然不用爬到顶上 push_down 了！
    while(!isroot(x)){
        int y=fa[x],z=fa[y];
        if(!isroot(y)) rotate((ls[y]==x)^(ls[z]==y)?x:y);
        rotate(x);
    } return ;
}
inline int findroot(int x){while(ls[x]) x=ls[x]; return x;}
inline void access(int x){
    //rs[x] 在这里是深度大的点，而ls[x] 是深度小的
    for(int y=0;x;x=fa[y=x]){
        splay(x);
        if(rs[x]){
            int t=rs[x]; t=findroot(t);
            update(1,l[t],r[t],1);//把新的颜色添加到原来的链顶的所有子树中
        } 
        rs[x]=y;
        if(rs[x]){
            int t=rs[x]; t=findroot(t);
            update(1,l[t],r[t],-1);//已经被添加贡献的子树要避免重复添加
        }
    } return ;
}
signed main(){
    n=read(); m=read();
    for(int i=1,u,v;i<n;++i) u=read(),v=read(),add(u,v),add(v,u);
    dfs(1,0); build(1,1,n);
    while(m--){
        int opt=read();
        if(opt==1) access(read());
        if(opt==2){
            int x=read(),y=read(),lc=lca(x,y);
            x=l[x],y=l[y],lc=l[lc];
            printf("%lld\n",query(1,x,x)+query(1,y,y)-2*query(1,lc,lc)+1);
        }
        if(opt==3){
            int x=read();
            printf("%lld\n",query(1,l[x],r[x]));
        }
    }
    return 0;
}
```