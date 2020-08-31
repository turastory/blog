# Android Jetpack - Jetpack Compose Part 2. Deep Dive

> 이 시리즈에서는 안드로이드의 새로운 UI 툴킷인 Jetpack Compose에 대해서 다룹니다.

Compose가 등장한 배경과 그 개념에 대해서 간단하게 알아봤으니, 한 번 써봐야겠죠?

Compose는 아직 pre-alpha 단계이기 때문에, 여타 다른 라이브러리처럼 간단하게 디펜던시를 추가할 수 없습니다. 대신 안드로이드 오픈 소스를 가져와서 직접 빌드해야 합니다.

*이번 글에서 다루는 내용은 현재 개발 단계인 Jetpack Compose의 API와 내부 구현입니다. 추후에 변경될 가능성이 매우 높으니 컨트리뷰트를 하고 싶은 분들이나, 구현 세부 사항이 궁금한 분들에게만 권해드립니다.*



## Try Jetpack Compose

### 1. Setup repo

AOSP에서는 **repo**라는 툴을 사용해서 버전 관리를 합니다. Git과 비슷하지만, 안드로이드에 좀 더 적합하게 만들었다고 해요.

다음 명령어를 통해서 repo를 다운로드할 수 있습니다.

```bash
$ curl https://storage.googleapis.com/git-repo-downloads/repo > repo
$ chmod a+x repo
```

이렇게 하나로 되어 있는 바이너리같은 경우 `~/bin`에 몰아넣어서 관리하면 편합니다.

```bash
$ mv repo ~/bin

# .bashrc or .zshrc
PATH=$HOME/bin:$PATH
```



### 2. Get source

이제 소스를 내려받을 디렉토리를 만들고, repo를 사용해서 소스를 받습니다.

```bash
$ mkdir androidx-master-dev
$ cd android-master-dev
$ repo init -u https://android.googlesource.com/platform/manifest -b androidx-master-dev
$ repo sync -j8 -c
```



### 3. Launch specific version of Android Studio

소스를 내려 받았다면, 다음 위치로 이동해서 안드로이드 스튜디오를 실행하세요. 평소에 사용하는 버전이 아니라 지정된 버전을 사용해야 하기 때문에, `studiow`를 통해서 안드로이드 스튜디오를 실행합니다.

```bash
$ cd androidx-master-dev/frameworks/support/ui
$ ./studiow
```



#### * 주의 사항 *

`frameworks/support`가 아니라 `frameworks/support/ui`에서 실행해야 한다는걸 명심하세요! 버전이 다를 경우에는 아마 다음과 같은 문제가 발생할 거에요.

```
ERROR: Could not set unknown property 'useIR' for task ':ui-android-view:compileDebugKotlin' of type org.jetbrains.kotlin.gradle.tasks.KotlinCompile.
```



### 4. Browse samples

여러 모듈들이 있지만, 몇 가지 중요한 모듈들만 체크해보면 다음과 같습니다.

```
:ui-demos
개발중인 여러 Component들의 예제가 담겨있습니다.

:ui-material:integration-tests:ui-material-studies
머테리얼 디자인 샘플 앱 중 Rally의 구현 예시입니다.

:compose-runtime
Compose의 핵심 API들입니다.
```

이 모듈들을 살펴 보면서 구체적으로 Compose를 어떻게 사용하는지, 그리고 어떻게 동작하는지 알아보겠습니다.

> 이제부터 나오는 내용들은 **compose-runtime** 모듈을 조사하면서 정리한 내용입니다. pre-alpha 단계인 만큼 변경될 여지가 있습니다.



## API - Annotations

Compose에는 3가지의 중요한 어노테이션이 있습니다. 그 중 실질적으로 많이 사용되는 건 앞의 두 가지일 것 같아요. 한 번 보시죠.



### @Composable

먼저 Composable 어노테이션입니다. 이 어노테이션을 붙인 함수는 Compose의 가장 기본이 되는 **Composable Widget**으로써 기능하게 되죠.

```kotlin
@Composable
fun RepositoryWidget() {
    Padding(8.dp) {
        Text(text = "Hello!")
    }
}
```

특이한 점은 일반적인 함수와는 다르게 **반환 타입을 따로 정의하지 않는다**는 건데요. **compose-plugin-cli** 모듈을 보시면, 다음과 같은 두 개의 클래스가 존재합니다.

```kotlin
class ComposeCommandLineProcessor : CommandLineProcessor
class ComposeComponentRegistrar : ComponentRegistrar
```

이 클래스들은 모두 코틀린 컴파일러 플러그인의 클래스들을 상속받아서 재정의하고 있는데, 이것으로 미루어 보아 내부적으로 컴파일러가 이 Composable 어노테이션이 붙은 함수들을 파싱해서 일종의 컴포넌트 그래프를 만들기 때문에 그런 것 같습니다.

지금 단계에서 기억할 건 다음의 두 가지입니다.

