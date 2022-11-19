---
layout: post
title: "프로그래머스 : 기능개발"
tags: [알고리즘, 프로그래머스, Stack]
comments: true
---

> Programmers  

[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/42586#)  

### 접근  
이 문제의 핵심은 몫을 얻을때, 반올림된 몫을 얻는 방법이다.  
예를들어, 7/3의 몫을 2가 아닌, 3을 얻고 싶을때 어떻게 하면 좋을까,  
바로 (x-1)/y+1 을 하면 된다.    

### 코드 
~~~c++
#include <string>
#include <vector>
#include <cmath>
#include <iostream>

using namespace std;

vector<int> solution(vector<int> progresses, vector<int> speeds) {
    vector<int> answer;
    vector<int> bepolist;
    int x,y,bepo;
    //(x-1)/y+1 == 반올림 몫
    for(int i=0;i<progresses.size();i++){
        x = 100-progresses[i];
        bepo = (x-1)/speeds[i]+1;
        if(!bepolist.empty() && bepolist.back()>= bepo){
            bepolist.push_back(bepolist.back());
            answer[answer.size()-1]++;
        } 
        else{
            bepolist.push_back(bepo);
            answer.push_back(1);
        }
    }
    return answer;
}
~~~
