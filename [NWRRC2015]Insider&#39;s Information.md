# Description

对于 $[1, n]$ 的排列，给 $m$ 个限制，第 $i$ 个限制描述 $b_i$ 出现在 $a_i$，$c_i$ 之间。保证存在一个排列满足所有限制。

构造一个 $n$ 的排列至少满足 $\lceil \frac m2 \rceil$ 个限制。

$n,m\le 10^5$

# Solution

将 $a_i,c_i$ 向 $b_i$ 连边但是只给 $b_i$ 的出度增加 $1$，然后跑拓扑排序

由于题目中保证了存在一个排列满足所有限制，那么这张图不会出环

假设 $a_i$ 是这个三元组中第一个出队的元素，那么可能的拓扑序的相对顺序是 $(a_i,b_i,c_i),(a_i,c_i,b_i)$

在拓扑排序的过程中从两端向中间填数字，维护两个指针表示填到了哪里，同时可以维护每个数字的位置

当填入限制的第二个数字时来计算是否要满足这个限制：

对于一个出队的元素，找到其作为 $c_i$ 的对应 $a_i$ 是在左指针以左还是右指针以右

如果在左指针以左那么要满足该限制就是将当前元素填到右指针对应位置（此时该限制 $b$ 元素将填在两指针之间，也就一定在 $a,c$ 之间），在右指针以右类似

作为 $b_i$ 的限制要满足条件也可以根据上述原理找到对应需求

那么当前元素填到哪里取决于两种限制哪种更多，由于每次都会满足待选限制中较多的部分，那么总起来满足的限制也不会少于 $\frac m2$ 个

时间复杂度 $\Theta(n)$

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define rep(i,a,b) for(int i=a;i<=b;++i)
const int N=1e5+10;
inline int read(){int x; scanf("%d",&x); return x;}
inline void print(int x){printf("%d ",x);}
int n,m,a[N],b[N],c[N],deg[N];
int pos[N],ans[N];
vector<int> G[N],s[N];
int main(){
	n=read(); m=read();
	for(int i=1;i<=m;++i){
		a[i]=read(),b[i]=read(),c[i]=read();
		G[a[i]].emplace_back(i);
		G[c[i]].emplace_back(i);
		s[b[i]].emplace_back(i);
		deg[b[i]]++;
	}
	queue<int> q;
	rep(i,1,n) if(!deg[i]) q.push(i);
	int indl=1,indr=n;
	while(q.size()){
		int fr=q.front(); q.pop();
		int cnt[2]={};
		for(auto id:G[fr]){
			if(pos[a[id]]||pos[c[id]]){
				cnt[pos[a[id]^c[id]^fr]<indl]++;
			}else{
				--deg[b[id]];
				if(!deg[b[id]]) q.push(b[id]);
			}
		}
		for(auto id:s[fr]){
			if(!(pos[a[id]]&&pos[c[id]])){
				cnt[(pos[a[id]]<pos[c[id]])>indr]++;
			}
		}
		if(cnt[0]<cnt[1]) pos[fr]=indr--;
		else pos[fr]=indl++;
		ans[pos[fr]]=fr;	
	}
	rep(i,1,n) print(ans[i]); putchar('\n');
	return 0;
}
```