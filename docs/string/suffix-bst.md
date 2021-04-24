## 定义

后缀之间的大小由字典序定义，后缀平衡树就是一个维护这些后缀顺序的平衡树，即 $T$ 的后缀平衡树是 $T$ 所有后缀的有序集合。后缀平衡树上的一个节点相当于原字符串的一个后缀。

特别地，后缀平衡树的中序遍历即为后缀数组。

## 构造过程

对长度为 $n$ 的字符串 $T$ 建立其后缀平衡树，考虑逆序将其后缀加入后缀平衡树。

记后缀平衡树维护的集合为 $X$，当前添加的后缀为 $S$，则添加下一个后缀就是向 $X$ 中加入 $cS$（亦可理解为后缀平衡树维护的字符串为 $S$，下一步往 $S$ 前加入一个字符）。这一操作其实就是向平衡树中插入节点。

这里使用期望树高为 $O(\log n)$ 的平衡树，例如替罪羊树或 Treap 等。

### 做法 1

插入时，暴力比较两个后缀之间的大小关系，从而判断之后是往哪一个子树添加。这样子，单次插入至多比较 $O(\log n)$ 次，单次比较的时间复杂度至多为 $O(n)$，一共 $O(n\log n)$。

一共会插入 $n$ 次，所以该做法的时间复杂度存在上界 $O(n^2 \log n)$。

### 做法 2

注意到 $cS$ 与 $S$ 的区别仅在于 $c$，且 $S$ 已经属于 $X$ 了，可以利用这一点来优化插入操作。

假设当前要比较 $cS$ 与 $A$ 两个字符串的大小，且 $A, S \in X$。每次比较时，首先比较两串的首字符。若首字符不等，则两串的大小关系就已经确定了；若首字符相等，那么就只需要判断去除首字符后，两字符串的大小关系，而两串去除首字符后都已经属于 $X$ 了，这时候可以借助平衡树来 $O(\log n)$ 来完成后续的比较。这样，单次插入的操作至多 $O(\log^2 n)$。

一共会插入 $n$ 次，所以该做法的时间复杂度存在上界 $O(n \log^2 n)$。

### 做法 3

根据做法 2，如果能够 $O(1)$ 判断平衡树中两个节点之间的大小关系，那么就可以在 $O(n \log n)$ 的时间内完成后缀平衡树的构造。

记 $val_i$ 表示节点 $i$ 的值。如果在建平衡树时，每个节点多维护一个标记 $tag_i$，使得若 $tag_i > tag_j \Leftrightarrow val_i > val_j$，那么就可以根据 $tag_i$ 的大小 $O(1)$ 判断平衡树中两个节点的大小。

不妨令平衡树中每个节点对应一个实数区间，令根节点对应 $(0, 1)$。对于节点 $i$，记其对应的实数区间为 $(l, r)$，则 $tag_i = \frac{l + r}{2}$，其左子树对应实数区间 $(l, tag_i)$，其右子树对应实数区间 $(tag_i, r)$。易证 $tag_i$ 满足上述要求。

由于使用了期望树高为 $O(\log n)$ 的平衡树，所以精度是有一定保证的。实际实现时也可以用一个较大的整数区间来做，例如让根对应 $[0, 2^{62}]$。对于节点 $i$，记其对应区间为对 $[l, r]$，则 $tag_i = \lfloor \frac{l + r}{2} \rfloor$，其左子树对应区间 $[l, tag_i - 1]$，其右子树对应区间 $[tag_i + 1, r]$。

### 做法 4

其实可以先构建出后缀数组，然后再根据后缀数组构建后缀平衡树。这样做的复杂度瓶颈在于后缀数组的构建复杂度或者后缀平衡树一次性插入 $n$ 个元素的复杂度。

## 后缀平衡树的优点

- 后缀平衡树的思路比较清晰，相比后缀自动机等后缀结构更好理解，会写平衡树就能写。
- 后缀平衡树的复杂度不依赖于字符集的大小
- 后缀平衡树支持在结尾删除一个字符
- 如果使用支持可持久化的平衡树，那么后缀平衡树也能可持久化

