---
layout: post
title:  "코딩테스트 백준 - 19598 최소 회의실 갯수"
date:   2021-09-21
excerpt: "코딩테스트 백준 - 19598 최소 회의실 갯수"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/19598.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/19598)

회의실 문제이다. 그리디 알고리즘 문제인 경우 최대 회의실 갯수를 구하는 문제가 있지만, 이번 문제는 최소 회의실의 갯수를 구하는 문제이다.

일단 먼저 정렬을 시킨 다음 작을 수대로 정렬한 우선순위 큐에 넣어 준다.  
그런 뒤, 꺼내면서 현재 회의실 보다 작은지 큰지 비교해서 값을 넣어준다.

```
#include <stdio.h>
#include <iostream>

#include <queue>
#include <vector>
#include <algorithm>

using namespace std;

struct Conference
{
	int startTime, endTime;
};

bool ConferenceCompare(const Conference& lValue, const Conference& rValue)
{
	return lValue.startTime < rValue.startTime;
}

struct TimeCompare
{
	bool operator()(const int& lValue, const int& rValue)
	{
		return lValue > rValue;
	}
};

int main()
{
	int N;
	cin >> N;
	vector<Conference> v(N);

	for (int i = 0; i < N; i++)
	{
		cin >> v[i].startTime >> v[i].endTime;
	}

	sort(v.begin(), v.end(), ConferenceCompare);
	priority_queue<int, vector<int>, TimeCompare> pq;
	pq.push(v[0].endTime);
	int roomCount = 1;
	for (int i = 1; i < N; i++)
	{
		if (v[i].startTime < pq.top())
		{
			roomCount++;
		}
		else
		{
			pq.pop();
		}
		pq.push(v[i].endTime);
	}
	cout << roomCount << "\n";
}
```