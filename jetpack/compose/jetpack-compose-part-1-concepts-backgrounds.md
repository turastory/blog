# Android Jetpack - Jetpack Compose Part 1. Concepts & Backgrounds

> 이 시리즈에서는 안드로이드의 새로운 UI 툴킷인 Jetpack Compose에 대해서 다룹니다.

이제는 선언적 UI 프로그래밍의 시대가 온 것 같습니다.

웹에서는 이를 위한 **React**라는 아주 훌륭한 UI 프레임워크가 있습니다. 안드로이드에는 **Compose**가 있고, iOS도 이번 WWDC에서 **SwiftUI**를 발표했죠. 뿐만 아니라 요즘 핫한 Flutter는 애초에 선언적 패러다임을 베이스로 하고 있습니다.

아직 개발 초기 단계이긴 하지만, 평소에 안드로이드의 UI 시스템에 굉장히 많은 불만을 가지고 있던 저로서는 Compose의 공개가 너무도 반가웠습니다.

구글이 야심차게 내놓은 Jetpack Compose가 무엇인지, 또 어떤 문제들을 해결하려고 하는지 함께 알아볼까요?



## Declarative UI Programming

본론으로 들어가기 전에, 선언적으로 UI를 작성한다는건 도대체 무슨 말일까요? 이해를 위해서 먼저 **명령형 프로그래밍**에 대해서 알아볼게요.

### Imperative Programming

프로그램에게 여러 명령을 내림으로써 원하는 상태, 혹은 결과를 만들어내는 방식이 명령형 프로그래밍입니다.

```kotlin
val link = createNewLinkText()
val container = binding.container
container.removeAllViews()
container.setBackgroundColor(Color.RED)
container.addView(link)
```

명령형 프로그래밍에서는 원하는 상태를 만들기 위해서 명령문을 사용합니다. 위 예시에서는 container에게 "모든 뷰를 제거", "새로운 뷰를 추가" 등의 명령을 내리고 있습니다.
이러한 방식은 제어 흐름이 명확하다는 장점은 있지만 반대로 어떤 상태에 도달하려는 것인지는 명확하지 않지요.

### Declarative Programming

선언적 프로그래밍에서는 반대로 제어 흐름이 아니라 어떤 상태를 원하는지 서술합니다.

```kotlin
return State(
    backgroundColor = Color.RED,
    children = [
        createNewButton()
    ]
)
```

이런 방식이 제대로 동작하기 위해서는 이전 상태와 다음 상태를 비교하고 전환을 수행하는 기능을 프레임워크가 제공해야 합니다. 리액트가 정확히 이런 방식으로 동작하죠. 상태를 비교하고 상태가 다를 경우 해당 부분만 렌더링합니다.

### Why Declarative?

선언적 프로그래밍이 UI 프로그래밍에 있어서 점점 부상하고 있는 이유는 갈수록 복잡해지고 다양해지는 UI/UX 때문이라고 할 수 있습니다. 특히나 애니메이션같은 부분은 명령형으로 처리하기가 매우 까다롭습니다.

우리는 가장 중요한 **"뷰의 상태"**라는 것에만 신경 쓰고, 디테일한 부분은 프레임워크가 알아서 해주기를 기대하는 것이죠.



## Problems

새로운 기술에는 항상 목적이 있습니다. 기존의 기술로는 해결이 불가능하거나 어려운 문제를 잘 해결하려고 하죠. Compose 역시 마찬가지입니다. 안드로이드 UI 시스템에 도대체 무슨 문제가 있어서 그러는걸까요? 지금도 개발하는데 문제가 없어보이는데 말이죠.

사실.. 문제가 꽤나 많습니다.

1. **안드로이드 프레임워크와 강하게 묶여 있음**
   AAC를 비롯해서 안드로이드에서 제공하는 다양한 androidx 라이브러리들은 저마다 버전이 따로따로 업데이트됩니다. 때문에 개발자는 다른 라이브러리를 신경쓰지 않고 필요한 버전만 가져다 쓸 수 있는 장점이 있죠.
   반면 UI는 SDK와 함께 버전업이 되기 때문에 새로운 기능을 사용하려면 SDK를 올려야 하는데, SDK에는 UI만 있는게 아니라서 쉽사리 버전을 올리기가 어렵습니다.
   대표적인 예시가 마시멜로(6.0)에서 생긴 권한 체크죠. 새로운 기능 좀 써보겠다고 버전 올렸다가 앱 전체를 뜯어 고쳐야하는 일이 생길 수도 있습니다.

