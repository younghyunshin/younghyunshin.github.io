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


## Calibration 적용하기

On Calibration of Modern Neural Networks 논문에서 가장 성능이 좋은 것으로 나왔던 Temperature scaling을 적용해보자.  

![Center example image](https://user-images.githubusercontent.com/48853055/178239766-03450fe0-c4b4-40fd-9f3e-d25699b999fc.png "Center" width="100" height="150"){: .center-image} 



## CABasicAnimation

```swift
let animation = CABasicAnimation(keyPath: "opacity")
animation.fromValue = 0
animation.toValue = 1
```
아래 코드는 backgroundColor를 변화시키는 코드이다.


위 세 가지 경우 모두 fromValue, toValue라는 값을 지정해주는 것을 알 수 있다. 직관적으로 뭔가 fromValue의 값에서 toValue의 값으로 animation이 동작한다는 것을 느낄 수 있다. 실제로 애플공식문서에서 설명하는 definition도 다음과 같다:

`fromValue` : Defines the value the receiver uses to **start** interpolation.

`toValue` : Defines the value the receiver uses to **end** interpolation.

`byValue` : Defines the value the receiver uses to **perform** relative interpolation.

이 세 property들은 모두 optional이며 적어도 두개는 채워져아 한다. (그런데 하나만 non-nil일 때와 모두 nil인 경우에 대해서도 써있다..?) 각각이 옵셔널일 때 animation의 behavior가 달라진다.

- fromValue, toValue 모두 non-nil : fromValue → toValue로 연결(interpolates)
- fromValue, byValue 모두 non-nil : fromValue → (fromValue + byValue)
- byValue, toValue 모두 non-nil : (to - byValue) → toValue
- fromValue만 non-nil : fromValue → current presentation value of the property
- toValue만 non-nil : current value of key path → toValue
- byValue만 non-nil : current value of key path → (current value + byValue)
- 모두 nil : previous value of key path → current value of key path



## References

[Deep play 블로그 - Calibration에 대하여](https://3months.tistory.com/490)

[논문 - On Calibration of Modern Neural Networks](https://arxiv.org/pdf/1706.04599.pdf)
