# Animation + Coroutine + DataBinding = ?

>  애니메이션, 코루틴, 데이터 바인딩을 섞으면 뭐가 나올까?





최근에 presentation 레이어에서 사용하는 아키텍처를 어느 정도 정리하게 되면서, View의 상태를 처리하는 대부분의 작업을 ViewModel과 XML로 옮겼습니다. 그런데 문제가 된 게 하나가 있어요. 바로 애니메이션!!





MVP, MVVM 등의 아키텍처들이 추구하는 방향이 상태 관리를 한 곳에서 하는 것이다보니, 라이프사이클과 상태 관리는 편해진 반면 애니메이션같이 순수 뷰 영역의 동작은 많이 소홀해진 것 같습니다.

[구글의 Nick Butcher 역시](https://medium.com/androiddevelopers/motional-intelligence-build-smarter-animations-821af4d5f8c0) 이것에 대해 지적한 바 있습니다. 데이터는 **Stateless**한 반면, 애니메이션은 **Stateful**하다구요. 해당 포스트에서는 세 가지 애니메이션의 속성(Reentrant, Continuous, Smoothness)을 이야기하면서, Stateless한 데이터를 가지고 애니메이션을 안정적으로 구현하는 방법을 소개하고 있습니다.





데이터바인딩은 흔히 이런 식으로 많이 사용하죠. 일반적으로 ViewModel과 같은 컨트롤러가 뷰의 상태를 들고 있고, XML 상에서 이 데이터가 바인딩되는 방식입니다.

```xml
<TextView
    android:id="@+id/tv_title"
    android:text="@{state.titleText}"
    android:textColor="@{state.titleTextColor}"
    ... />
```



여기서 애니메이션이 들어간다고 하면 어쩔 수 없이 뷰의 상태를 변경하는 코드가 액티비티같은 곳에 들어가게 되요.

```kotlin
vm.state.observe(viewLifecycleOwner) { state ->
    binding.tvTitle.animateTextColor(to = state.titleTextColor)
}
```







애니메이션을 단순히 실행만 하는거라면 데이터바인딩으로 처리해도 문제 없지만, 









"""

사실 애니메이션 자체를 견고하게 설계해놓으면 이런 라이프사이클 처리는 크게 문제가 되지는 않습니다.

애니메이션은 일반적으로 View의 상태를 스무스하게 바꾸기 위해서 사용하는데,
애니메이션이 도중에 중단되는 경우는 아래 두 가지가 전부입니다.

1. View가 더 이상 보이지 않거나 사라지는 경우
2. 유저 액션으로 인해 새로운 애니메이션을 실행해야 하는 경우

여기서 View의 상태란, 사용자가 인지할 수 있는 UI 상의 상태를 의미합니다. 뷰가 있거나/없거나, 혹은 로딩 중이거나, 에러 메시지가 표시됬거나 하는 것들이요.







"""











## Coroutine

그러다 코루틴이 나오고, 기존에 비동기 처리하는 부분을 Rx에서 코루틴으로 갈아 치웠습니다.



데이터바인딩을 쓰다 보면 항상 아쉬운게 애니메이션입니다. 





ViewModel을 사용하면 뷰의 상태 관리는 쉬워지는 반면, 뷰에서 어떤 동작을 하고 싶을 때 곤란한 상황이 많이 나옵니다.



자, 이제 한 걸음

