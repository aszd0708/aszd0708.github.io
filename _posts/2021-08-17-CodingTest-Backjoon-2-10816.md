---
layout: post
title:  "코딩테스트 백준 - 10816 숫자 카드 2"
date:   2021-08-17
excerpt: "코딩테스트 백준 - 10816 숫자 카드 2"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/10816.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/10816)

이분탐색과 비슷한 방법이긴 하지만, 조금 다르다

숫자들 중에 원하는 숫자 이하의 시작하는 인덱스와 원하는 숫자의 시작되는 인덱스를 구하는 UpperBound, LowerBound를 사용해야 한다.

```

#include <stdio.h>

#include <iostream>
#include <vector>

using namespace std;

typedef long long LL;

void MergeSort(vector<LL>& v, int left, int mid, int right)
{
	int i = left, j = mid + 1;
	int k = 0;
	vector<int> temp(right - left + 1);
	while (i <= mid && j <= right)
	{
		if (v[i] <= v[j])
		{
			temp[k] = v[i];
			i++;
		}
		else
		{
			temp[k] = v[j];
			j++;
		}
		k++;
	}

	while (i <= mid)
	{
		temp[k] = v[i];
		i++;
		k++;
	}

	while (j <= right)
	{
		temp[k] = v[j];
		j++;
		k++;
	}

	k--;
	while (k >= 0)
	{
		v[left + k] = temp[k];
		k--;
	}
}

void MergeSort(vector<LL>& v, int start, int end)
{
	if (start < end)
	{
		int mid = (start + end) / 2;

		MergeSort(v, start, mid);
		MergeSort(v, mid + 1, end);

		MergeSort(v, start, mid, end);
	}
}

int LowerBound(vector<LL>& card, int targetCard)
{
	int left = 0, right = card.size() - 1;

	while (right > left)
	{
		int mid = (left + right) / 2;

		if (card[mid] >= targetCard)
		{
			right = mid;
		}

		else
		{
			left = mid + 1;
		}
	}
	return right;
}

int UpperBound(vector<LL>& card, LL targetCard)
{
	int left = 0, right = card.size() - 1;

	while (right > left)
	{
		int mid = (left + right) / 2;

		if (card[mid] > targetCard)
		{
			right = mid;
		}

		else
		{
			left = mid + 1;
		}
	}
	return right;
}

int main()
{
	int N;
	cin >> N;

	vector<LL> card(N);
	for (int i = 0; i < N; i++)
	{
		cin >> card[i];
	}

	MergeSort(card, 0, card.size() - 1);

	int M;
	cin >> M;

	vector<int> results(M);
	for (int i = 0; i < M; i++)
	{
		LL target;
		cin >> target;

		int lower = LowerBound(card, target);
		int upper = UpperBound(card, target);
		int result = upper - lower;

		if (upper == card.size() - 1 && card[card.size() - 1] == target) { result++; }

		results[i] = result;
	}

	for (int i = 0; i < M; i++)
	{
		printf("%d\n", results[i]);
	}
}
```
(소팅 알고리즘 연습하기 위해 MergeSort를 구현했다.)
```
if (upper == card.size() - 1 && card[card.size() - 1] == target) { result++; }
```
이 부분이 이 문제에서 가장 중요하다.

원하는 값이 가장 큰 값이거나 마지막에 있는 값과 같을 경우엔 UpperBound가 초과하는 값이 아닌 이상으로 되기 때문에 값을 한개 올려준다.