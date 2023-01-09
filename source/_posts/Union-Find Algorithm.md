title: Union-Find Algorithm
tags:
  - Graph
categories:
  - Algorithms
mathjax: true
date: 2016-07-12 14:13:00
---
The algorithm is used to detect a cycle in a undirected graph

There are two ways to implement the algorithm, one of which has the worst case in linear time.

First method:

Worst case: linear time {% math %}O(n){% endmath %}

1. If the question has N vertices and M edges, after creating a graph based on previous vertices and edges,now we create a parent list (length N, 每一個index代表vertices, 而存放數字代表其parent) and initialise it with -1 for each elements.
2. 當我們要尋找某一個edge是否產生cycle時，利用edge的source and destination來查看是否產生。透過parent list，當我們在尋找某x時，就利用recursive來一直不斷地找到某一element的parent是-1，然後回傳該parent；同樣的方法，尋找y，若是回傳的parent和x 回傳的parent一樣，代表他們在同一個subset，`​若是再加入x and y，就會產生Cycle`​；若沒有，則將x的subset和y的subset併成一個subset。<br>

<!--more-->


```cpp
#include <iostream>
#include <vector>
using namespace std;

// a structure to represent an edge in graph
struct Edge
{
	int src;
	int dest;
};

// a structure to represent a graph
struct Graph
{
	int V;
	int E;
	vector<Edge> edge;
};

// Creates a graph with V vertices and E edges
Graph* createGraph(int V, int E)
{
	Graph *graph = new Graph;
	graph->V = V;
	graph->E = E;
	graph->edge = vector<Edge>(E);
	return graph;
}

int find(const vector<int> &parent, int i)
{
	if(parent[i]==-1)
		return i;
	return find(parent, parent[i]);
}

void Union(vector<int> &parent, int x, int y)
{
	parent[x] = y;
}

// The main function to check whether a given graph contains 
// cycle or not
bool isCycle(Graph* graph, int V, int E)
{
    //用以尋找是否有cycle
	vector<int> parent(V, -1);
	for(int i=0; i<E; i++){
		int x = find(parent, graph->edge[i].src);
		int y = find(parent, graph->edge[i].dest);

		if(x==y)
			return 1;

		Union(parent, x, y);
	}
	return 0;
}


int main()
{
/* Let us create following graph
         0
         | 
         1 
       /   \        
      /      \
     2 - - -  3        
*/  
         
	int V = 4, E = 4;
	struct Graph *graph = createGraph(V,E);

	graph->edge[0].src  = 0;
	graph->edge[0].dest = 1;

	graph->edge[1].src  = 1;
	graph->edge[1].dest = 3;

	graph->edge[2].src  = 1;
	graph->edge[2].dest = 2;

	graph->edge[3].src  = 2;
	graph->edge[3].dest = 3;

	if(isCycle(graph, V, E))
		cout<<"Has Cycle"<<endl;
	else
		cout<<"No Cycle"<<endl;

	return 0;
}
```

簡單看parent list的變化
{% codeblock line_number:false %}
V  0  1  2  3 
p -1 -1 -1 -1
{% endcodeblock %}

加入Edge 0 v0 -\> v1 &nbsp;&nbsp;*將V0的parent 變成 1， V0 and V1 are now in the same subset*
{% codeblock line_number:false %}
V 0  1  2  3
p 1 -1 -1 -1
{% endcodeblock %}

加入Edge 1 v1 -\> v3 &nbsp;&nbsp;  *V0, V1 and V3 are now in the same subset*
{% codeblock line_number:false %}
V 0 1  2  3 
p 1 3 -1 -1 
{% endcodeblock %}
加入Edge 2 v1 -\> v2  &nbsp;&nbsp;*在find V1時，會回傳V3，所以改變的是V3的parent* 
{% codeblock line_number:false %}
V 0 1  2 3    //V0, V1, V2 and V3 are now in the same subset

p 1 3 -1 2
{% endcodeblock %}

加入Edge 3 v2 -\> v3  &nbsp;&nbsp; *because V2 and V3 are in the same subset, if we add edge 3, a cycle will be created.*

---

但是，以上方法可以看出，每次find的時候，會把自己屬於的subset全部跑過一遍，一直找到parent是-1的vertex，其worst case O(N)

該方法可以被optimised to O(logN), the technique is called `​union by rank`​

方法：

1. 每個節點在初始時其parent都是自己，rank=0(每有比誰高或低）
2. 每次搜尋兩個vertices，都會回傳他的root，若是root相同，代表兩點在同一個subset
3. 並且在搜尋時，會使各parent和該Node的parent都成為該subset的root (path compression)，這樣子下次再做搜尋時，可以直接找到該subset的root，由此好好利用每一次的搜尋
4. 若是兩點不在同一個subset，代表現在若要加一條edge，那個兩點之subset會合併成同一subset

```cpp
#include <iostream>
#include <vector>
struct Edge
{
	int src;
	int dest;
};

struct Graph
{
	int V;
	int E;
	vector<Edge> edge;
};

struct subset
{
    int parent;
    int rank;
};

Graph* createGraph(int V, int E)
{
	Graph *graph = new Graph;
	graph->V = V;
	graph->E = E;
	graph->edge = vector<Edge>(E);
	return graph;
}

int find(vector<subset> &subsets, int i)
{
    // find root and make root as parent of i (path compression)
	if(subsets[i].parent!=i)
		subsets[i].parent = find(subsets, subsets[i].parent);
	return subsets[i].parent;
}

void Union(vector<subset> &subsets, int xroot, int yroot)
{
	// Attach smaller rank tree under root of high rank tree
	if(subsets[xroot].rank < subsets[yroot].rank)
		subsets[xroot].parent = yroot;
	else if(subsets[xroot].rank > subsets[yroot].rank)
		subsets[yroot].parent = xroot;

	// If ranks are same, then make one as root and increment
    // its rank by one
	else{
		subsets[xroot].parent = yroot;
		subsets[yroot].rank++;
	}
}

bool isCycle(Graph* graph, int V, int E)
{
	vector<subset> subsets(V);
	for(int v=0; v<V; v++){
		subsets[v].parent = v;
		subsets[v].rank = 0;
	}
	
	// Iterate through all edges of graph, find sets of both
    // vertices of every edge, if sets are same, then there is
    // cycle in graph.
	for(int i=0; i<E; i++){
		int x = find(subsets, graph->edge[i].src);
		int y = find(subsets, graph->edge[i].dest);

		if(x==y)
			return 1;

		Union(subsets, x, y);
	}
	return 0;
}


int main()
{
	int V = 4, E = 4;
	struct Graph *graph = createGraph(V,E);

	graph->edge[0].src  = 0;
	graph->edge[0].dest = 1;

	graph->edge[1].src  = 1;
	graph->edge[1].dest = 3;

	graph->edge[2].src  = 1;
	graph->edge[2].dest = 2;

	graph->edge[3].src  = 2;
	graph->edge[3].dest = 3;


	if(isCycle(graph, V, E))
		cout<<"Has Cycle"<<endl;
	else
		cout<<"No Cycle"<<endl;

	return 0;

}
```

可參考：<http://www.csie.ntnu.edu.tw/~u91029/Set.html> 同樣方法不同說法