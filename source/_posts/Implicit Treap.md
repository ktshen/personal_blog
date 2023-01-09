title: Implicit Treap
tags:
  - Data Structures
  - Hackerrank
categories:
  - Algorithms
mathjax: true
date: 2016-05-29 14:13:00
---
#### [Hackerrank: Array and simple queries](https://www.hackerrank.com/challenges/array-and-simple-queries/problem)

Given two numbers *N* and *M*. *N* indicates the number of elements in the array *A[](1 - indexed)* and *M* indicates number of queries. You need to perform two types of queries on the array *A[]*. 

You are given *M* queries. Queries can be of two types, type **1** and type **2**. 

* Type 1 queries are represented as `1 i j` : Modify the given array by removing elements from *i* to *j* and adding them to the front.
* Type 2 queries are represented as `2 i j` : Modify the given array by removing elements from *i* to *j* and adding them to the back.

Your task is to simply print  of the resulting array after the execution of  queries followed by the resulting array. 
<!--more-->

**Note** While adding at back or front the order of elements is preserved. 

**Input Format**

First line consists of two space-separated integers, *N* and *M*. 
Second line contains *N* integers, which represent the elements of the array. 
*M* queries follow. Each line contains a query of either *type 1* or *type 2* in the form *i* *j*

**Constraints** 

- {% math %}1\leqslant N, M \leqslant 10^5{% endmath %}
- {% math %}1\leqslant A[i] \leqslant 10^9{% endmath %}
- {% math %}1\leqslant i\leqslant j\leqslant N{% endmath %}

**Output Format**

Print the absolute value i.e. {% math %}abs(A[i]-A[j]){% endmath %}in the first line. 
Print elements of the resulting array in the second line. Each element should be seperated by a single space.

**Sample Input**

<pre>
8 4
1 2 3 4 5 6 7 8
1 2 4
2 3 5
1 4 7
2 1 4
</pre>

**Sample Output**

<pre>
1
2 3 6 5 7 8 4 1
</pre>

**Explanation**

Given array is *{1,2,3,4,5,6,7,8}* .

After execution of query 1 2 4 , the array becomes {2,3,4,1,5,6,7,8} . 

After execution of query 2 3 5 , the array becomes {2,3,6,7,8,4,1,5}. 

After execution of query 1 4 7, the array becomes {7,8,4,1,2,3,6,5}. 

After execution of query 2 1 4, the array becomes *{2,3,6,5,7,8,4,1}*. 

Now {% math %}|A[1]-A[N]|{% endmath %} is {% math %}|(2-1)|{% endmath %} i.e. {% math %}1{% endmath %} and the array is 23657841

```c
#include <iostream>
using namespace std;

struct Node{
	/*  
	treap = tree + heap
	各Node的priority是隨機給的，以此建立一個穩定的log(n) tree
	parent.priority<children.priority
	*/
	int priority;
	//存取array element
	int value;
	/* the method using here is also called implicit treap, since we store size not index
	   size = the number of nodes below the nodes + 1
	   	    = left.size + right.size + 1
	   利用size來得到index(所以才叫implicit)，若為nullptr則為0，若為葉子則為1，透過Inorder使treap取代一array
	*/
	int size;
	//left tree and right tree
	Node *l, *r;
};
//return the size of the node
int sz(Node *t)
{
	return t? t->size:0;
}
//update size of the node when split or merge is done
void update_size(Node *t)
{
	if(t) t->size = sz(t->l)+1+sz(t->r);
}


//used to split a tree into left and right tree
//left contains index from 0 ~ key, right contains index form key+1 ~ N
void split(Node *t, int key, Node *&left, Node *&right, int &add)
{
	if(t==nullptr){
		left = right = nullptr;
		return;
	} 
	//需考慮到因為看整棵tree是利用inorder，所以右邊的tree還需要加root的pos才是對的
	int cur_pos = add+sz(t->l)+1;
	//左邊的樹有包含到該key
	//t為subtree，需要被檢查的部分，利用copy pointer而已

	//因為t的left tree 必定小於key，所以傳遞t的right tree到下一個function以檢查，而因為此t小於
	//等於key，所以屬於left的
	if(cur_pos<=key) add = cur_pos, split(t->r, key, t->r, right, add),  left=t;
	//因為t大於key，所以屬於right的
	else split(t->l, key, left, t->l, add), right = t;
	//完成merge後需要更新該節點的size
	update_size(t);
}

//merge left tree and right into a tree
void merge(Node *&t, Node *left, Node *right)
{
	if(!left || !right) t = left?left:right;
	//merge時，需要考慮到該treap是heap，所以組合時仍需要考慮priorty，以維持該tree為log(n)，
	//而merge時， 已確信兩個left and right are in correct order，
	//而left之index在right的index 之前，透過交換left and right這個來解本題的move front or back
	else if(left->priority<=right->priority) merge(left->r, left->r, right), t = left;
	else merge(right->l, left, right->l), t = right;
	update_size(t);
}

void insert(Node *&t, int pos, int value)
{
	Node *left, *right;
	int add=0;
	split(t, pos-1, left, right, add);

	Node *newnode = new Node;
	newnode->value = value;
	newnode->size = 1;
	newnode->priority = int(rand());
	newnode->l = nullptr;
	newnode->r = nullptr;

	t = nullptr;
	Node  *t1;
	merge(t1, left, newnode);
	merge(t, t1, right);
}

void inOrder(Node *root)
{
    if(root!=nullptr){
        inOrder(root->l);
        cout<<root->value<<" ";
        inOrder(root->r);
    }
}

int main()
{
	int number;
	int total_cmd;
	cin>>number;
	cin>>total_cmd;

	Node *head = nullptr;
	for(int j=1; j<=number; j++){
		int input;
		cin>>input;
		insert(head, j, input);
	}

	while(total_cmd>0){
		int cmd;
		//left index and right index
		int left_x, right_x;
		cin>>cmd;
		cin>>left_x;
		cin>>right_x;

		Node *right = nullptr, *left = nullptr, *mid = nullptr;
		Node *mid_t = nullptr;
		int add=0;

		split(head, left_x-1, left, mid_t, add);
		split(mid_t, right_x, mid, right, add);
		

		head = nullptr;
		Node *result1 = nullptr;
		switch(cmd){
			case 1:
				merge(result1, mid, left);
				merge(head, result1, right);
				break;
			case 2:
				merge(result1, left, right);
				merge(head, result1, mid);
				break;
		}
		total_cmd--;
	}

		int value_1=0;
		int value_2=0;
		Node *tmp_1 = head;
		Node *tmp_2 = head;
		while(tmp_1!=nullptr){
			value_1 = tmp_1->value;
			tmp_1 = tmp_1->l;
		}
		while(tmp_2!=nullptr){
			value_2 = tmp_2->value;
			tmp_2 = tmp_2->r;
		}
		if(value_1>=value_2) cout<<value_1-value_2<<endl;
		else cout<<value_2-value_1<<endl;
		
		inOrder(head);
		cout<<endl;
	return 0;

}
```

reference: <https://threads-iiith.quora.com/Treaps-One-Tree-to-Rule-em-all-Part-2>