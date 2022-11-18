---
layout: post
title: "프로그래머스 : 주식가격"
tags: [알고리즘, 프로그래머스, Stack]
comments: true
---

> Programmers  

[주식가격 문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/42584)  

### 접근  
이 문제의 핵심은 '정체되거나 증가하는 부분'들은 상관없기에, 이 부분들을 스택에 저장해놓는 것이다.  
또한, 스택에 넣는 것은 '초' 에 대한 정보를 저장한다는 것이다.  
그래서 감소하는 부분이 발생하면 스택에서 제일 top에 있는 것부터 순차적으로 현재값과 비교하면서, 과거가 더 크면 (현재초-과거초)를 answer[과거]에 넣고, 현재초를 스택에 넣는다.  
극단적으로 증가하는 경우, 극단적으로 감소하는 경우를 생각하면 쉽다.   

### 코드  
~~~c++
#include <string>
#include <vector>
#include <stack>
#include <iostream>

using namespace std;

vector<int> solution(vector<int> prices) {
    int le = prices.size(); 
    vector<int> answer(le);
    stack<int> s;
    s.push(0);
    for(int i=1;i<le;i++){
        if(prices[i-1] <= prices[i]){//증가
            s.push(i);
        }
        else{//감소
            while(!s.empty() && prices[s.top()] > prices[i]){
                int past = s.top();
                answer[past] = i-past;
                s.pop();
            }
            s.push(i);
        }
    }
    while(!s.empty()){
        answer[s.top()] = (le-1)-s.top();
        s.pop();
    }
    return answer;
}
~~~
