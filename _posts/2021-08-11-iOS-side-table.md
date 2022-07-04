---
layout: post
title: "Side Table in Swift"
tags: [iOS, Swift]
comments: true
---

> 약한참조와 Side table  

## Weak, Unowned Reference의 비밀?

Swift의 메모리 관리와 관련한 아티클을 읽다가 흥미로운 것을 발견했다. 메모리 관리에 대해서는 면접 준비하면서 열심히 봐서 나름 제대로 알고 있다고 생각했는데 내가 알고 있던 부분들과 상이한 점들이 보였다.

> So wait… are weak and unowned references the same other than the fact that one is optional and the other isn’t… 🤔.
Well, not really… When we talk about reference count, we usually refer to the strong reference count of the object. Similarly, swift maintains an unowned reference count and weak reference counts for the object. But what really sets weak reference apart from unowned and strong is that weak reference points to something known as a `side table` rather than the object itself. This is why, when a weak reference goes out of scope the object can be de-initialized and deallocated… Since weak reference doesn’t point to the object at all. When the strong reference count reaches zero, the object gets deallocated, `but if the unowned reference count is more than zero, it leaves behind a dangling pointer.`
The side table is a separate chunk of memory that stores the object’s additional information. An object initially doesn’t have a side table entry, it is automatically created when a weak reference is created for the object.

weak, unowned refernece 사이의 차이점을 설명하면서 side table이라는 개념이 슬쩍 나오고 이어서는 unowned reference가 남아있다면 strong reference의 count가 0으로 줄어도 여전히 dangling pointer를 남긴다고한다. weak, unowned refernece의 차이점은 optional 여부일 뿐 strong reference count가 0이되면 메모리에서 정상적으로 deallocated 된다고 생각했는데, dangling pointer가 웬말인가!

무튼 잠시 정리하고 가자면 weak, unowned reference count의 개수와 상관없이 strong reference count가 0이되면 deallocate 된다는 것은 사실이 아니라는 것이다. (실화? 내가 알던 메모리 관리에 대한 개념이 부정당하는 기분) 이 혼란스러움을 해결하려면 deallocate과 deinitialize의 차이에 대해서 짚고 넘어가야한다. 이를 위해 Swift 4부터 weak reference에 적용된 side table 개념을 가져와보자.

## Deinitialize의 진실

Side table은 메모리 관리 향상을 위해 Swift 4부터 추가되었다. 무엇을 향상시켜야했는지 Swift 4이전의 메모리 관리 개념부터 살펴보자.

- Strong reference는 인스턴스를 강하게 참조하여 strong reference count가 1이상으로 남아있다면 인스턴스는 deallocate되지 않는다.
- Weak reference는 인스턴스를 강하게 참조하지 않아 ARC가 참조된 인스턴스를 dispose하는 것을 막지 않는다.
- Unowned reference는 weak reference와 마찬가지로 인스턴스를 강하게 참조하지 않는다. 단, unowned reference count가 0일 때 접근 시 런타임 에러가 발생한다.

기존의 개념은 현재 내 머리속에 들어있는 내용과 별 다를 바가 없는 듯 하다. 그럼 도대체 뭘 향상시켰고 그 과정에서 side table이 어떻게 쓰였으며 위에서 언급한 strong reference count가 0이어도 unowned reference count가 1 이상이라면 왜 dangling pointer가 남는지 그림으로 이해해보자.

id, email 프로퍼티를 가진 User라는 클래스가 있다.

```bash
class User {
    let id: Int
    let email: String
}
```

이 클래스의 인스턴스를 강하게 참조하면 아래와 같다.

