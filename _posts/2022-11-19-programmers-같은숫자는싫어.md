---
layout: post
title: "프로그래머스 : 같은숫자는 싫어"
tags: [알고리즘, 프로그래머스, Stack]
comments: true
---

> Programmers  

[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/12906)  

### 접근  
algorithm 헤더에 포함되어있는 unique함수와 , vector 헤더에 포함되어있는 v.erase(지울 위치(iterator), 지울 끝위치 한칸뒤(포함안됨))을 사용하는 방법과  
배열을 한번 훑으면서 가는 방법  
해서 2가지 방법으로 풀 수 있다.  

### 코드 (첫번째 방법) 
~~~c++
#include <vector>
#include <iostream>

using namespace std;

vector<int> solution(vector<int> arr) 
{
    vector<int> answer;
    for(int i=0;i<arr.size();i++){
        if(answer.empty()) answer.push_back(arr[i]);
        if(answer.back()!=arr[i]) answer.push_back(arr[i]); //첫번째 넣은것과 또 비교해도 괜찮음, 어차피 자기자신과 비교하는 것이라 같기때문에 
    }
    return answer;
}
~~~

### 코드 (두번째 방법) 
~~~c++
#include <vector>//v.erase
#include <iostream>
#include <algorithm>//unique : arr자체를 변화시키고, 패딩시킨 첫번째 원소위치 리턴
using namespace std;

vector<int> solution(vector<int> arr) 
{
    arr.erase( unique(arr.begin(),arr.end()), arr.end());
    vector<int> answer = arr;
    return answer;
}
~~~