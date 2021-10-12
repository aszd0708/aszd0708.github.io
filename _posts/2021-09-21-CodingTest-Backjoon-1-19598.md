---
layout: post
title:  "코딩테스트 백준 - 11559"
date:   2021-09-19
excerpt: "코딩테스트 백준 - 11559"
tag:
- CodingTest
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/11559.PNG" width="100%">

[문제](https://www.acmicpc.net/problem/11559)

뿌요뿌요 문제이다.

연결되어있는 같은 색이 4개 있으면 없애준다.  
그런 뒤, 밑에 빈공간이 있는 색을 밑으로 내려준다.  
그런뒤 같은 색이 있으면 없애준다.  
그런 뒤, 밑에 빈공간이 있는 색을 밑으로 내려준다. ......

반복이다.

여기서 틀렸둔 부분이 있는데 같은 색이 한번 없어졌을 때 한번인지, 같은 색 들이 없어졌을때 한번인지 몰라서 틀렸다 ㅋ

그리고 내리는 부분도 헷갈려서 시간이 좀 걸렸다.

```
#include <stdio.h>
#include <iostream>
#include <queue>
#include <vector>
#include <string>

using namespace std;

struct Vector2
{
	int x, y;
};

Vector2 operator+(const Vector2& lValue, const Vector2 rValue)
{
	return{ lValue.x + rValue.x , lValue.y + rValue.y };
}

Vector2 dir[4] = { {1,0},{-1,0}, {0,1}, {0,-1} };

bool SetPop(vector<string>& v, vector<vector<bool>>& bIsVisited, const Vector2& startPosition)
{
	queue<Vector2> q;
	q.push(startPosition);
	char curColor = v[startPosition.y][startPosition.x];
	vector<Vector2> positions;
	while (!q.empty())
	{
		Vector2 currentPosition = q.front();
		q.pop();

		if (bIsVisited[currentPosition.y][currentPosition.x] == true) { continue; }
		bIsVisited[currentPosition.y][currentPosition.x] = true;

		positions.emplace_back(currentPosition);

		for (int i = 0; i < 4; i++)
		{
			Vector2 movePosition = currentPosition + dir[i];

			if (movePosition.x < 0 || movePosition.x >= v[0].size() || movePosition.y < 0 || movePosition.y >= v.size()) { continue; }
			if (v[movePosition.y][movePosition.x] == curColor)
			{
				q.push(movePosition);
			}
		}
	}

	if (positions.size() >= 4)
	{
		for (int i = 0; i < positions.size(); i++)
		{
			v[positions[i].y][positions[i].x] = '.';
		}
		return true;
	}
	else
	{
		return false;
	}
}

void SetDown(vector<string>& v)
{
	for (int i = v.size() - 1; i >= 0; i--)
	{
		for (int j = 0; j < v[i].size(); j++)
		{
			if (v[i][j] == '.') { continue; }
			int currentY = i;
			while (true)
			{
				if (currentY >= v.size() - 1 || j >= v[i].size()) { break; }
				if (v[currentY + 1][j] != '.') { break; }

				v[currentY + 1][j] = v[currentY][j];
				v[currentY][j] = '.';
				currentY++;
			}
		}
	}
}

int main()
{
	vector<string> v(12);

	for (int i = 0; i < v.size(); i++)
	{
		cin >> v[i];
	}
	int count = 0;
	bool bIsPop = true;
	vector<vector<bool>> bIsVisited(v.size(), vector<bool>(v[0].size(), false));
	while (bIsPop)
	{
		bIsPop = false;
		for (int i = 0; i < v.size(); i++)
		{
			for (int j = 0; j < v[i].size(); j++)
			{
				if (v[i][j] != '.')
				{
					bool isPop = SetPop(v, bIsVisited, { j,i });
					if (isPop)
					{
						bIsPop = true;
					}
				}
			}
		}
		SetDown(v);
		for (int i = 0; i < v.size(); i++)
		{
			fill(bIsVisited[i].begin(), bIsVisited[i].end(), false);
		}
		if (bIsPop == true)
		{
			count++;
		}
	}

	cout << count << "\n";
}
```