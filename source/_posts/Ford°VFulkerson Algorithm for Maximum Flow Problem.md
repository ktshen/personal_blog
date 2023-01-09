title: Ford–Fulkerson Algorithm for Maximum Flow Problem
tags:
  - Graph
categories:
  - Algorithms
mathjax: true
date: 2016-12-23 14:13:00
---
Given a graph which represents a flow network where every edge has a capacity. Also given two vertices source **s** and sink **t** in the graph, find the maximum possible flow from s to t with following constraints:

a) Flow on an edge doesn’t exceed the given capacity of the edge.

b) Incoming flow is equal to outgoing flow for every vertex except **s** and **t**.

---

首先，給予各點之間的關係（construct an adjacent list)
基本的找法是： 

* 尋找一條從s -\> t 的有效路徑 (BFS or DFS)
* 找到此路徑能通過的最小flow，更新Residual Graph
* 將flow加到 max\_flow
* 結束後，return max\_flow

Residual Graph：一開始先複製original graph，而其點與點之間存放的是“剩餘可以通過的Flow (residual capacity)”。
<!--more-->
換句話說，每當找到一條路徑可以是flow通過時，假設某edge的capacity 是 C，而通過的flow 是 F，則將更新此edge於該Residual Graph成 C-F，若是C-F=0，代表此edge不能再被挑選，因為沒有剩餘的quota。

需要注意的是，在更新residual某條edge時，假設是從u -\> v，則u -\> v 將被減掉 F，但是反方向 v -\> u 卻要增加F (起初反方向是0)，這個動作叫做”Cancellation”。

> **各Node的進出要維持不變**

以下圖為例 （整條黑色路徑假設是s -\> b -\> v -\> a -\> c -\> t）

現在先看上面那條，若要推送一個10 flow時，在更新 10 flow / 15 capacity這條路徑時`（設u ->`(10/10)` a ->(10/15) v)`，則` u->v`在 Residual Graph上應該被減去10 ，剩餘5 ；但在反方向 `v -> u` 這邊，卻要增加**10**。

之所以如此，我們可以看到若是在選擇上面那條後，準備要選擇中間這條路徑時 `(u -> b -> v)`，我們可以將 `10/15` 這條的` 5` 搬移到上面的 `0/9 ( a -> c )`，使其變成 `5/9`，而讓中間這條能夠傳一個`Flow = 5 `至 **v** 再到 **t** ，對於**v**來說，輸入的還是總和還是**10**（守恆）。換個角度想，就好像是轉換跑道，從`s -> b -> v -> a -> c -> t。`​ 所以，在更新時增加的flow (reverse direction)，可以當作是能夠被Cancel掉的quota。

<p align="center">
<img src="/Ford–Fulkerson Algorithm for Maximum Flow Problem/8192498D233C4AD0BAE0735BE71A5BCF.jpg" width="500" height="300" />
</p>

---

From Geeksforgeeks

<p align="center">
<img src="/Ford–Fulkerson Algorithm for Maximum Flow Problem/D3E155A097BE2558402055553E91AF0B.png" width="500" height="200" />
</p>

ans: The maximum possible flow in the above graph is 23.

<p align="center">
<img src="/Ford–Fulkerson Algorithm for Maximum Flow Problem/E8F4263CD10E3CF47AEFDE0F3E249A6A.png" width="500" height="200" />
</p>

```cpp
#include <iostream>
#include <limits.h>
#include <string.h>
#include <queue>
using namespace std;
 
// Number of vertices in given graph
#define V 6
 
/* Returns true if there is a path from source 's' to sink 't' in
  residual graph. Also fills parent[] to store the path */
bool bfs(int rGraph[V][V], int s, int t, int parent[])
{
    // Create a visited array and mark all vertices as not visited
    bool visited[V];
    memset(visited, 0, sizeof(visited));
 
    // Create a queue, enqueue source vertex and mark source vertex
    // as visited
    queue <int> q;
    q.push(s);
    visited[s] = true;
    parent[s] = -1;
 
    // Standard BFS Loop
    while (!q.empty())
    {
        int u = q.front();
        q.pop();
        
        // if u = t, because all t's adjacent list elements are zero, no new element push to queue, queue will be empty
        for (int v=0; v<V; v++)   
        {
            if (visited[v]==false && rGraph[u][v] > 0)
            {
                q.push(v);
                parent[v] = u;
                visited[v] = true;
            }
        }
    }
 
    // If we reached sink in BFS starting from source, then return
    // true, else false
    return (visited[t] == true);
}
 
// Returns tne maximum flow from s to t in the given graph
int fordFulkerson(int graph[V][V], int s, int t)
{
    int u, v;
 
    // Create a residual graph and fill the residual graph with
    // given capacities in the original graph as residual capacities
    // in residual graph
    int rGraph[V][V]; // Residual graph where rGraph[i][j] indicates 
                     // residual capacity of edge from i to j (if there
                     // is an edge. If rGraph[i][j] is 0, then there is not)  
    for (u = 0; u < V; u++)
        for (v = 0; v < V; v++)
             rGraph[u][v] = graph[u][v];
 
    int parent[V];  // This array is filled by BFS and to store path
 
    int max_flow = 0;  // There is no flow initially
 
    // Augment the flow while there is path from source to sink
    while (bfs(rGraph, s, t, parent))
    {
        // Find minimum residual capacity of the edges along the
        // path filled by BFS. Or we can say find the maximum flow
        // through the path found.
        int path_flow = INT_MAX;
        for (v=t; v!=s; v=parent[v])
        {
            u = parent[v];
            path_flow = min(path_flow, rGraph[u][v]);
        }
 
        // update residual capacities of the edges and reverse edges
        // along the path
        for (v=t; v != s; v=parent[v])
        {
            u = parent[v];
            rGraph[u][v] -= path_flow;
            rGraph[v][u] += path_flow;   
        }
 
        // Add path flow to overall flow
        max_flow += path_flow;
    }
 
    // Return the overall flow
    return max_flow;
}
 
// Driver program to test above functions
int main()
{
    // Let us create a graph shown in the above example
    int graph[V][V] = { {0, 16, 13, 0, 0, 0},
                        {0, 0, 10, 12, 0, 0},
                        {0, 4, 0, 0, 14, 0},
                        {0, 0, 9, 0, 0, 20},
                        {0, 0, 0, 7, 0, 4},
                        {0, 0, 0, 0, 0, 0}
                      };
 
    cout << "The maximum possible flow is " << fordFulkerson(graph, 0, 5);
 
    return 0;
}
```