<!--?title Lowest Common Ancestor - O(sqrt(N)) and O(log N) with O(N) preprocessing -->

# Lowest Common Ancestor - O(sqrt(N)) and O(log N) with O(N) preprocessing

Given a tree $G$. Given queries of the form ($v_1$, $v_2$), for each query you need to find the lowest common ancestor (or least common ancestor), i.e. a vertex $v$ that lies on the path from the root to $v_1$ and the path from the root to $v_2$, and the vertex should be the lowest. In other words, the desired vertex $v$ is the most bottom ancestor of $v_1$ and $v_2$. It is obvious that their lowest common ancestor lies on a shortest path from $v_1$ and $v_2$. Also, if $v_1$ is the ancestor of $v_2$, $v_1$ is the lowest common ancestor.

### The Idea of the Algorithm

Before answering the queries, we need to do *preprocessing*. Run DFS from the root using preorder traversal and it will build a list of visit order of the vertices (current vertex is added to the list at the entrance to the vertex and after return from its child (children)). It is clear that the size of this list will be $O(N)$.








### Implementation

The implementation of described LCA algorithm is as follows:

```
typedef vector < vector<int> > graph;
typedef vector<int>::const_iterator const_graph_iter;


vector<int> lca_h, lca_dfs_list, lca_first, lca_tree;
vector<char> lca_dfs_used;

void lca_dfs (const graph & g, int v, int h = 1)
{
    lca_dfs_used[v] = true;
    lca_h[v] = h;
    lca_dfs_list.push_back (v);
    for (const_graph_iter i = g[v].begin(); i != g[v].end(); ++i)
        if (!lca_dfs_used[*i])
        {
            lca_dfs (g, *i, h+1);
            lca_dfs_list.push_back (v);
        }
}

void lca_build_tree (int i, int l, int r)
{
    if (l == r)
        lca_tree[i] = lca_dfs_list[l];
    else
    {
        int m = (l + r) >> 1;
        lca_build_tree (i+i, l, m);
        lca_build_tree (i+i+1, m+1, r);
        if (lca_h[lca_tree[i+i]] < lca_h[lca_tree[i+i+1]])
            lca_tree[i] = lca_tree[i+i];
        else
            lca_tree[i] = lca_tree[i+i+1];
    }
}

void lca_prepare (const graph & g, int root)
{
    int n = (int) g.size();
    lca_h.resize (n);
    lca_dfs_list.reserve (n*2);
    lca_dfs_used.assign (n, 0);

    lca_dfs (g, root);

    int m = (int) lca_dfs_list.size();
    lca_tree.assign (lca_dfs_list.size() * 4 + 1, -1);
    lca_build_tree (1, 0, m-1);

    lca_first.assign (n, -1);
    for (int i = 0; i < m; ++i)
    {
        int v = lca_dfs_list[i];
        if (lca_first[v] == -1)
            lca_first[v] = i;
    }
}

int lca_tree_min (int i, int sl, int sr, int l, int r)
{
    if (sl == l && sr == r)
        return lca_tree[i];
    int sm = (sl + sr) >> 1;
    if (r <= sm)
        return lca_tree_min (i+i, sl, sm, l, r);
    if (l > sm)
        return lca_tree_min (i+i+1, sm+1, sr, l, r);
    int ans1 = lca_tree_min (i+i, sl, sm, l, sm);
    int ans2 = lca_tree_min (i+i+1, sm+1, sr, sm+1, r);
    return lca_h[ans1] < lca_h[ans2] ? ans1 : ans2;
}

int lca (int a, int b)
{
    int left = lca_first[a],
        right = lca_first[b];
    if (left > right)  swap (left, right);
    return lca_tree_min (1, 0, (int)lca_dfs_list.size()-1, left, right);
}

int main()
{
    graph g;
    int root;
    ... чтение графа ...

    lca_prepare (g, root);

    for (;;)
    {
        int v1, v2; // поступил запрос
        int v = lca (v1, v2); // ответ на запрос
    }
}
```



Let, $f$ be some _reversible_ function and $A$ be an array of integers of length $N$.

Fenwick tree is a data structure which:

* calculates the value of function $f$ in the given range $[l; r]$ (i.e. $f(A_l, A_{l+1}, \dots, A_r)$) in $O(lg\ n)$ time;
* updates the value of an element of $A$ in $O(lg\ n)$ time;
* requires $O(N)$ memory, or in other words, exactly the same memory required for $A$;
* is easy to use and code, especially, in the case of multidimensional arrays.

> Fenwick tree is also called Binary Indexed Tree.

The most common application of Fenwick tree is _calculating the sum of a range_ (i.e. $f(A_1, A_2, \dots, A_k) = A_1 + A_2 + \dots + A_k$).

Fenwick tree was first described in the paper titled "A new data structure for cumulative frequency tables" (Peter M. Fenwick, 1994).

### Description

For the sake of simplicity, we will assume that function $f$ is just *sum function*.

Given an array of integers $A[0 \dots N-1]$. Fenwick tree is an array $T[0 \dots N-1]$, where each of its elements is equal to the sum of elements of $A$ in some range $[g(i); i]$:

$$T_i = \sum_{j = g(i)}^{i}{A_j}$$

where $g$ is some function that satisfies $(g(i) \le i)$, and we will define it a bit later.

> **Note:** Fenwick tree presented here **does support** 0-based indexing (in case you were told that it does not support it).

Now we can write pseudo-code for the two operations mentioned above &mdash; get the sum of elements of $A$ in range $[0; r]$ and update some element $A_i$:

```python
def sum (int r):
    res = 0
    while (r >= 0):
        res += t[r]
        r = g(r) - 1
    return res

def inc (int i, int delta):
    for all j, where g(j) <= i <= j
        t[j] += delta
```

