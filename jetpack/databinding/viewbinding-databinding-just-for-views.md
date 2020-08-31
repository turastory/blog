Google I/O 2019에서 새롭게 소개된, 앞으로 추가될 `ViewBinding`에 대해 알아보겠습니다.



### Problem: Accessing View

![view-access-ways.png](https://images.velog.io/post-images/tura/c2348370-7483-11e9-aa1c-695f58669bee/view-access-ways.png)

안드로이드에서 뷰에 접근하는 방법은 다양합니다. 위의 슬라이드처럼 전통적인 `findViewById`를 사용할 수도 있고, 코틀린을 쓴다면 Android Kotlin Extension을 사용해서 뷰에 쉽게 접근할 수 있습니다.



안드로이드 팀에서는 이런 방법을 비교할 때 다음과 같은 세 가지의 기준을 사용했습니다.

- 코드를 깔끔하게 작성할 수 있는지 (Elegance)
- 컴파일 시간에 문제를 잡아낼 수 있는지 (Compile Time Safety)
- 빌드 속도에 영향을 주지는 않는지 (Build Speed Impact)

슬라이드에서 볼 수 있는 것처럼, 그 중 어느 것도 세 가지를 모두 만족하지 않고 있습니다. 이러한 상황에서 `Why not?`이라는 생각으로 나오게 된 것이, 오늘 소개할 `ViewBinding`입니다.




### Idea: Databinding just for views

전 세계의 수 많은 안드로이드 개발자들 역시 뷰 접근에 대한 좀 더 좋은 방법을 고민하고 있습니다. 안드로이드 팀에서는 많은 수의 개발자들이 **단순히 View에 대한 참조를 얻기 위한 목적으로 데이터바인딩을 사용**하는 것을 발견했다고 합니다.

![databinding-just-for-view-references.png](https://images.velog.io/post-images/tura/c5e057f0-7484-11e9-9a26-a9ae8c917a07/databinding-just-for-view-references.png)

본래 데이터바인딩은 View와 Model을 엮어주는 역할인데, 이렇게 View에 대한 참조를 얻기 위한 목적이라면 **뷰 바인딩이라고 부르는 것이 더 어울리지 않을까요?**



### Solution: ViewBinding

이런 맥락에서 탄생한 것이 바로 `ViewBinding`입니다.



![viewbinding-summary.png](https://images.velog.io/post-images/tura/8b9a5310-7485-11e9-aa1c-695f58669bee/viewbinding-summary.png)

뷰 바인딩은 아직 개발 중이고, Android Studio 3.6에 추가될 예정입니다. Kotlin Synthetic과는 다르게 Compile Time Safety가 100% 지원되고, 데이터바인딩과 호환이 가능하다고 합니다.



간단한 사용 예시를 보면 다음과 같습니다.

![viewbinding-sample.png](https://images.velog.io/post-images/tura/18d8e210-7485-11e9-aa1c-695f58669bee/viewbinding-sample.png)

XML 쪽에서는 기존에 데이터바인딩을 사용할 때 들어가던 `<layout>` 태그가 빠지고, 일반적인 레이아웃과 동일한 모습입니다. 코드 쪽에서는 기존과 전혀 바뀐 점이 없습니다.



### 결론

사용 예시를 통해서 알 수 있듯이, 그 모습이 데이터바인딩과 아주 흡사합니다. 즉, 기존에 데이터바인딩을 통해서 뷰 접근을 하고 있었다면 아주 쉽게 마이그레이션이 가능하다는 이야기입니다.

개인적으로 사용할 때의 편의성때문에 Kotlin Synthetic을 많이 사용했는데, ViewBinding이 추가된다고 하면 미리 준비도 할 겸 DataBinding을 사용하지 않을 이유는 없을 것 같습니다.



영상 참조: https://youtu.be/Qxj2eBmXLHg?t=443