---
layout: post
title: "프로그래머스 : 올바른괄호"
tags: [알고리즘, 프로그래머스, Stack]
comments: true
---

> Programmers  

[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/12909)  

### 접근  
내가 푼 방법은 2개의 스택에 저장해서 각각 비교해서 '('index값 < ')'index값이 항상 큰지 확인하도록 풀었다.  
그런데 다른 사람의 풀이중 인상깊은 풀이가 있어 추가했다.  
그냥 변수 하나로 (일땐 증가시키고, )일땐 감소시켜 마이너스일때 false로 판단하게 했다.  

### 코드 (내가 한 방법) 
~~~c++
#include<string>
#include <iostream>
#include <vector>

using namespace std;

bool solution(string s)
{
    bool answer = true;
    vector<int> be;
    vector<int> end;
    for(int i=0;i<s.size();i++){
        if(s[i]=='(') be.push_back(i);
        else end.push_back(i);
    }
    int flag=0;
    if(be.size()==end.size()){
        for(int i=0;i<be.size();i++){
            if(be.back() > end.back()){
                flag=1;
                break;
            }
            else{
                be.pop_back();
                end.pop_back();
            }
        }
    }
    else answer = false;
    
    if(flag==1) answer = false;
    return answer;
}
~~~

### 코드 (다른사람 방법) 
~~~c++
#include<string>
#include <iostream>

using namespace std;

bool solution(string s)
{
    bool answer = true;
    int n=0;
    for(int i=0;i<s.size();i++){
        if(n<0)return false;
        if(s[i]=='(')n++;
        else n--;
    }
    return n==0;
}
~~~