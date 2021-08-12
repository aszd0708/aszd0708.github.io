---
layout: post
title:  "코딩테스트 백준 - 8858"
date:   2021-08-12
excerpt: "딩테스트 백준 - 8858"
tag:
- C++
- CodingTest
- 백준
comments: false
---

<img src = "../assets/img/project/codingtest/backjoon/5585.PNG" width="100%">

그리드 문제중에 가장 간단한 문제이다.

1000엔에 받은 정수를 빼고 남은 정수를 큰 순서대로 나눠주고 그 몫만큼 곱해서 빼주고, 반복하면 된다.

```
#include <stdio.h>
#include <iostream>

using namespace std;

int main()
{
	int currentMoney;
	cin >> currentMoney;

	currentMoney = 1000 - currentMoney;

	int coins[6] = { 500, 100, 50, 10, 5, 1 };
	int coinCount = 0;
	int coinIndex = 0;
	while (currentMoney > 0)
	{
		int temp = currentMoney / coins[coinIndex];

		currentMoney -= coins[coinIndex] * temp;
		coinCount += temp;

		coinIndex++;
	}

	printf("%d\n", coinCount);
}
```

정말 간단하다.