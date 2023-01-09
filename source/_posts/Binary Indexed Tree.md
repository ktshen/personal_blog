title: Binary Indexed Tree
tags:
  - Data Structures
  - Hackerrank
categories:
  - Algorithms
mathjax: true
date: 2016-07-02 14:13:00
---
給予一個陣列arr，在有以下兩個請求時，可以透過binary indexed tree 加速運算速度

1. (Update) add x to arr[index]
2. (Sum) find the sum from arr[L] to arr[R]

---

{% math %}index=1...N{% endmath %} <br>
{% math %}f[i]= {% endmath %} **frequency of each index **<br>
{% math %}c[i]= {% endmath %} **cummulation from index 1 to index i**<br>
{% math %}tree[i]={% endmath %} 在tree各自負責區段的總和 

usually we will set index 0 as dummy, f[0]=0, c[0] =0

{% codeblock line_number:false %}
num  1  2  3  4  5  6   7   8  9  10  11  12  13  14  15  16

f    1  0  2  1  1  3   0   4  2   5   2   2   3   1  0    2

c    1  1  3  4  5  8   8  12 14  19  21  23  26  27  27  29

tree 1  1  2  4  1  4   0  12  2   7   2  11   3   4   0  29

{% endcodeblock %}

<!--more-->

<p align="center">
<img src="/Binary Indexed Tree/9385E82DD2EEB5AB2AB9C2BABDD9CDA5.gif">
</p>
BIT (according to the graph above)
<p align="center">
<img src="/Binary Indexed Tree/17D73226B1C6E38DC57F3305CB3A4FE3.jpg" width="400" height="400" />
</p>

---

在BIT當中，每一個Node都存放著自己負責“區段”的總和。

將index轉換成binary representation後，透過binary number有幾個1，來尋找從{% math %}1 \Rightarrow index{% endmath %}的總和

若某一個Node的index = idx， 則其存放的總和 {% math %}= idx - 2^r +1 \Rightarrow idx {% endmath %} ( r是該idx從左而右最後一個1的位置）

e.g. 譬如idx = 13;

13 = 01101， 最後一個1的位置是0th（由右而作從零開始，所以Node 13負責的總和為 {% math %}index = 13-2^0+1 = 13 \Rightarrow 13 {% endmath %}，負責自己的Node)

(if idx = 12, 其負責的總和為{% math %} index = 12-2^2+1 = 9 \Rightarrow 12{% endmath %})

換句話說，就是 `(自己的left subtree + 自己的frequency)`

若要求得c[13] = c[01101] = tree[01101] + tree[01100] + tree[01000]

由此可知，皆是從Node移動到Root

* BIT的主要概念即是給一個index 轉換成binary後，若要得知從1至該index的總和，就是把binary index 從右至左每次把最右邊的1砍掉後（刪去最低為的1)，全部的總和。
* 如此作法可以比通常各Node是從1......index全部加起來還要快，因為此方法是看binary index 中有多少個1，就加多少個Node即可，不需要從頭到尾累加。
* 只可以更新數值，不能插入新的節點

---

### 如何得到最低位的1(Rightmost Set bit) ? 

`index & -index`

Proof:

if index = A(anything)1B(all zero) = (...101110101) 1 (000000000….)

-index = **(A1B)¯ + 1 ****(to get the negative binary number of a specific index, take complement and add 1) num****¯ = complement of num**

** = A¯0B¯ + 1 (since B = (1111111…), if we add 1, then B = (000000), and 0 becomes 1)**

** = ****A¯1B **

---

### Sum:

given an index, go from node to root to find the summation 

從BIT 樹圖分析，

從Node開始，若是這個Node是parent 的left tree, 則不需要將sum+=tree[index]

若是這個Node是parent 的right tree, 則需要將sum+=tree[index]

```c
int getSum(const vector<int> &BIT, int index){
    int sum = 0;
    while(index>0){
        sum += BIT[index];
        index -= (index & -index);    //刪去最低位的1
    }
    return sum;
}
```

---

### Update:

跟Sum類似，更新有負責(包含)到該index的Node

換句話說，若是該Node是parent’s left child的話，則需要更新parent

若是該Node是parent’s right child的話，則不需要更新parent

```c
int update(vector<int> BIT, int index, int addition){
    while(index < BIT.size()){      //index < size of the tree
        BIT[index] += addition;
        index += (index & -index);  //將最低位的1加1，就可以找到parent
    }
}
```