1. **@Composable을 사용해서 위젯을 정의할 수 있다.**
2. Compose는 코틀린 컴파일러 플러그인을 통해서 구현된다. 따라서 컴파일 전과 후의 코드는 완전히 다르고, **우리가 일반적으로 알던 코틀린의 실행 흐름과는 다르게 동작할 수 있다.** 



### @Model

Model 어노테이션을 통해서 위젯에서 사용되는 데이터를 정의할 수 있습니다.

```kotlin
@Model
class LoginState(
    var username: String,
    var password: String
) {
    fun login() = Api.login(username, password)
}

@Composable
fun LoginScreen {
    val model = +memo { LoginState() }
    
    EditText(text=model.username, onTextChange={ model.username = it })
    EditText(text=model.password, onTextChange={ model.password = it })
    Button(text="Login", onPress={ model.login() })
}
```

위와 같이 정의된 모델은, Composable 위젯에서 사용할 수 있고, **이 모델이 변경될 경우 위젯이 다시 렌더링됩니다.** 이전 글에서도 Observable Model 이라는 개념으로 한 번 소개한 적이 있었죠.



## API - Effects

*Effect*는 **위젯을 그리는 단계에서 어떠한 코드를 실행하고 싶을 때** 사용합니다. Effect를 설명하기 위해서, 먼저 Compose의 실행 흐름을 이해할 필요가 있습니다.

### Execution Flow

> 여기에 정리된 내용은 코드를 통해서 분석한 내용입니다. 실제 구현 의도와는 다를 수 있습니다.

Compose의 실행 흐름은 두 가지로 나누어집니다.

- **Composition Phase**
  이 단계에서는 어떤 위젯을 그릴 것인지 선언합니다. 선언적 프로그래밍 패러다임에는 **지연 실행**이라는 개념이 있는데, 최대한 실제 실행을 뒤로 미루는 것을 말합니다.
  이렇게 "화면에 렌더링을 한다"라는 실제 실행을 하기 이전에 "**어떻게 그릴 것인가**"를 정의하는 단계가 바로 Composition Phase 입니다.
- **Execution Phase**
  Composition Phase에서 정의했던 Component의 그래프를 화면에 렌더링하는 단계입니다.

다음 코드를 보시죠.

```kotlin
@Model
data class Counter(val number: Int = 0) {
    fun increment(): Counter = 
        Counter(number + 1)
}

@Composable
fun CounterWidget() {
    var counter = Counter()
    Text(text = "Count: ${counter.value}")
    Button(onClick = { counter = counter.increment() })
}
```

CounterWidget에는 **counter**라는 로컬 변수, 즉 **상태**가 존재합니다. Text에서는 그것을 보여주고 Button에서는 그 값을 업데이트하고 있습니다. @Model로 모델 클래스라고 명시해줬으니 잘될 것 같죠? 하지만 실제로 위 코드를 실행해보면 버튼을 눌러도 값이 증가하지 않습니다.

이전에 @Composable 어노테이션을 이야기할 때, 코드가 일반적인 코틀린의 실행 흐름과는 다르게 동작할 수 있다는 이야기를 했죠? Compose의 위젯들은 일종의 **Context 위에서 실행됩니다.** 따라서 위젯에서 필요한 데이터 역시 이 Context 위에 올라가 있어야 하는데 그러기 위해서 **Effect**를 사용할 수 있습니다.

위의 코드는 다음과 같이 **memo** Effect와, 해당 Effect를 resolve하는 **+(unaryPlus)** 연산자 오버로딩 함수를 붙여주면 제대로 동작합니다.

```kotlin
@Model
data class Counter(val number: Int = 0) {
    fun increment(): Counter = 
        Counter(number + 1)
}

@Composable
fun CounterWidget() {
    var counter = +memo { Counter() }
    Text(text = "Count: ${counter.value}")
    Button(onClick = { counter = counter.increment() })
}
```

내부적으로 정확히 어떻게 동작하는지는 모르지만 무척이나 신기하네요..

그나저나 이름이 왜 Effect인가 곰곰히 생각해봤는데.. 다음 Effect 클래스의 주석에 따르면 Effect는 개념적으로 **리턴 값을 가지고 있는 Composable 함수**이고 이게 Compose의 실행 흐름에 **일종의 효과를 적용하는 것**과 비슷해서 Effect라고 이름지은 게 아닐까 조심스럽게 추측해봅니다..

```
 * NOTE: Effects are equivalent to [Composable] functions that are able to have a return value, where the [unaryPlus] operator
 * is the manner in which it is "invoked". Effect will likely go away in favor of unifying with [Composable].
```



### Predefined Effects

몇 가지 미리 정의되어 있는 Effect들이 존재하는데, 그 중 몇 가지를 이야기해볼까 합니다.

#### Memo

바로 이전에 사용했던 memo 입니다. 이 녀석이 하는 일은 별 게 없습니다. 단순히 어떤 실행 블록을 기억하는 역할을 합니다.

