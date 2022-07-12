---
layout: post
title: "자체 개발 모델에 Calibration 적용해볼..까?"
tags: [딥러닝, 논문]
comments: true
---

> (스포) 적용 안함..ㅋㅋ  

⚠ 딥러닝 알못의 글이므로 틀린 정보가 있을 수 있습니다.  

# Calibration이 뭘까?

일단 먼저 이 개념이 나오게 된 뒷 배경을 알아야한다.   
Generally 현대 모델은 `overconfident`하다. 다시 말해, 모델이 긴가민가 한 것도 90%이상의 확률로 무엇이다! 예측한다는 것이다.  
모르면 나 몰랑! 해야하는데, 하나로 결정을 해버리는 현상을 overconfident하다고 말한다.    
그렇기에, 이러한 모델의 문제점을 개선하고자 하는 것이 calibration이다. 
그럼 calibration이 뭐하는데?!   
calibration이란, 출력값이 실제 confidence (논문에서 calibrated confidence 로 표현) 를 반영하도록 만드는 것이다.  
정의만 보면 뭔소리지 할 수 있다. 쉽게 말해, 모델이 헷갈리면 헷갈려!! 할 수 있게 해주고, 모델이 잘 판단하면 뭔지알겠어!!하게 해주는 것이다.  


## Why Calibration?

모델이 헷갈리는 것들에 대해 따로 기각하거나, 사람이 확인해야하는 작업이 필요할 때가 많다.  
내가 이 논문을 읽는 이유도, 눈을 감았는지 떴는지 판별하는 모델이 선글라스를 쓴 눈처럼 판별하기 애매한 상황에서 모델의 출력값을 보고 기각하기위함이다.  
실제 모델에서 선글라스 처럼 애매한 상황에서의 score가 비슷하게 나오기도하고, 한쪽으로 쏠리게 나오기도 했다.  
이렇듯, 모델의 출력값을 통해 모델이 헷갈려 하는 상황, 또는 문제가 될 수 있는 상황을 따로 분류하기 위해 필요할 것이다.   


## Temperature scaling

On Calibration of Modern Neural Networks 논문에서 가장 성능이 좋은 것으로 나왔던 Temperature scaling에 대해 알아보자.    

![Center example image](https://user-images.githubusercontent.com/48853055/178239766-03450fe0-c4b4-40fd-9f3e-d25699b999fc.png "Center" width="100" height="150"){: .center-image} 

처음에 식을 보곤 읭.. 뭐지 했었다.  
차근차근 하나씩 이해해보자. 먼저 Temperature scaling은 multi classification model에 사용하는 것이다.  
예를 들어, 5개의 클래스를 가진 multi classification model(`k==5가 될 것임`)이라고 해보자.  
zi는 예시로 [0.5, 0.5, 1.0, 0.5, 0.5] 이렇게 되어있다고 해보자.  
그러면 여기서 가장 큰 값이 1.0이고 이것의 k는 3이기에 3클래스(`network outputs a class prediction`)로 판별이 될 것이다.  
그러나 여기서 식은 T로 모든 값들을 나눈다. T를 1로 하면, 값이 변화가 없을 것이다.  
그런다음 softmax로 전체의 합이 1이되게 하기에, 전체를 전체 합인 3.0으로 나눈다.  
zi는 [1/6, 1/6, 2/6, 1/6, 1/6]이 될 것이다.  
그러면 이제 p는 2/6이 된다. 그렇기에 T를 1로 하면, 원래 softmax를 거치는 것과 동일한 효과를 얻게 된다.  
T값이 1이 아닌 값으로 나누게 되면, 예를들어 2로 나누게 되면 모든 값들이 작아지게 되면서, 평준화가 된다.        

## 적용 안하기로.. 

원래 이것을 적용하기로 한 이유가, 나의 모델은 눈이 Open(떴는지) / Close(감았는지)를 판별하는 모델이었다.  
그러다 여기서 판별하기 어려운 상황, 예를들면 선글라스를 쓴 눈 같은 것도 판별하기 위해 Calibration을 적용하기로 했었다.  
그전에 클래스를 하나 더 만들어 Open(떴는지) / Close(감았는지) / Unknown(기각 클래스)로 바꿔서 학습했다.    
그 결과 나쁘지 않은 결과를 얻었다..!    
Calibration을 적용해도 일단 Max값 자체는 변화하지 않으니 기각 클래스가 있다면 Calibration을 적용해도 별 차이가 없을 거라는 결론을 내렸다.  
그렇기에 기각 클래스를 없애고 Calibration을 적용해서 Overconfident를 줄인 다음에 일정 Threshold 미만되는 경우에 기각하게 하는 형태로 적용해야 의미가 있을 것 같아, 나중에 적용해보기로..!    
이와 별개로 Unknown과 Open/Close 간에 헷갈리는 게 많다면 Unknown에 대한 민감도를 좀 낮추더라도 Open/Close 정확도를 올리는 형태로 학습하려 했는데, Confusion Matrix를 보니 Open/Close 둘이 헷갈리고 있다;;; ㅎㅎ..   


## References

[Deep play 블로그 - Calibration에 대하여](https://3months.tistory.com/490)

[논문 - On Calibration of Modern Neural Networks](https://arxiv.org/pdf/1706.04599.pdf)
