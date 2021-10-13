---
layout: post
title:  "코딩테스트 백준 - 17952 과제는 끝나지 않아!"
date:   2021-09-23
excerpt: "코딩테스트 백준 - 17952 과제는 끝나지 않아!"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/17952.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/17952)

간단한 스텍 문제이다.


그냥 스텍 시간 끝나면 갱신해주기만 하면 된다 

```
#include <iostream>
#include <vector>

#include <algorithm>

using namespace std;

struct Person
{
	int testValue;
	int interviewValue;
};

bool ComparePerson(const Person& lValue, const Person& r
{
	if (lValue.testValue == rValue.testValue)
	{
		return lValue.interviewValue > rValue.interviewVal
	}

	return lValue.testValue < rValue.testValue;
}

int GetPersonCount(vector<Person>& v)
{
	sort(v.begin(), v.end(), ComparePerson);

	int maxRank = v[0].interviewValue;
	int count = 1;
	for (int i = 1; i < v.size(); i++)
	{
		if (maxRank > v[i].interviewValue)
		{
			count++;
			maxRank = v[i].interviewValue;
		}
	}
	return count;
}

int main()
{
	int T;
	cin >> T;

	for (int i = 0; i < T; i++)
	{
		int N;
		cin >> N;
		vector<Person> v(N);
		for (int j = 0; j < N; j++)
		{
			cin >> v[j].testValue >> v[j].interviewValue;
		}

		cout << GetPersonCount(v) << "\n";
	}
}
```