```kotlin
fun <T> memo(calculation: () -> T) = effectOf<T> {
    context.remember(calculation)
}
```

calculation 코드 블록에서 반환하는 클래스에 @Model이 붙어있지 않으면 Composition이 다시 이루어져도 이 코드 블록은 재실행되지 않습니다.

다음은 memo에 대한 테스트입니다. **then()**을 통해서 Composition이 두 번 호출되었음에도 불구하고 값이 1로 고정되있는 걸 볼 수 있죠.

```kotlin
@Test
fun testMemoization1() {
    var inc = 0
    
    compose {
        +memo { ++inc }
    }.then {
        assertEquals(1, inc)
    }.then {
        assertEquals(1, inc)
    }
}
```

실제 사용 예를 보면 대부분 실행 단계에서 필요한 어떤 값을 가져오기 위한 목적으로 쓰는 것 같더라구요.

```kotlin
// Wrapper.kt
@Composable
fun CraneWrapper(/* ... */) {
    val focusManager = +memo { FocusManager() }
}
```



#### State

state는 memo와 비슷하지만, 단순히 어떤 코드 블록을 기억하는 목적이 아니라 **위젯에서 사용할 Local State의 개념**을 가지고 있는 Effect입니다. 여기서 사용하고 있는 **State**는 Compose의 모델 클래스이고, 단순히 어떤 값을 wrapping하고 있는 클래스입니다.

```kotlin
fun <T> state(init: () -> T) = memo { State(init()) }

@Model
class State<T> internal constructor(value: T) : Framed {
    /* ... */
}
```

이전에 작성했던 카운터 예제를 State를 이용하면 더 편하게 구현할 수 있습니다.

```kotlin
@Composable
fun CounterWidget() {
    var counter = +state { 0 }
    Text(text = "Count: ${counter.value}")
    Button(onClick = { counter++ })
}
```



#### Lifecycle Effects

그 외에 세 가지 Composition의 라이프사이클에 관련된 Effect가 존재합니다.

```kotlin
@EffectsDsl
interface CommitScope {
    fun onDispose(callback: () -> Unit)
}

fun onActive(callback: CommitScope.() -> Unit) = effectOf<Unit> {
    context.remember { PostCommitScopeImpl(callback) }
}

fun onDispose(callback: () -> Unit) = onActive { onDispose(callback) }

fun onCommit(callback: CommitScope.() -> Unit) = effectOf<Unit> {
    context.changed(PostCommitScopeImpl(callback))
}
```

- **onActive** - **Composition이 처음 실행될 때** 호출되고, 그 이후로는 호출되지 않습니다.
- **onDispose** - 정확한 시점을 알아볼 수는 없었지만, 테스트 코드를 통해 유추해보았을 때 **해당 위젯이 더 이상 화면에서 보이지 않을 때** 호출되는 것 같습니다.
- **onCommit** - Composition이 실행될 때마다 호출됩니다.

Activity, Fragment 등의 라이프 사이클을 가지고 있는 녀석들을 많이 다뤄보신 분들이라면 각각의 Effect가 무엇을 위해서 존재하는지 감이 오실거에요. 그다지 어려운 개념은 아니죠?



## Wrap up

이렇게 Compose의 코드를 대략적으로나마 탐색을 해봤습니다.

Compose는 단순한 라이브러리가 아닙니다. annotation processing을 사용하지 않고 **kotlin compiler plugin**을 사용하기 때문에 Java에서는 사용이 불가능한 반면, 내부적으로 LLVM IR을 생성해서 멀티 플랫폼으로 확장될 가능성도 있습니다. (Kotlin의 컴파일러 플러그인에 대해서 궁금하신 분들은 [Kevin Most의 이야기](https://www.youtube.com/watch?v=w-GMlaziIyo)를 들어보시면 좋을 것 같아요. 저도 조만간 봐야 합니다..)

더불어서 코드를 훑어보면서 굉장히 우아하게 짜여있다는 느낌을 계속 받았어요. 특히 테스트가 일일히 다 작성되어 있고, 테스트를 위한 DSL을 직접 만들어서 사용하는 것에서 감동.. 덕분에 코드를 보기도 편했고, 클래스나 함수가 무슨 일을 하는지도 쉽게 알 수 있었습니다.

다만 조금 아쉬웠던 것은 각 클래스들의 역할이나 기능을 명확하게 알기 어려웠다는 것인데요, 아직 개발 단계인 만큼 클래스의 이름이나 기능 명세같은 것들은 충분히 개선될 여지가 있겠죠?

Compose에 대한 이야기는 여기서 마무리를 지으려고 해요. 개인적으로 조금 더 파보겠지만, 깊은 수준의 내용을 설명하기에는 제가 스스로 감당하지 못할 것 같기도 하고, 아직 개발 단계인 만큼 얼마든지 바뀔 수 있기 때문에 큰 의미가 없을 것 같습니다.

다음에 봐요!