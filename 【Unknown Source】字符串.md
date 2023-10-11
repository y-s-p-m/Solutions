# Description

给定一个 $01$ 串，将其所有后缀插入 Trie 树中，将每个后缀的终止节点、Trie 树上 根和度数大于 $1$ 的点染黑

求得到的 Trie 树上存在祖孙关系且路径上不再有黑点的两个黑点之间的边形成的字符串的本质不同子串数量的和

$|S|\le 10^6$

# Solution

参考了 OneInDark 的讲解，JZM yyds！

题目中描述的 Trie 树就是反串 SAM，也即 SAM 是压缩 Trie

一个匪夷所思的想法是将上面所说的黑点放到正串 SAM 上刻画，容易发现如果 endpos 中包含 $n$ 或者 在 DAG 上有多条出边的点会被涂黑（考察 Trie 树上出边含义容易得到）

此时 Trie 上两个最近黑点组成的 **字符串集合** 就是将正串 DAG 的边反向，每个黑点在不经过其它黑点情况下到达另一个黑点的所有路径对应的字符串形成的集合

说是集合是因为每条路径对应的字符串都会在 Trie 出现 路径终点的 $len_x-len_{fa}$ 次

在 SAM 上走反向边等价与找到一个 endpos 反向加入字符，那么每个黑点找到每种路径长度下终点的 $\sum len-len_{fa}$ 再乘这段 $01$ 串的本质不同子串数 可以贡献给答案

根据上文描述的黑点定义，反向 DAG 上这些路径之间是无交的，路径总长不超过 DAG 的边数，暴力搜索路径配合在线求本质不同子串数即可

复杂度 $\Theta(n\Sigma)$

# Code

```cpp
const int N=2e6+10;
struct SAM{
	int pos[N],len[N],son[N][2],fa[N],tot,las;
	int sum=0;
	inline void init(){
		rep(i,1,tot){
			son[i][0]=son[i][1]=0;
			fa[i]=len[i]=0;
		}
		sum=0;
		tot=las=1;
	}
	inline void extend(int x,int id){
		int tmp=las,np=las=++tot;
		len[np]=len[tmp]+1; pos[np]=id;
		while(tmp&&!son[tmp][x]) son[tmp][x]=np,tmp=fa[tmp];
		if(!tmp){
			sum+=len[np]-len[fa[np]=1];
			return ;
		}
		int q=son[tmp][x];
		if(len[q]==len[tmp]+1){
			sum+=len[np]-len[fa[np]=q];
			return ;
		}
		int clone=++tot; len[clone]=len[tmp]+1;
		pos[clone]=pos[q]; fa[clone]=fa[q];
		son[clone][0]=son[q][0]; son[clone][1]=son[q][1];
		fa[q]=fa[np]=clone;
		while(son[tmp][x]==q) son[tmp][x]=clone,tmp=fa[tmp];
		sum+=len[np]-len[clone];
		return ;
	}
}S,calc;
char s[N];
int n,ans,coef[N];
bool mark[N];
vector<int> G[N];
signed main(){
	freopen("string.in","r",stdin); freopen("string.out","w",stdout);
	scanf("%s",s+1); n=strlen(s+1); 
	S.init();
	for(int i=1;i<=n;++i) S.extend(s[i]-'0',i);
	int tmp=S.las;
	while(tmp){
		mark[tmp]=1;
		tmp=S.fa[tmp];
	}
	for(int i=1;i<=S.tot;++i){
		rep(j,0,1) if(S.son[i][j]) G[S.son[i][j]].emplace_back(i);
		if(S.son[i][0]&&S.son[i][1]) mark[i]=1;
	}
	for(int i=2;i<=S.tot;++i) if(mark[i]){
		int len=0;
		function<void(int,int)>dfs=[&](const int x,const int cur){
			if(mark[x]){
				coef[cur]+=S.len[x]-S.len[S.fa[x]];
				if(x==1) coef[cur]++;
				ckmax(len,cur);
				return ;
			}
			for(auto t:G[x]) dfs(t,cur+1);	
		};
		for(int t:G[i]) dfs(t,1);
		calc.init();
		int ind=S.pos[i],cur=1;
		while(cur<=len){
			calc.extend(s[ind]-'0',ind);
			ans+=calc.sum*coef[cur]; coef[cur]=0;
			cur++;
			--ind;
		}
	}
	print(ans);
	return 0;
}
```