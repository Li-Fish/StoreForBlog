---
title: 数位DP（模板）
date: 2017-04-24 16:53:28
categories: [ACM, DP, 数位DP]
tags:
---
# 推荐博客：
http://zyk1997.github.io/2015/03/20/ShuWeiDP/

-----------------------------
# 模板题：
http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1009

http://acm.hdu.edu.cn/showproblem.php?pid=3555

http://acm.hdu.edu.cn/showproblem.php?pid=2089

----------------------------------------
# 模板：
```cpp
//pos当前的位置，pre上一位的数字，ok现在是否已满足题目的条件，bound是否受到限制。
LL dfs(int pos,int pre,int ok,int bound){
	//如果递归到个位，就到达了边界并返回。
    if(pos < 0)
        return ok;
        
    //如果不受限制，那么可以查看dp数组当前状态是否计算过。
    if(!bound && dp[x][pre][ok]!=-1)
        return dp[x][pre][ok];
       
    //根据是否受限制，确定需要枚举的数。
    int last=bound? num[x]:9;
    
    LL rst=0;
    
    for(int i=0;i<=last;i++)
	    //成立条件根据题目要求，如果当前枚举的是最后应该是，并且受限制，那么下一位也受限制。
        re+=dfs(x-1, i, ok || (成立条件), bound && (i==last));
    
    //如果不受限制，那么更新dp数组。
    if(!bound) f[x][pre][ok]=re;
    return re;
} 

LL solve(LL n){
    int len=0;
    while(n){
        num[len++]=n%10;
        n/=10;
    }
    return dfs(len-1,0,0,1);
}
```