---

### Get single node frequency 

最直覺的方式就是 f[index] = c[index] - c[index-1]，意思就是跑兩次Sum;

但如何可以只跑一次就好？

c[index] 和 c[index-1] 一定有相同的

if index = x = A1B**¯** , then y = index - 1 = A0B

從這邊可以看出 

 c[x] = c[A0**B¯**] + tree[A1B**¯**] ,

 c[y] = c[A0**B¯**] + tree[A0(111...111)] + tree[A0(111..110] + tree[A0(111..100] + ………

=\> c[x] - c[y] = tree[A1B**¯**] - { tree[A0(111...111)] + tree[A0(111..110] + tree[A0(111..100] + ………}

所以當y開始一直刪去最低位的1後，最後 y 會變成A0B¯(common predecessor ) ，也就是停止的時候。

＊換句話說，將x負責的區段一直刪去只剩下f[x]而已

```c
int getFrequency(vector<int> BIT, int index){   //index!=0
    int sum = BIT[index];        //set sum = tree[index]
    int common_pre = index - (index & -index);    //find common predecessor
    index--;        //index is not important anymore, we could use it as our variable
    while(index != common_pre){     //after deleting the rightmost set bit, it will become common_pre
        sum -= BIT[index];
        index -= (index & -index);
    }
    return sum;
}
```

---

```c
#include <iostream>
#include <vector>
#include <iomanip>
using namespace std;

int getSum(const vector<int> &BIT, int index){
    int sum = 0;
    while(index>0){
        sum += BIT[index];
        index -= (index & -index);    //刪去最低位的1
    }
    return sum;
}

void update(vector<int> &BIT, int index, int addition){
    while(index < BIT.size()){      //index < size of the tree
        BIT[index] += addition;
        index += (index & -index);  //將最低位的1加1，就可以找到parent
    }
}

int getFrequency(vector<int> BIT, int index){   //index!=0
    int sum = BIT[index];        //set sum = tree[index]
    int common_pre = index - (index & -index);    //find common predecessor
    index--;        //index is not important anymore, we could use it as our variable
    while(index != common_pre){     //after deleting the rightmost set bit, it will become common_pre
        sum -= BIT[index];
        index -= (index & -index);
    }
    return sum;
}

void printTree(const vector<int> &BIT){
    cout<<"i    ";
    for(int i=1; i<BIT.size(); i++) cout<<setw(3)<<i;
    cout<<endl;

    cout<<"F    ";
    for(int k=1; k<BIT.size(); k++)
        cout<<setw(3)<<getFrequency(BIT, k);
    cout<<endl;

    cout<<"C    ";
    for(int t=1; t<BIT.size(); t++)
        cout<<setw(3)<<getSum(BIT, t);
    cout<<endl;
    
    cout<<"Tree ";
    for(int p=1; p<BIT.size(); p++)
        cout<<setw(3)<<BIT[p];
    cout<<endl;
}

vector<int> constructBIT(int arr[], int N)
{
    vector<int> BIT(N+1);
    for(int i=0; i<=N; i++) BIT[i] = 0;  //initialize tree

    for(int j=0; j<N; j++)
        update(BIT, j+1, arr[j]);

    return BIT;
}

int main(int argc, char const *argv[])
{
    int N = 16;   //length of frequecy array
    int freq_arr[] =  { 1, 0, 2, 1, 1, 3, 0, 4, 2, 5, 2, 2, 3, 1, 0, 2};  //given data
    vector<int> BITree = constructBIT(freq_arr, N);

    /* use to demontrate */
    printTree(BITree);
    return 0;
}
```


### [Hackerrank: Similar Pair](https://www.hackerrank.com/challenges/similarpair/problem)

A pair of nodes, ***(a,b)***, is a *similar pair* if both of the following conditions are true:

1. Node ***a*** is the ancestor of node ***b***
2. {% math %}abs(a-b) \leqslant k {% endmath %}

Given a tree where each node is labeled from ***1*** to ***n*** , find the number of similar pairs in the tree.

For example, given the following tree:

<p align="center">
<img src="/Binary Indexed Tree/similarpairsExample.png" width="200" height="250" />
</p>


We have the following pairs of ancestors and dependents:

```
Pair      abs(a-b)
1,2         1
1,3         2
1,4         3
1,5         4
1,6         5
3,4         1
3,5         2
3,6         3
```


If ***k=3*** for example, we have **6** pairs that are similar, where {% math %}abs(a,b) \leqslant k{% endmath %}.


**Function Description**

Complete the similarPair function in the editor below. It should return an integer that represents the number of pairs meeting the criteria.

similarPair has the following parameter(s):

- n: an integer that represents the number of nodes
- k: an integer
- edges: a two dimensional array where each element consists of two integers that represent connected node numbers


**Input Format**

The first line contains two space-separated integers **n** and  **k** , the number of nodes and the similarity threshold. 
Each of the next **n-1** lines contains two space-separated integers defining an edge connecting nodes  **p[i]**  and **c[i]**, where node **p[i]** is the parent to node **c[i]**.

**Constraints**

- {% math %}1 \leqslant n \leqslant 10^5{% endmath %} 
- {% math %}0 \leqslant k \leqslant n{% endmath %} 
- {% math %}1 \leqslant p[i], c[i] \leqslant n{% endmath %} 


**Output Format**

Print a single integer denoting the number of similar pairs in the tree.

**Sample Input**

<pre>
5 2
3 2
3 1
1 4
1 5
</pre>

**Sample Output**

<pre>
4
</pre>

**Explanation**

The similar pairs are **(3,2)**, **(3,1)**, **(3,4)**, and **(3,5)**, so we print **4** as our answer. Observe that **(1,4)** and **(1,5)** are *not* similar pairs because they do not satisfy {% math %}abs(a,b) \leqslant k{% endmath %} for {% math %}k=2{% endmath %}.

<p align="center">
<img src="/Binary Indexed Tree/BA897C1126BE8DC856F1F1EE1090A016.png" width="250" height="300" />
</p>


**Solution**

1. 先建構好題目要的樹（不是BIT)。
2. BIT的用途是，透過存放parent index至BIT，使其Child在尋找他所需要的範圍時(child+k, child-k)，可以透過Sum，統計在這個範圍裡parent 的 total
3. 每次訪問一個Node的時候，先問BIT該Node index的 index+k and index-K這個範圍當中，有多少，然後將加到similar pair
4. 接著就將Node 的index 更新(該index於BIT +1)
5. 然後再用Recursive 跑這個Node的Child
6. 跑完全部該Node的child後，又更新一次Node index (-1)
7. 所以由此可以確保每次訪問BIT時，BIT裡面的各Index一定是該Node的ancestor
8. 如此可以不需要一個一個和parent比對，因為若有M個ancestor，則只需要花Olog(M)的時間來找到該Node的similiar pair，而非O(M)


```cpp
#include <iostream>
#include <vector>
using namespace std;

//bt : bit array
//i and j are starting and ending index INCLUSIVE
long long bit_q(long long * bt,int i,int j)
{
	//尋找範圍裡的總數
    long long sum=0ll;
    while(j>0)
    {
        sum+=bt[j];
        j -= (j & (j*-1));
    }
    i--;
    while(i>0)
    {
        sum-=bt[i];
        i -= (i & (i*-1));
    }
    return sum;
}
//bt : binary indexed tree
//n : size of bit array
//i is the index to be updated
void bit_up(long long * bt,int n,int i,long long diff)
{
    while(i<=n)
    {
        bt[i] += diff;
        i += (i & (i*-1));
    }
}

int n,k;
vector<int> al[100005]; //adjacency list
long long similar_pair;
long long bit[100005];     //binary indexed tree

void dfs(int node)
{
    similar_pair += bit_q(bit,max(1,node-k),min(n,node+k));
    bit_up(bit,n,node,1);       //將該index在binary indexed tree 的值+1，換句話說就是將index push to bit
    for(int i=0; i<al[node].size();i++)
    	dfs(al[node][i]);
	bit_up(bit,n,node,-1);
}
int main() {
    int x,y;
    cin>>n>>k;
    similar_pair=0;   //answer
    for(int i=0;i<=n;i++)   //initialize binary indexed tree
        bit[i]=0ll;

    int r ;
    for(int i=0;i<n-1;i++)
    {
        cin>>x>>y;
        if(i==0) r=x;      //the first one is root
        al[x].push_back(y);   
    }
    dfs(r);
    cout<<similar_pair<<endl;
    return 0;
}
```