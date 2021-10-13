---
layout: post
title:  "코딩테스트 백준 - 2263 트리의 순회"
date:   2021-09-07
excerpt: "코딩테스트 백준 - 2263 트리의 순회"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/2263.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/2263)

인오더와 포스트오더의 출력 순서를 받고 프리오더를 출력하는 트리 문제이다.

```
In Order : 	8 4 2 9 5 1 10 6 3 11 7  
Post Order : 	8 4 9 5 2 10 6 11 7 3 1  
```
위에 오더순서에서 포스트오더의 맨 마지막의 숫자를 인오더에서 찾아보자  
그 기준으로 왼쪽 노드 오른쪽 노드를 나눌 수 있다.

```
          	<--Left--   ---Right-->
In Order : 	8 4 2 9 5 1 10 6 3 11 7
           	<--Left--   ---Right-->
PostOrder : 	8 4 9 5 2 10 6 11 7 3 1
```

이 방법을 왼쪽 오른쪽에도 사용하면 원래의 완전한 트리를 알 수 있게 된다.

```
#include <stdio.h>
#include <iostream>

#include <vector>

#include <map>
#include <unordered_map>

using namespace std;

struct Node
{
	Node(int value, Node* left, Node* right)
	{
		this->value = value;
		this->left = left;
		this->right = right;
	}

	int value;
	Node* left;
	Node* right;
};

void CreateTree(const int* inOrder, const int*postOrder, Node** tree, const int& inOrderStartIndex, const int& inOrderEndIndex, const int& postOrderStartIndex, const int& postOrderEndIndex)
{
	if(inOrderStartIndex > inOrderEndIndex)
	{
		return;
	}
	if (postOrderStartIndex > postOrderEndIndex)
	{
		return;
	}

	int root = postOrder[postOrderEndIndex];
	int rootIndex = -1;

	for (int i = inOrderStartIndex; i <= inOrderEndIndex; i++)
	{
		if (root == inOrder[i])
		{
			rootIndex = i;
			break;
		}
	}

	if (*tree == NULL)
	{
		Node* node = new Node(root, NULL, NULL);
		*tree = node;
	}
	
	CreateTree(inOrder, postOrder, &(*tree)->left, inOrderStartIndex, rootIndex - 1, postOrderStartIndex, postOrderStartIndex + (rootIndex - 1 - inOrderStartIndex));
	CreateTree(inOrder, postOrder, &(*tree)->right, rootIndex + 1, inOrderEndIndex, postOrderStartIndex + (rootIndex - inOrderStartIndex), postOrderEndIndex - 1);
}

void PreOrder(Node* tree)
{
	if(tree == NULL) {return;}

	printf("%d ", tree->value);
	PreOrder(tree->left);
	PreOrder(tree->right);

	delete tree;
}

int main()
{
	unordered_map<string, int> um;
	um["SEX"] =0;

	int N;
	cin >> N;
	int* inOrder = new int[N];
	int* postOrder = new int[N];

	for (int i = 0; i < N; i++)
	{
		cin >> inOrder[i];
	}

	for (int i = 0; i < N; i++)
	{
		cin >> postOrder[i];
	}

	Node* tree = NULL;

	CreateTree(inOrder, postOrder, &tree, 0, N-1, 0, N-1);
	PreOrder(tree);

	delete[] inOrder;
	delete[] postOrder;
}
```

트리를 만들지 않고 그냥 바로 출려해도 괜찮은데 그냥 트리를 만들고 싶었다.