2. **복잡한 코드와 클래스 구조**
   간혹 프레임워크를 탐색하다가 한 번쯤은 봤을 *View.java* 소스는 현재 약 3만줄에 육박하고 있습니다. 주석을 감안하더라도 3만은 너무 길어요..
   또 다른 예시로는 특정 뷰에 어울리지 않는 동작들이 있습니다. 일례로 Button이 TextView의 속성을 그대로 가지고 있다보니 버튼 안의 텍스트를 선택하거나 수정하는 등의 동작이 가능합니다.
   영상에서는 이를 *API Regret*이라고 표현했습니다. 이러한 문제들은 1번의 문제로 인한 연쇄적인 문제라고 생각을 합니다. 한 번 릴리즈하고 나서 업데이트가 어렵다보니 코드의 유지보수가 어려워질 수밖에 없는거죠.

3. **곳곳으로 분산된 코드**
   커스텀뷰를 한 번이라도 작성해봤다면 여기에 전적으로 동의할 수밖에 없을겁니다. 조금만 나열을 해봐도, 커스텀뷰에 새로운 속성을 추가하거나 하려면 건드려야하는 곳이 정말 많습니다.

   - CustomView.kt
   - layout_custom_view.xml
   - attrs.xml
   - styles.xml (Optional)

   하나의 커스텀뷰라는 개념적인 대상에 대해서 이렇게 분산되어 있는 코드는, 수정하다가 잠깐 한눈을 팔면 다시 작업을 시작하기 위한 비용이 많이 듭니다.
   이건 개인적인 의견이지만, 코드 하이라이팅도 안되고 일일히 태그로 감싸야 하는 XML로 UI를 작성하는게 진짜 스트레스에요.. -ㅅ-

4. 일관되지 않은 데이터 흐름
   사실 이번 단락의 핵심이라고 할 수 있는 부분입니다. 그동안 얼마나 많은 사람들이 이 문제에 시달렸을까요?
   MVP, MVVM, MVI 등의 프론트엔드 아키텍처는 **"데이터 흐름을 일관되게 유지하는 것"**을 목적으로 하고 있다고 해도 과언이 아닙니다. 이러한 아키텍처들은 대부분 뷰에 상태를 두지 않고, 뷰의 변화를 관찰하는 개체가 뷰의 상태를 관리합니다. 이를 통해서 데이터의 흐름이 한 방향으로 흐르도록 하죠.
   반면 안드로이드 UI 시스템은 이러한 아키텍처들과 상반되는 구조를 가지고 있습니다. 안드로이드의 뷰들은 각자만의 상태를 가지고 있고, 스스로 상태를 제어합니다. 이렇다보니 코드를 정말 깔끔하게 만들지 않으면 UI를 구현할 때나, 디버깅을 할 때 너무나 힘이 듭니다..

보시다시피 꽤나 골치아픈 문제들입니다. 과연 Compose가 이 문제를 어떻게 해결할 수 있을까요?



## Goals

Compose는 앞서 이야기한 문제들과 맞물리는, 몇 가지 목적을 가지고 있습니다.

1. 라이브러리를 프레임워크로부터 분리
   궁극적으로는 지금의 android.widget 패키지의 역할을 Jetpack으로 옮기는 것을 목표로 하고 있다고 합니다.
   UI 자체가 안드로이드 프레임워크와 굉장히 많이 연관되어 있기 때문에 쉽게 가능할 것 같지는 않습니다만, 이렇게만 된다면 새로운 기능을 추가하는 것도, 기존의 API를 리팩토링하는 것도 굉장히 쉬워지죠?
2. 재사용 가능한 UI를 쉽게
   안드로이드에서 UI를 재사용 가능하도록 만들기 위한 방법은 크게 Fragment와 Custom View를 작성하는 것인데, 이 두가지 방법은 모두 어느 정도의 Boilerplate가 존재할 뿐더러 기존에 있는 UI에 새로운 기능을 추가하거나 수정하는게 너무 불편하죠. 별 다른 고민 없이 간단하게 재사용 가능한 UI를 만들 수 있도록 하겠다고 합니다. (*Compose*라는 이름에 주목하세요!)
3. 명확한 상태 관리 및 이벤트 핸들링
   이 부분에서 특히나 많은 기대를 하고 있습니다. 상태 관리와 이벤트 핸들링은 사실상 UI의 전부라고 봐도 무방합니다. 앱의 전체적인 데이터 흐름을 설계하는 관점이 달라질 가능성도 있습니다.

이러한 목적을 염두에 두고, Jetpack Compose에 대해서 본격적으로 알아보겠습니다.



