---
title: CodeForce727E - Games on a CD（字符串双hash模板）
date: 2017-11-28 11:09:40
categories: [ACM, 字符串, 字符串Hash]
tags:
---
# 题目链接：

http://codeforces.com/problemset/problem/727/E

# 题目大意：

给出一个长度为 $N \times k$  的字符串，可以将其看成一个环首尾相接。问这个字符串是否可以由下面给出的  g 个字符串组成，保证字符串不同并且**长度均为 k**，每个字符串只能用一次。

$1\le N, k \le 10^5$    $n \cdot k, g \cdot k \le 2 \times 10^6$

# 题目分析：

这题只要会字符串 Hash 就会做了，首先把原串复制一遍放到结尾。因为每个串的长度都为 k 所以从 0 到 k 枚举起点，然后不断的跳到后面 k 个位置，判断 g 个字符串是否匹配上这一段子串并且没有被使用，如果找到一组解就输出答案。

复杂度$O(N\log n)$这里把 Hash 后的 pair 映射成整数，多了一个 log。




# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

const int MAX = 200000 + 100;

//--------------------双哈希---------------------------------------
const int MOD[] = {1000000007, 1000000009};
const int P = 10000017;

vector<ll> hashValue[2][MAX];
string str[MAX];
ll powNum[2][MAX];

//对一个字符串进行 hash 并保留前缀和
inline void init(int n) {
    for (int k = 0; k < 2; k++) {
        hashValue[k][n].clear();
        hashValue[k][n].push_back(0);
        for (int i = 1; i <= str[n].size(); i++) {
            hashValue[k][n].push_back((hashValue[k][n][i - 1] * P + str[n][i - 1]) % MOD[k]);
        }
    }
}

//获得某个字符串的任一子串的 hash 值
inline pair<int, int> getHash(int n, int l = 0, int r = -1) {
    if (r == -1) r = str[n].size() - 1;
    l++, r++;
    vector<int> ans;
    for (int k = 0; k < 2; k++) {
        ans.push_back((int) (((hashValue[k][n][r] - hashValue[k][n][l - 1] * powNum[k][r - l + 1]) % MOD[k] + MOD[k]) %
                             MOD[k]));
    }
    return make_pair(ans[0], ans[1]);
}

//初始化幂，计算过程中使用
void init_pow() {
    for (int k = 0; k < 2; k++) {
        powNum[k][0] = 1;
        for (int i = 1; i < MAX; i++) {
            powNum[k][i] = powNum[k][i - 1] * P % MOD[k];
        }
    }
}

//将 hash 得到的 pair 映射成整数
vector<int> store;
map<pair<int, int>, int> hashName;

inline int getID(pair<int, int> n, int index = 0) {
    if (hashName.count(n)) return hashName[n];
    store.push_back(index);
    return hashName[n] = store.size() - 1;
}
//--------------------双哈希------------------------------------------------

bool ok[MAX];

int main() {
    init_pow();

    ios::sync_with_stdio(false);

    int n, k, g;
    cin >> n >> k;
    cin >> str[0];
    str[0] = str[0] + str[0];
    init(0);

    cin >> g;
    for (int i = 1; i <= g; i++) {
        cin >> str[i];
        init(i);
        getID(getHash(i), i);
    }

    //枚举起点
    for (int st = 0; st < k; st++) {
        vector<int> ans;
        bool flag = true;
        //计算从起点到终点中，是否可以有 g 个字符串组成
        for (int i = st; i < st + str[0].size() / 2; i += k) {
            pair<int, int> v = getHash(0, i + 1, i + k);
            if (!hashName.count(v)) {
                flag = false;
                break;
            }
            int idx = getID(v);
            if (ok[idx]) {
                flag = false;
                break;
            }
            ans.push_back(idx);
            ok[idx] = true;
        }
        if (flag) {
            printf("YES\n");
            for (int i = 0; i < ans.size(); i++) {
                printf("%d ", store[ans[i]]);
            }
            return 0;
        }
        for (int i = 0; i < ans.size(); i++) {
            ok[ans[i]] = false;
        }
    }

    printf("NO\n");
}
```
# 解题过程：

Hash 过程的代码完全参照[卿学姐的视频](https://www.bilibili.com/video/av7230433/?from=search&seid=6735421237600758359)写的，非常感谢！

[PPT链接](http://pan.baidu.com/s/1c27PEF2)

