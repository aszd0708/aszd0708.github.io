---
layout: post
title:  "Multi Thread 공부 3일차 (List - 성긴 동기화)"
date:   2021-06-10
excerpt: "Multi Thread 공부 3일차 (List - 성긴 동기화)"
tag:
- C++
- Multi Thread
comments: false
---

# List
이제 하나의 리스트를 여러 스레드가 사용할 수 있게 만들었다.

여러가지 동기화 방식중에 이번엔 성긴 동기화를 사용해서 구현해봤다.

## 성긴동기화
단어부터 생소한 성긴 동기화부터 알아보자

여러개의 스레드가 동시에 오면 한 스레드만 실행시키는 방법이다.

따라서 List에 값을 추가할때 Lock을 걸고 값을 넣은 후 Unlock을 하는 방법이다.

```
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

#define MAX 10000

using namespace std;
using namespace std::chrono;

class Node
{
public:
	int key;
	Node* next;

	Node() { key = 0; next = nullptr; }
		
	Node(int key)
	{
		this->key = key;
		next = nullptr;
	}
	~Node() = default;
};

class List
{
public:
	Node head, tail;
	mutex lock;
	
	List()
	{
		head.key = 0x80000000;
		tail.key = 0x7FFFFFFF;
		head.next = &tail;
	}
	~List(){}

	void Clear()
	{
		Node* ptr;
		while (head.next != &tail)
		{
			ptr = head.next;
			head.next = head.next->next;
			delete ptr;
		}
	}

	bool Add(int key)
	{
		Node* pred;
		Node* curr;

		pred = &head;
		lock.lock();

		curr = pred->next;

		while (curr != &tail)
		{
			pred = curr;
			curr = curr->next;
		}

		if (key == curr->key)
		{
			lock.unlock();
			return false;
		}
		else
		{
			Node* node = new Node(key);
			node->next = curr;
			pred->next = node;
			lock.unlock();
			return true;
		}
	}

	void Print()
	{
		Node* curr = head.next;

		while (curr != &tail)
		{
			cout << curr->key << " ";
			curr = curr->next;
		}
		cout << endl;
	}

	void Print(int count)
	{
		Node* curr = head.next;

		for (int i = 0; i < count; i++)
		{
			cout << curr->key << " ";
			curr = curr->next;
		}
		cout << endl;
	}
};

List temp_list;

void Thread(int thread_count)
{
	for (int i = 0; i < MAX / thread_count; i++)
	{
		temp_list.Add(i);
	}
}

int main()
{
	for (int i = 1; i <= 8; i *= 2)
	{
		steady_clock::time_point start_time = high_resolution_clock::now();
		vector<thread> v;
		for (int j = 0; j < i; j++)
		{
			v.emplace_back(thread(Thread, i));
		}
	
		for (int j = 0; j < v.size(); j++)
		{
			v[j].join();
		}
		steady_clock::time_point end_time = high_resolution_clock::now();
		auto time = end_time - start_time;
		int result_time = duration_cast<milliseconds>(time).count();
	
		temp_list.Print(20);
		cout << "스레드 갯수 : " << i << endl;

		cout << "걸린 시간 : " << result_time << endl;
	}
}
```
```
bool Add(int key)
{
	Node* pred;
	Node* curr;

	pred = &head;
	lock.lock();

	curr = pred->next;

	while (curr != &tail)
	{
		pred = curr;
		curr = curr->next;
	}

	if (key == curr->key)
	{
		lock.unlock();
		return false;
	}
	else
	{
		Node* node = new Node(key);
		node->next = curr;
		pred->next = node;
		lock.unlock();
		return true;
	}
}
```
이 부분을 보면 값을 찾기 전부터 넣은 후까지 락을 걸어주고 있다.

이 리스트를 사용해서 1000개의 값을 리스트에 하나하나 넣은 후 결과를 봤다.

```
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
스레드 갯수 : 1
걸린 시간 : 196
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
스레드 갯수 : 2
걸린 시간 : 757
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
스레드 갯수 : 4
걸린 시간 : 1328
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
스레드 갯수 : 8
걸린 시간 : 1693
```
값은 잘 들어간다.

하지만 문제가 있다. 위에서 말한 성긴동기화의 특성 상 멀티스레드 효과를 못본다. 

아니 시간이 더 걸리는것 같다.

[출처](https://popcorntree.tistory.com/15?category=813523)