## Jetpack Compose

Compose는 원대한 목표만큼 커다란 프로젝트입니다. Compose의 모든 것을 글로 풀어내는 건..  저도 아직 공부가 부족할 뿐더러 글 하나로는 무리가 있습니다. 여기에서는 다섯 가지 핵심적인 아이디어를 중심으로 Compose를 이야기해보려고 합니다.

- UI as a function
- Composable is composable
- Observable Model
- One-way data flow
- Single source of truth



### UI as a function

함수는 어떤 입력을 받아서 어떤 결과를 반환합니다. 아래의 함수는 입력한 값의 제곱을 반환하는 함수입니다.

```kotlin
fun square(x: Int) {
    return x * x
}
```

같은 아이디어를 UI로 확장해서, **어떤 입력을 받아서 UI 구조를 반환하는 함수**가 있다고 생각해보겠습니다.

```kotlin
fun Greeting(name: String) {
    Text("Hello, ${name}")
}
```

UI를 업데이트하고 싶으면, 이 함수를 호출하면 됩니다.

```kotlin
val user = repo.getUser()
Greeting(user.name)
```

조건에 따라서 다르게 보여주고 싶으면, 조건문을 쓰면 됩니다. 여러 개의 위젯을 보여주고 싶으면, 반복문을 쓰면 됩니다!

```kotlin
repo.getFriends().let { friends ->
    if (friends.isEmpty()) {
        Loading()
    } else {
        friends.forEach { friend ->
            Padding(8.dp) { 
                Text(friend.name)
            }
        }
    }
}
```

이 아이디어가 바로 Compose의 핵심입니다. 함수는 그 어떠한 상태도 가지고 있지 않고, 외부의 어떤 상태도 변경해서는 안됩니다. 그저 자신이 그려야할 UI의 계층을 반환할 뿐입니다. 이런 걸 순수함수라고 하던가요?
또한, Kotlin의 함수로 UI를 표현하기 때문에 ~~거지같은~~ XML에서는 불가능한 **프로그래밍 언어의 다양한 기능을 UI 작성하는 데에도 활용**할 수 있습니다.



### Composable is composable

Compose의 UI 위젯을 쓰기 위해서는 다음과 같이 `@Composable` 어노테이션을 붙여줘야 합니다.

```kotlin
// Mark function as composable
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

이름에서 알 수 있듯이, 이 함수는 다른 Composable 함수를 포함할 수 있습니다. 코틀린 기반의 함수로 UI를 작성하는 건데 이게 불가능하면 안되겠죠?

```kotlin
@Composable
fun RepositoryWidget(repo: RepositoryData) {
    Padding(8.dp) {
        Column {
            Title(repo.name)
            Text(repo.description)
            Row {
                StarWidget(repo.numStar)
                WatchWidget(repo.numWatch)
            }
        }
    }
}
```

다음과 같이 RecyclerView의 기능을 하는 제네릭 위젯을 만들어서 이곳저곳에서 사용할 수도 있습니다!

```kotlin
@Composable
fun <T> ScrollingList(
    data: List<T>
    body: @Composable() (T) -> Unit
) {
    if (cond)
        body(item)
}

@Composable
fun Main() {
    ScrollingList<RepositoryData> { item ->
        RepositoryWidget(item)
    }
}
```

구체적인 사용 예시는 다음 포스팅에서 좀 더 자세하게 이야기할 예정입니다.



### Observable Model

데이터의 변경이 생길 때마다 직접 UI를 업데이트하는 건 꽤나 번거로운 작업입니다. 이런 문제를 해결하기 위해서 우리는 RxJava나 LiveData같은 Observable을 사용해왔죠.

```kotlin
vm.state.observe(this) { state ->
    updateUi(state)
}
```

데이터가 바뀔 때마다 UI를 변경시키기 위해서 Compose에서는 어떤 방법을 사용할까요?
일반적으로 자주 사용하는 LiveData를 쓸 수도 있습니다.

```kotlin
vm.name.observe(this) { name ->
    Greeting(name)
}
```

하지만 그려야할 데이터가 많은 경우엔 각 데이터에 대해서 일일히 LiveData를 만들어줘야 하는 불편함이 있죠.
다음과 같이 data class에 `@Model` 어노테이션을 붙여주면, 값이 변경될 때마다 해당 값을 참조하고 있는 Composable 함수가 다시 컴포즈된다고 합니다.

```kotlin
@Model
data class GreetingData(
    var name: String
)
```



### One-way data flow

Compose에서는 데이터 흐름의 단순화를 위해서 데이터는 위에서 아래로, 이벤트는 아래에서 위로 흐르도록 하는 단방향 구조를 강조하고 있는데, 이 부분에서 React의 느낌을 강하게 받았습니다. 코드를 한 번 보시죠!

```kotlin
@Composable
fun FeedWidget(
    repos: List<RepositoryData>,
    onSelected: (RepositoryData) -> Unit
) {
    ScrollingList(repos) { repo ->
        RepositoryWidget(
            repo,
            onClick = { onSelected(repo) }
        )
    }
}

