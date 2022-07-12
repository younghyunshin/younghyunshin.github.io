---
layout: post
title: "모델에 Calibration 적용하기"
tags: [딥러닝, 논문]
comments: true
---

> 말로만 듣던 Calibration에 대해  

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
이런 효과를 통해 한쪽의 값이 높은걸 약하게 할 수 있다.      

## 적용 안하기로.. 

모델의 결과 confusion matrix를 보니, open/close/unknown간의 관계에서 open/close가 서로 헷갈리는 걸로 나왔다. 
Unknown과 헷갈리는 경우는 거의 없었다. 즉, Unknown을 새로 추가함으로 모델이 어려움을 겪는 것이 없다는 결론이 나왔다.   
Calibration이 별로 효과가 없을 것이라 예상하여 적용 안하기로 ...


## References

[Deep play 블로그 - Calibration에 대하여](https://3months.tistory.com/490)

[논문 - On Calibration of Modern Neural Networks](https://arxiv.org/pdf/1706.04599.pdf)
