---
layout: post
author: leventegego
title: Non-academic problem (Bridge Finding)
date: 2024-07-04 12:00:00 +0100
summary: 'Solving CF 1986F with bridge finding.'
categories: [codeforces, graph-theory]
keywords: codeforces, dfs, graph-theory, trees
tags:
 - codeforces
 - dfs
 - graph-theory
 - trees
math: true
---

In [CF 1986F: Non-academic problem](https://codeforces.com/contest/1986/problem/F) we are given a simple undirected graph. The graph is connected, meaning that every vertex can be reached from every other vertex. Our task is to ruin this property, as in "minimize the number of pairs $1 \leq u \lt v \leq n$, between which there exists a path". To do this, we are allowed to choose and remove a single edge.

## Understanding the Problem

By removing an edge, the graph can be broken into two components. The number of connected pairs we would eliminate is the product of the number of vertices in the components. However, not all edges can split the graph in this way. An edge that connects two, otherwise disconnected subgraphs is called a _bridge_. Bridges in the graph below are marked in red.

![bridge](/assets/posts/2024-08-04-non-academic-problem/bridges.svg)


## Solution

Notice that an edge is a bridge if and only if it does not belong to any cycle. If it is part of a cycle, then removing it does not break the graph because all paths that pass through the edge can be rerouted along the cycle. If it is not in any cycle, then there is no path that connects its two endpoints while avoiding the edge, and therefore removing it makes those two points disconnected.

Cycles in a graph can be found via depth-first search. We can extend this algorithm to enumerate bridges. The DFS tree has the special property, that for every non-tree edge $(u, v)$ is a back edge, meaning that $v$ is an ancestor of $u$. Since every cycle contains at least one non tree edge, if an edge $(u, v)$ is contained in a cycle, then by searching $v$'s subtree, a back edge that connects one of the descendants of $v$ to one of its ancestors can be found.

We can keep count of such back edges, and when a vertex is finished, we can determine whether the edge between it and its parent is a bridge.

![bridge](/assets/posts/2024-08-04-non-academic-problem/back-edges.svg)

Here, you can see a possible search tree starting from node $1$. The dashed lines mark back edges.

To find possible solutions, we can run a second DFS to count the vertices in the subtree of each bridge. Let the number of vertices be $n$ in the whole graph, and $k$ in the current subtree. Then the graph has $\binom{n}{2}$ paths in total, and we can break $k \cdot (n - k)$ of those by removing the current edge.

The complexity of this method is $O(n + m)$ since we only visit the vertices and edges once in each DFS.

## Implementation

The below implementation involves a recursive version of DFS, extended to account for back edges. An edge is considered finished, if both of its endpoints are finished. We determine an edge $(u, v)$ to be a bridge, if and only if there are no unfinished back edges remaining in $v$'s subtree, after $v$ has been finished.

In the second DFS, we count the vertices of the subtrees, and choose between the possible solutions at the bridges.

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

struct edge
{
    int t; // target node
    bool bridge;
};
struct node
{
    vector<edge> adj;

    bool vis;
    int bck; // helper variable for dfs_bridge
};
vector<node> g;

int dfs_bridge(int p, int i)
{
    node &v = g[i];
    if(v.vis) { ++v.bck; return 1; } // v.bck stores the number of back edges that end in v
    v.vis = true;

    int bck = 0; // count of the unfinished back edges in the subgraph
    for(edge &e : v.adj)
    {
        if(e.t == p) continue;

        int r = dfs_bridge(i, e.t);
        bck += r;
        e.bridge = (r == 0); 
    }
    // return the number of unfinished back edges
    return bck - v.bck * 2; // back edges that end in v were counted from both sides
}

int dfs_sol(int p, int i, ll &best)
{
    node &v = g[i];
    if(v.vis) { return 0; }
    v.vis = true;

    int count = 1;
    for(edge &e : v.adj)
    {
        if(e.t == p) continue;

        int r = dfs_sol(i, e.t, best);
        if(e.bridge)
            // best stores the largest number of paths that can be broken
            best = max(best, ((ll)g.size() - r) * r);
        count += r;
    }
    // return the number of vertices in the subgraph
    return count;
}

int main()
{
    int t; cin >> t; while(t--)
    {
        int n, m; cin >> n >> m;
        g = vector<node>(n);
        for(int i = 0; i < m; ++i)
        {
            int u, v; cin >> u >> v; --u; --v;
            g[u].adj.push_back({ v, false });
            g[v].adj.push_back({ u, false });
        }

        // find bridges
        dfs_bridge(-1, 0);
        for(node &v : g) v.vis = false;

        // find solutions
        ll best = 0;
        dfs_sol(-1, 0, best);
        cout << (ll)n * (n - 1) / 2 - best << endl;
    }

    return 0;
}


```
