---
layout: post
title:  "코딩테스트 백준 - 1080"
date:   2021-09-17
excerpt: "코딩테스트 백준 - 1080"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/1080.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/1080)

그리드 문제인데 문제만 봤을때는 그래프 탐색 문제인 줄 알았다.  그래서 좀 뻘짓거리를 하루동안 해서 좀 걸렸다 ㅋㅋㅋㅋㅋ

푸는 방법은 두개의 배열에서 각 요소를 비교해 다른 요소가 있으면 그 요소를 기준으로 3x3 배열을 뒤집어준다  
계속 반복해서 현재 배열이 같은지 아닌지 확인한 뒤, 같으면 뒤집은 횟수를 출력 다르면 -1을 출력해주면 된다.

```
#include <stdio.h>
#include <iostream>

#include <vector>
#include <queue>

using namespace std;

struct Vector2
{
	int x, y;
};

Vector2 operator+(const Vector2& lValue, const Vector2& rValue)
{
	return { lValue.x + rValue.x, lValue.y + rValue.y };
}

Vector2 threeByThree = { 3,3 };

void Flip(vector<vector<int>>& flipCards, const Vector2& currnetPosition)
{
	Vector2 flipPosition = currnetPosition + threeByThree;

	for (int i = currnetPosition.y; i < flipPosition.y; i++)
	{
		for (int j = currnetPosition.x; j < flipPosition.x; j++)
		{
			if (flipCards[i][j] == 0)
			{
				flipCards[i][j] = 1;
			}
			else
			{
				flipCards[i][j] = 0;
			}
		}
	}
}

int main()
{
	int N, M;
	cin >> N >> M;
	vector<string> tempA(N);
	vector<string> tempB(N);

	vector<vector<int>> cardsA(N, vector<int>(M));
	vector<vector<int>> cardsB(N, vector<int>(M));
	vector<vector<int>> temp(N, vector<int>(M));

	for (int i = 0; i < N; i++)
	{
		cin >> tempA[i];
	}

	for (int i = 0; i < N; i++)
	{
		cin >> tempB[i];
	}

	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < M; j++)
		{
			cardsA[i][j] = tempA[i][j] - '0';
			cardsB[i][j] = tempB[i][j] - '0';
			temp[i][j] = cardsA[i][j] + cardsB[i][j];
		}
	}
	tempA.clear();
	tempA.shrink_to_fit();
	tempB.clear();
	tempB.shrink_to_fit();

	int count = 0;
	for (int y = 0; y < N-2; y++)
	{
		for (int x = 0; x < M-2; x++)
		{
			if (cardsA[y][x] != cardsB[y][x])
			{
				Flip(cardsA, {x,y});
				count++;
			}
		}
	}

	bool bIsSame = true;
	for (int y = 0; y < N; y++)
	{
		for (int x = 0; x < M; x++)
		{
			if (cardsA[y][x] != cardsB[y][x])
			{
				bIsSame = false;
			}
		}
	}

	if (bIsSame == false)
	{
		printf("-1\n");
	}
	else
	{
		printf("%d\n", count);
	}
}
```