@Composable
fun RepositoryWidget(
    repo: RepositoryData,
    onClick: () -> Unit
) {
    Clickable(onClick) {
        // use repo to draw UI
    }
}
```

FeedWidget은 UI를 그리는데 필요한 데이터를 받아서, RepositoryWidget에 넘겨주고 있습니다. 반대로 RepositoryWidget은 리스너를 통해서 클릭 이벤트를 위쪽으로 넘겨주고 있습니다.

이러한 단방향 데이터 흐름으로 인해서 UI구조가 단순해집니다. RepositoryWidget은 자신이 어떻게 Compose될지는 생각하지 않아도 됩니다. 단지 받은 데이터를 그리고, 이벤트를 전달하는 것만 생각하면 되죠.

다시 한 번 [관심사의 분리](https://en.wikipedia.org/wiki/Separation_of_concerns)가 얼마나 중요한지 느낄 수 있는 대목입니다.



### Single source of truth

기존의 안드로이드 UI 시스템에서는 각각의 뷰들이 상태를 관리했습니다. 이 때문에 UI의 상태와 데이터의 상태가 달라지는 등의 문제가 생겼죠. 이 **상태를 각각의 뷰가 아니라 앱이 관리**하도록 함으로써, 함수의 순수성이 유지됩니다. 코드를 보시면 이해가 쉬울거에요.

```kotlin
@Composable
fun PhoneInputWidget(data: InputData) {
    EditableText(
        text = data.phoneNumber,
        onChange = { newValue ->
            newValue
                .let(this::filterInvalidPhoneCharacters)
                .let(this::formatPhone)
                .let {
                    data.phoneNumber = it
                }
        }
    )
}
```

첫 번째로, 이 위젯의 텍스트는 항상 `data.phoneNumber`의 값을 보여줍니다. 위젯 자체가 데이터를 가지고 있지 않고, 앱으로부터 (혹은 외부로부터) 데이터를 받고 있는 것을 볼 수 있죠?

두 번째로, 위젯이 데이터를 가지고 있지 않기 때문에 값을 설정하기 전에 다양한 전처리 작업을 수행할 수 있습니다. 지금은 단순한 텍스트 필터링이었지만, 이러한 개념은 얼마든지 확장될 수 있다는 점을 기억하세요! (라디오 버튼이나 스피너 등)



## Wrap up

이렇게 신나게 이야기했지만, 사실 Compose는 아직 Experimental 단계입니다. 알파라도 되면 좋을텐데, 아직 EAP도 아니기 때문에 실제 프로덕션에 적용해보려면 시간이 한참 지나야할 듯 싶습니다.

실제로 지금까지 이야기한 것으로는 해결되지 않은 문제들이 많이 있습니다. 몇 가지를 나열해보면 다음과 같습니다.

- 기존 뷰 시스템과의 호환성
- IDE 지원과 Preview
- 애니메이션
- 스타일 / 테마
- ConstraintLayout

당장 뭔가를 해볼 수 없는 건 아쉽지만, 두근두근하기도 합니다. 기존의 UI 프레임 워크를 대체하는 새로운 라이브러리의 탄생부터 성장을 지켜보고, 운이 좋다면 개발에 참여해볼 수도 있는 기회니까요!

이번 글에서는 Compose의 등장 배경과 목적, 그리고 핵심 아이디어를 살펴보았습니다. 거의 발 정도 담궜다고 할 수 있겠네요.

다음에는 실제로 구글 소스를 가지고 몇 가지 데모를 탐색하면서 Compose의 내부로 좀 더 들어가볼 예정입니다. Compose의 열렬한 지지자로써 이 글을 읽는 여러분들이 Compose의 발전에 관심을 가지고 지켜봐주셨으면 좋겠습니다 :)



### Further Resouces

- [Declarative UI Patterns (Google I/O'19)](https://www.youtube.com/watch?v=VsStyq4Lzxo)
- [AOSP - Jetpack Compose](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/ui/README.md)