## 例题

### [P3809【模板】后缀排序](https://www.luogu.com.cn/problem/P3809)

建出后缀平衡树之后，通过中序遍历得到后缀数组。

??? note "SGT版本的参考代码"
    ```cpp
    #include <bits/stdc++.h>
    using namespace std;
    
    using ll = long long;
    const int N = 1e6 + 5;
    const ll INF = 1LL << 60;
    
    int n, m, sa[N];
    char t[N];
    
    // SuffixBST(SGT Ver)
    
    // 以i开始的后缀，对应节点的编号为i。
    const double alpha = 0.75;
    int root;
    int siz[N], L[N], R[N];
    ll tag[N];
    int buffer_size, buffer[N];
    
    // O(1) x, y对应后缀的大小
    // 即比较平衡树中x, y两个节点的大小
    bool cmp(int x, int y) {
      // 先比较首字符
      if (t[x] != t[y]) return t[x] < t[y];
      // 否则去除首字符，再比较
      return tag[x + 1] < tag[y + 1];
    }
    
    void init() { root = 0; }
    
    void new_node(int& rt, int p, ll lv, ll rv) {
      rt = p;
      siz[rt] = 1;
      tag[rt] = (lv + rv) >> 1LL;
      L[rt] = R[rt] = 0;
    }
    
    void push_up(int x) {
      if (!x) return;
      siz[x] = siz[L[x]] + 1 + siz[R[x]];
    }
    
    bool balance(int rt) { return alpha * siz[rt] >= max(siz[L[rt]], siz[R[rt]]); }
    
    void flatten(int rt) {
      if (!rt) return;
      flatten(L[rt]);
      buffer[++buffer_size] = rt;
      flatten(R[rt]);
    }
    
    void build(int& rt, int l, int r, ll lv, ll rv) {
      if (l > r) {
        rt = 0;
        return;
      }
      int mid = (l + r) >> 1;
      ll mv = (lv + rv) >> 1LL;
    
      rt = buffer[mid];
      tag[rt] = mv;
      build(L[rt], l, mid - 1, lv, mv - 1);
      build(R[rt], mid + 1, r, mv + 1, rv);
      push_up(rt);
    }
    
    void rebuild(int& rt, ll lv, ll rv) {
      buffer_size = 0;
      flatten(rt);
      build(rt, 1, buffer_size, lv, rv);
    }
    
    void insert(int& rt, int p, ll lv, ll rv) {
      if (!rt) {
        new_node(rt, p, lv, rv);
        return;
      }
    
      ll mv = (lv + rv) >> 1LL;
      if (cmp(p, rt))
        insert(L[rt], p, lv, mv - 1);
      else
        insert(R[rt], p, mv + 1, rv);
    
      push_up(rt);
      if (!balance(rt)) rebuild(rt, lv, rv);
    }
    
    void inorder(int rt) {
      if (!rt) return;
      inorder(L[rt]);
      sa[++m] = rt;
      inorder(R[rt]);
    }
    
    void solve(int Case) {
      scanf("%s", t + 1);
      n = strlen(t + 1);
    
      init();
      for (int i = n; i >= 1; --i) {
        insert(root, i, 0, INF);
      }
    
      // 后缀平衡树的中序遍历即为后缀数组
      m = 0;
      inorder(root);
    
      for (int i = 1; i <= n; ++i) printf("%d ", sa[i]);
      printf("\n");
    
      // tag[sa[i]]应为升序
      // for (int i = 1; i <= n; ++i)
      //     printf("%lld ", tag[sa[i]]);
      // printf("\n");
    }
    
    int main() {
      int T = 1;
      // cin >> T;
      for (int i = 1; i <= T; ++i) solve(i);
      return 0;
    }
    ```

## 参考资料

[^1]: 陈立杰 -《重量平衡树和后缀平衡树在信息学奥赛中的应用》