# Description

[link](https://www.luogu.com.cn/problem/P4762)

给定一个字符串，求通过以下两种方式最少用几次就可以构造出来它

$1.$ 在当前串尾添加任意一个字符

$2.$ 在串开头或末尾加一个当前的逆串

# Solution

首先发现 $2$ 操作就是可以把一个偶数长度的回文串通过较少次构造出来

对这个给定的串建 $PAM$ 吧，顺便求一下 $trans$ 指针

然后在 $PAM$ 上面 $dp$

直接从偶根 $bfs$ （像 $AC$ 自动机构建 $fail$ 指针一样）

设 $f_i$ 为构造出来当前的串至少需要多少次操作

然后 $ans=\min\limits _ {i=2} ^{tot} n-len_i+f_i$ 

转移分为以下两种：

$1.$ 当前串的长度$len_i$

$2.$ 从自动机上面直接转移 $f_i=min(f_i,f_{fail_i}+1)$

$3.$ 从 $trans$ 指针上面转移

$$f_i=min(f_i,f_{trans_i} +1 +\frac {len_i}{2}-len_{trans_i})$$

（补齐前缀然后复制）

一坑点：$ans$ 记得初始最大值设成 $n$

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=1e5+10;
	int ans;
	struct PAM{
		int t[N],fail[N],ch[N][4],tot,las,n,len[N],f[N];
		char s[N];
		inline void init()
		{			
			for(int i=0,j;i<=tot;++i) 
			{
				for(j=0;j<4;++j) ch[i][j]=0;
				t[i]=0; f[i]=0; fail[i]=0; len[i]=0;
			}
			ans=2e9+10; fail[0]=fail[1]=tot=1; len[1]=-1; las=0;
			return ;
		}
		inline void extend(int i)
		{
			int c=s[i]-'A',p=las;
			while(s[i-len[p]-1]!=s[i]) p=fail[p];
			if(ch[p][c]) return las=ch[p][c],void();
			int y=++tot,x=fail[p];
			while(s[i-len[x]-1]!=s[i]) x=fail[x];
			len[y]=len[p]+2; fail[y]=ch[x][c]; las=ch[p][c]=y; 
			if(len[y]<=2) return t[y]=fail[y],void();
			int q=t[p]; while(len[q]*2+4>len[y]||s[i-len[q]-1]!=s[i]) q=fail[q];
			t[y]=ch[q][c];
			return ;
		}
		inline void work()
		{
			queue<int> q; for(int i=2;i<=tot;++i) f[i]=len[i];
			for(int i=0;i<4;++i) if(ch[0][i]) q.push(ch[0][i]);
			while(!q.empty())
			{
				int fr=q.front(); q.pop();
				f[fr]=min(f[fr],f[t[fr]]+1+len[fr]/2-len[t[fr]]);
				ans=min(ans,n-len[fr]+f[fr]); 
				for(int i=0;i<4;++i)
				{
					if(!ch[fr][i]) continue;
					f[ch[fr][i]]=min(f[ch[fr][i]],f[fr]+1); q.push(ch[fr][i]);
				}
			} return ;
		}
	};
	PAM a;
	inline char c(char p)
	{
		if(p=='A') return 'A';
		if(p=='G') return 'B';
		if(p=='C') return 'C';
		if(p=='T') return 'D';
		else return 0;
	}
	inline void work()
	{
		scanf("%s",a.s+1); a.init(); a.s[0]='#';
		int len=strlen(a.s+1); a.n=len; 
		for(int i=1;i<=len;++i) a.s[i]=c(a.s[i]),a.extend(i);
		a.work(); ans=min(ans,len);
		printf("%d\n",ans); 
		return ;
	}
	signed main()
	{
		int T=read(); while(T--) work();
		return 0;
	}
}
signed main(){return yspm::main();} 
```
