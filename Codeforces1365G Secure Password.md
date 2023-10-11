# Description

现在存在长度为 $n$ 的数组 $\{a\}$，定义 $\{p\}$ 满足 $\displaystyle p_i={\rm{OR}_{j\neq i}} \ a_j$

现在给定 $n\le 1000$，每次可以给定一个非空的下标集合 $i_1,i_2,\dots i_k$，$k$ 任意，返回这个集合里面所有数的或和

要求在不给定 $\{a\}$ 的情况下使用不超过 $13$ 次操作求出来 $\{p\}$

# Solution

非常直观的想法就是将 $[1,n]$ **二进制分组**，枚举二进制位，每次询问这位是 $0/1$ 的位的数字的异或和

这样子是 $20$ 次询问，考虑优化

上面的做法的局限性在于 $0,1$ 的地位是不相等的，例如 $1$ 是 $3$ 的子集，那么对于第一个二进制位的询问 $3$ 和 对第二个二进制位询问 $3$ 的重复构成了冗余

**“剥离包含关系”**，给 $[1,n]$ 重新标号，使用 $13$ 位二进制数中 $\text{popcount}$ 为 $6$ 的那些赋给 $[1,n]$，再次枚举二进制位的时候只用询问这位为 $0/1$ 的一种即可

每个 $j\neq i$ 的 $j$ 会在 $id_j\neq id_i$ 的二进制位里面被统计进答案

# Code

```cpp
const int N=1010;
int id[N],ans[N],n;
signed main(){
    int S=1<<13,cnt=0; --S;
    for(int i=0;i<S;++i){
        if(__builtin_popcount(i)==6) id[++cnt]=i;
        if(cnt>=1000) break;
    }
    n=read();
    rep(i,1,13){
        vector<int> qu;
        rep(j,1,n) if(id[j]>>(i-1)&1) qu.push_back(j);
        if(!qu.size()) continue;
        printf("? "); print(qu.size());
        for(auto t:qu) print(t);
        putchar('\n');
        fflush(stdout);
        int res=read();
        rep(j,1,n) if(!(id[j]>>(i-1)&1)) ans[j]|=res;
    }
    printf("! ");
    rep(i,1,n) print(ans[i]);
    putchar('\n');
    fflush(stdout);
    return 0;
}
```