---
layout: post
title: "프로그래머스 : 프린터"
tags: [알고리즘, 프로그래머스, Queue]
comments: true
---

> Programmers  

[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/42587#)  

### 접근  
이 문제의 핵심은 priority_queue를 사용하는 것인데, front()가 없으므로 top()을 이용해야한다.  
또한, pair를 가진 queue를 만들어서 사용하면 된다. 
중간에 문제를 잘못 이해해서 애를 먹었다. [1,9,3,8,2,1,7,1]에서 location을 6으로 했을때, 3이 출력이 되어야한다.   
가장 큰 값을 찾았어도, 뒤에 숫자들 또한 sort가 되어야한다.  

### 코드 
~~~c++
#include <string>
#include <vector>
#include <queue>
#include <iostream>
using namespace std;

int solution(vector<int> priorities, int location) {
    int answer = 0;
    priority_queue<int> pq;
    queue<pair<int,int>> q;
    vector<pair<int,int>> sorted;//sort된 것들만 저장
    for(int i=0;i<priorities.size();i++){
        pq.push(priorities[i]);
        q.push(make_pair(i,priorities[i]));
    }
    pair<int,int> tmp;
    while(!q.empty()){
        if(q.front().second < pq.top()){
            tmp = q.front();
            q.pop();
            q.push(tmp);
        }
        else {
            sorted.push_back(q.front());
            q.pop();
            pq.pop();
        }
    }
    for(int i=0;i<priorities.size();i++){
        if(sorted[i].first == location) return i+1;
    }
}
~~~
