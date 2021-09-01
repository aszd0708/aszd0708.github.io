---
layout: post
title:  "코딩테스트 백준 - 1920"
date:   2021-08-17
excerpt: "코딩테스트 백준 - 1920"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/1920.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/1920)

이분탐색에 관한 문제이다.

주어진 수들을 정렬한 뒤, 그 수를 처음과 끝 사이에 있는 값보다 작거나 클 경우를 비교해서 수의 범위를 좁혀가는 방법이다.

```
#include <stdio.h>

#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int GetBinarySearch(vector<int>& v, int findNumber)
{
	int left = 0;
	int right = v.size() - 1;

	while (left <= right)
	{
		int middle = (left + right) / 2;

		if (v[middle] > findNumber)
		{
			right = middle - 1;
		}
		else if (v[middle] < findNumber)
		{
			left = middle + 1;
		}

		else
		{
			return 1;
		}
	}
	return 0;
}

int BinarySearch(const vector<int>& v, int target, int left, int right)
{
	if (left > right)
	{
		return 0;
	}

	int middle = (left + right) / 2;

	if (v[middle] > target)
	{
		return BinarySearch(v, target, left, middle - 1);
	}

	else if (v[middle] < target)
	{
		return BinarySearch(v, target, middle + 1, right);
	}
	else
	{
		return 1;
	}
}

int main()
{
	int N;
	cin >> N;
	vector<int> v(N);

	for (int i = 0; i < N; i++)
	{
		cin >> v[i];
	}
	sort(v.begin(), v.end());
	int M;
	cin >> M;
	vector<int> answer(M);
	for (int i = 0; i < M; i++)
	{
		int num;
		cin >> num;
		answer[i] = BinarySearch(v, num, 0, v.size() - 1);
	}
	for (int i = 0; i < M; i++)
	{
		printf("%d\n", answer[i]);
	}
}
```
재귀함수를 사용하는 방법과 반복문으로 사용하는 방법을 사용했다.