The function `sum` works as follows:

1. firstly, it adds the sum of the range $[g(r); r]$ (i.e. $T[r]$) to the `result` ;
2. then, it "jumps" to the range $[g(g(r)-1); g(r)-1]$, and adds this range's sum to the `result` ;
3. and so on, till it "jumps" from $[0; g(g( \dots g(r)-1 \dots -1)-1)]$ to $[g(-1); -1]$; that is where the `sum` stops jumping.

The function `inc` works with the same analogy, but "jumps" in the direction of increasing indices:

1. sums of the ranges $[g(j); j]$ that satisfy the condition $g(j) \le i \le j$ are increased by `delta` , that is `t[j] += delta` .

It is obvious that complexity of both `sum` and `upd` do depend on the function $g$. We will define the function $g$ in such a way, so that, both of the operations will have logarithmic complexity $O(lg N)$.

**Definition of $g(i)$.** Let us consider the least significant digit of $i$ in binary. If this digit is $0$, then let $g(i) = i$. Otherwise, if we examine digits of $i$ in binary (in decreasing order of significance), then $i$ should end with one or more $1$'s. We convert these $1$'s to $0$'s and assign the new number as the value of $g(i)$.

There exists a trivial solution for the non-trivial operation described above:

```
g(i) = i & (i+1)
```

where `&` is logical AND operator. It is not hard to convince yourself that this solution does the same thing as the operation described above.

Now, we need to find a way to find all such $j$'s, so that, *g(j) <= i <= j*.

It is easy to see that we can find all such $j$'s, by starting with $i$ and replacing the least significant one of all $0$'s with a $1$. For example, for $i = 10$ we have:

```
j = 10, binary 0001010
j = 11, binary 0001011
j = 15, binary 0001111
j = 31, binary 0011111
j = 63, binary 0111111
...
```

Not surprisingly, there also exists a simple way to do the above operation:

```
h(j) = j | (j+1)
```

where `|` is the logical OR operator.

### Implementation: finding sum in one-dimensional array

```cpp
struct FenwickTree {
    vector<int> bit; // binary indexed tree
    int n;

    void init(int n) {
        this->n = n;
        bit.assign(n, 0);
    }
    int sum(int r) {
        int ret = 0;
        for (; r >= 0; r = (r & (r+1)) - 1)
            ret += bit[r];
        return ret;
    }
    void add(int idx, int delta) {
        for (; idx < n; idx = idx | (idx+1))
            bit[idx] += delta;
    }
    int sum(int l, int r) {
        return sum(r) - sum(l-1);
    }
    void init(vector<int> a) {
        init(a.size());
        for (size_t i = 0; i < a.size(); i++)
            add(i, a[i]);
    }
};
```

### Implementation: finding minimum of $[0; r]$ in one-dimensional array

It is obvious that there is no way of finding minimum of range $[l; r]$ using Fenwick tree, as Fenwick tree can only answer the queries of type [0; r]. Additionally, each time a value is `update`'d, new value should be smaller than the current value (because, the $min$ function is not reversible). These, of course, are significant limitations.

```cpp
struct FenwickTreeMin {
    vector<int> bit;
    int n;
    const int INF = (int)1e9;
    void init (int n) {
        this->n = n;
        bit.assign (n, INF);
    }
    int getmin (int r) {
        int ret = INF;
        for (; r >= 0; r = (r & (r+1)) - 1)
            ret = min(ret, bit[r]);
        return ret;
    }
    void update (int idx, int val) {
        for (; idx < n; idx = idx | (idx+1))
            bit[idx] = min(bit[idx], val);
    }
    void init (vector<int> a) {
        init (a.size());
        for (size_t i = 0; i < a.size(); i++)
            update(i, a[i]);
    }
};
```

### Implementation: finding sum in two-dimensional array

As claimed before, it is easy to implement Fenwick Tree for multidimensional array.

```cpp
struct FenwickTree2D {
    vector <vector <int> > bit;
    int n, m;
    // init(...) { ... }
    int sum (int x, int y) {
        int ret = 0;
        for (int i = x; i >= 0; i = (i & (i+1)) - 1)
            for (int j = y; j >= 0; j = (j & (j+1)) - 1)
                ret += bit[i][j];
        return ret;
    }
    void add(int x, int y, int delta) {
        for (int i = x; i < n; i = i | (i+1))
            for (int j = y; j < m; j = j | (j+1))
                bit[i][j] += delta;
    }
};
```

## Practice Problems

* [UVA 12086 - Potentiometers](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=3238)
* [LOJ 1112 - Curious Robin Hood](http://www.lightoj.com/volume_showproblem.php?problem=1112)
* [LOJ 1266 - Points in Rectangle](http://www.lightoj.com/volume_showproblem.php?problem=1266 "2D Fenwick Tree")
* [Codechef - SPREAD](http://www.codechef.com/problems/SPREAD)
* [SPOJ - CTRICK](http://www.spoj.com/problems/CTRICK/)
* [SPOJ - MATSUM](http://www.spoj.com/problems/MATSUM/)
* [SPOJ - DQUERY](http://www.spoj.com/problems/DQUERY/)
* [SPOJ - NKTEAM](http://www.spoj.com/problems/NKTEAM/)
* [SPOJ - YODANESS](http://www.spoj.com/problems/YODANESS/)

### Other sources

* [Fenwick tree on Wikipedia](http://en.wikipedia.org/wiki/Fenwick_tree)  
* [Binary indexed trees tutorial on TopCoder](https://www.topcoder.com/community/data-science/data-science-tutorials/binary-indexed-trees/)


Translated by [sylap97](http://codeforces.com/profile/sylap97)