![Untitled](https://user-images.githubusercontent.com/35067611/128985902-d8040055-404a-4fd2-875f-cdd11f48ea6f.png)

약한참조를 사용하면 아래와 같다.

![Untitled 1](https://user-images.githubusercontent.com/35067611/128985880-b4b2722c-6b9d-40cd-8cf8-910ee9cbee49.png)

weak reference는 1이지만 strong reference가 0이므로 ARC에 의해서 객체가 메모리에서 삭제될 것이다. 여기서 deinitialized라는 단어에 주목하자. **객체가 `deinitialize` 되었다는 것이 메모리가 객체를 저장해둔 메모리가 `deallocate`되었다는 뜻이 아니다!!** 바로 여기서부터 모든 오해가 시작된 듯 하다. Mike Ash는 Friday Q&A에서 이 현상을 "the object is destroyed but its memory is not deallocated. This leaves a sort of zombie object sitting in memory, which the remaining weak references point to." 라고 설명한다.

그렇다면 이 좀비객체를 갖고있는 메모리는 언제 deallocate될까? Weak reference가 로드되면 좀비 객체가 있는지 런타임에 체크한다. 만약 있다면, weak reference를 0으로 만들고 그 때 비로소 객체의 메모리가 deallocate된다. 이는 결국 좀비 객체는 자신을 참조하는 weak reference가 모두 access된 후에야 사라질 수 있음을 의미한다. 결론적으로 strong reference count가 0에 도달하여 객체가 deinitialize 됐을지라도 memory deallocation은 일어나지 않았으며 좀비객체는 꽤 오랜시간동안 살아있을 수 있다는 것이다.

바로 이런 문제점을 해결하기 위해 `Side table`이 등장한다.

## Side Table

Side table은 객체의 추가적인 정보를 저장하기 위한 분리된 메모리다. Side table 자체가 optional이며 처음에는 존재하지 않다가 객체가 weak reference에 의해 참조되면 생성된다.

![Untitled 2](https://user-images.githubusercontent.com/35067611/128985891-addf923f-43cb-4049-94fa-67f7c52d1ede.png)

위 그림을 보면 strong, unowned reference는 객체 자체를 참조하고 있으나, weak reference는 side table을 참조한다. 이것이 바로 위에서 언급한 메모리 관리 향상을 위한 side table의 사용이며 weak와 unowend refernece의 매우 중요한 차이점이다.

![Untitled 3](https://user-images.githubusercontent.com/35067611/128985897-9e988be4-99a3-4834-a514-bee406fb58b1.png)

Swift 4부터 weak refernece는 side table을 참조하며 이러한 특성때문에 **strong reference count가 0에 도달하면 weak reference count와 상관없이 `object deinitialize`, `memory deallocate`가 모두 일어나게된다.**

## 결론

그동안 weak와 unowed의 차이점은 optional 여부라고 생각했다. 이런 차이점은 Swift라는 언어로 프로그램을 짤 때 주의해야할 점이었다. 하지만 더 로우 레벨에는 각각의 참조가 실제로 객체를 어떻게 참조하는지 방식에도 차이가 있었고, 이런 차이 때문에 strong reference count가 0이 되었을 때 즉시 메모리가 해제되기도하고 좀비 객체로 남기도 한다.

아니 이렇게 중요한 차이점을 몰랐다니 뒤통수를 세게 맞은 느낌이다. (나만 몰랐던건가싶어서 여기저기 "이거 알고 있었어!?" 물어봤다..)그런데 그도 그럴것이 Swift Language Guide에도 unowned reference 설명 초입에 weak와의 공통점과 차이점에 대해서 아래와 같이 짧게 설명하고 넘어간다. 이런 배경설명 링크라도 좀 달아주지..

> Like a weak reference, an unowned reference doesn’t keep a strong hold on the instance it refers to. Unlike a weak reference, however, an unowned reference is used when the other instance has the same lifetime or a longer lifetime.

오랜만에 글감이 생겨 개발관련 글을 써봤다. 또 언제 다시 돌아올지 모르겠지만 오늘은 여기까지~

## References

[Memory Management in Swift with ARC](https://levelup.gitconnected.com/memory-management-in-swift-with-arc-34f10d6f189a)

[Automatic Reference Counting - The Swift Programming Language (Swift 5.5)](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID52)

[Discover Side Tables - Weak Reference Management Concept in Swift](https://maximeremenko.com/swift-arc-weak-references)

[Friday Q&A 2017-09-22: Swift 4 Weak References](https://www.mikeash.com/pyblog/friday-qa-2017-09-22-swift-4-weak-references.html)
