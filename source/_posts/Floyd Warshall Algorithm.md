title: Floyd Warshall Algorithm
tags:
  - Graph
  - Hackerrank
categories:
  - Algorithms
mathjax: true
date: 2016-07-19 14:13:00
---
給一個有向圖(direct graph)，建立一個table，用以搜尋從one source to one destination最短路徑。

方法：

假設有N個點，從0開始一直挑到N-1，每次挑的時候，就把它納入intermediate set（一開始是空集合)。

納入之後，讓該點當作intermediate vertex，開始以這個intermediate vertex作為中繼站，試每個點到另一個點的最短路徑(N\*N次），不斷更新該table。
<!--more-->
意思就是若
**i** - - - \> **k** - - - \> **j** 的distance  &nbsp;&nbsp;< &nbsp; &nbsp;**i** - - - \> **j** 的distance <br>
，則更新該table’s **i** to **j** as **table[i][k] + table[k][j]**

當我們挑到**k**點時，表示我們已經考慮了**0, 1, …. k-1個**點了

example: from A to E
<pre>
    B - - - - - 7- - - - - > E
   /                      /
 5/                      / 1
 A - - 2 - - > C - - 3 ->D
</pre>

 
{% codeblock line_number:false %}
                                                                             A->E distance
1. 挑A， table[a][e]== INF is not allowed to test                                  -1
2. 挑B，table[a][b] + table[b][e] < table[a][e] == INF, let table[a][e]=12         12
3. 挑Ｃ，仍無法到達E，因Ｄ還沒在intermediate set，但更新了A->D為5                        12
4. 挑D， table[a][d] + table[d][e] == 5+1 < table[a][e] == 12                       6
5. 挑E，table[a][e] + table[e][e], table[e][e]== INF is not allowed to test         6
ans == 6
{% endcodeblock %}


---

# [Floyd : City of Blinding Lights](https://www.hackerrank.com/challenges/floyd-city-of-blinding-lights/problem)

Given a directed weighted graph where weight indicates distance, for each query, determine the length of the shortest path between nodes. There may be many queries, so efficiency counts.

For example, your graph consists of **5** nodes as in the following:
<p align="center">
<img src="/Floyd Warshall Algorithm/1525461069-142e0d306a-blindingLightsExample.png" />
</p>


A few queries are from node **4** to node **3**, node **2** to node **5** , and node **5** to node **3**.

1. There are two paths from **4** to **3**: 

	- {% math %}4 \Rightarrow 1 \Rightarrow 2 \Rightarrow 3{% endmath %} at a distance of **4 + 5 + 1 = 10**
	- {% math %}4 \Rightarrow 1 \Rightarrow 5 \Rightarrow 3{% endmath %} at a distance of **4 + 3 + 2 = 9** <br>
	In this case we choose path **2**

2. There is no path from **2** to **5**, so we return **-1**.
3. There is one path from **5** to **3**:
	- 	{% math %}4 \Rightarrow 5{% endmath %} at a distance of **2**


**Input Format**

First line has two integers ***N***, denoting the number of nodes in the graph and ***M***, denoting the number of edges in the graph. 

The next **M** lines each consist of three space separated integers *x* *y* *r*  , where *x* and  *y* denote the two nodes between which the *directed* edge ( *x* -\> *y* ) exists, *r* denotes the length of the edge between the corresponding edges. 
The next line contains a single integer **Q** , denoting number of queries. 

The next **Q** lines each, contain two space separated integers *a* and *b*, denoting the node numbers specified according to the question. 

**Constraints** 

- {% math %}2 \leqslant N \leqslant 400 {% endmath %}
- {% math %}1 \leqslant M \leqslant frac{N*(N-1)}{2} {% endmath %}
- {% math %}1 \leqslant Q \leqslant 10^5 {% endmath %}
- {% math %}1 \leqslant x,y \leqslant N {% endmath %}
- {% math %}1 \leqslant r \leqslant 350 {% endmath %}

**If there are edges between the same pair of nodes with different weights, the last one (most recent) is to be considered as the only edge between them.**

**Output Format**

Print **Q** lines, each containing a single integer, specifying the shortest distance between the nodes specified for that query in the input. 

If the distance between a pair of nodes is infinite (not reachable), then print -1 as the shortest distance. 

**Sample Input**

<pre>
4 5
1 2 5
1 4 24
2 4 6
3 4 4
3 2 7
3
1 2
3 1
1 4

</pre>

**Sample Output**

<pre>
5
-1
11
</pre>

**Explanation**

The graph given in the test case is shown as :
<p align="center">
<img src="/Floyd Warshall Algorithm/F67D409077D41849E6DA24F72FA750E5.png" />
</p>


* The nodes A,B,C and D denote the 1,2,3 and 4 node numbers.

The shortest paths for the 3 queries are :

* **A-\>B** (Direct Path is shortest with weight 5)
* **-1** (There is no way of reaching node 1 from node 3, hence unreachable)
* **A-\>B-\>D** (Indirect path is shortest with weight (5+6) = 11 units, the direct path is longer with 24 units length)

```cpp
#include <vector>
#include <iostream>
#include <limits.h>
using namespace std;

void floyd(vector< vector<int> > graph)
{
    //Add all vertices one by one to the set of intermediate vertices.
    for(int k=0; k<graph.size(); k++){
        
        // Pick all vertices as source one by one
        for(int i=0; i<graph.size(); i++){
            
            // Pick all vertices as destination for the
            // above picked source
            for(int j=0; j<graph.size(); j++){
                
                //either i to k or k to j is INT_MAX, there's no way to move from the source to destination
                //however if i==k || k==j, there's no need to consider it
                if(graph[i][k]!=INT_MAX && graph[k][j]!=INT_MAX){
                    if(graph[i][k]+graph[k][j]<graph[i][j])
                        graph[i][j] = graph[i][k]+graph[k][j];
                }
            }
        }
    }
    
    //print solution
    int q;
    cin>>q;
    while(q>0){
        int c, d;
        cin>>c>>d;
        c--;
        d--;
        //need to consider that source and destination are the same, self to self
        if(c==d) cout<<0<<endl;
        
        //if destination is unreachable
        else if(graph[c][d]==INT_MAX) cout<<-1<<endl;
        
        else cout<<graph[c][d]<<endl;
        q--;
    }
}

int main() {
    int N, M;
    cin>>N>>M;
    vector< vector<int> > graph(N, vector<int> (N));
    for(int i=0; i<N; i++)
        for(int j=0; j<N; j++)
            graph[i][j] = INT_MAX;
    while(M>0){
        int a, b, w;
        cin>>a>>b>>w;
        a--;
        b--;
        graph[a][b] = w;
        M--;
    }
    floyd(graph);
    return 0;
}
```