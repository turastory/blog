# Android Jetpack - Jetpack Compose Part 2. Deep dive

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



#### 주의 사항

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

Composable 어노테이션입니다. 이 어노테이션을 붙인 함수는 Compose의 가장 기본이 되는 **Composable Widget**으로써 기능하게 되죠.

```kotlin
@Composable
fun RepositoryWidget() {
    Padding(8.dp) {
        Text("Hello!")
    }
}
```

특이한 점은 일반적인 함수와는 다르게 반환 타입을 따로 정의하지 않는다는 건데요. **compose-plugin-cli** 모듈을 보시면, 다음과 같은 두 개의 클래스가 존재합니다.

```kotlin
class ComposeCommandLineProcessor : CommandLineProcessor
class ComposeComponentRegistrar : ComponentRegistrar
```

이 클래스들은 모두 코틀린 플러그인을 상속받아서 재정의하고 있는데, 이것으로 미루어 보아 내부적으로 컴파일러가 이 Composable 어노테이션이 붙은 함수들을 파싱해서 일종의 컴포넌트 그래프를 만들기 때문에 그런 것 같습니다.

어쨌든 지금 단계에서 기억할 건 다음의 두 가지입니다.

1. **@Composable을 사용해서 위젯을 정의할 수 있다.**
2. **Compose는 코틀린 컴파일러 플러그인을 통해서 구현된다.**



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

위와 같이 정의된 모델은, Composable 위젯에서 사용할 수 있고, **이 모델이 변경될 경우 위젯이 다시 렌더링됩니다.**

이 Model 어노테이션 역시 annotation processing을 쓰지 않고 컴파일러 플러그인을 사용하는 것으로 보입니다.



## API - Effects

### Execution Flow

Compose의 실행 흐름은 두 가지로 나누어집니다.

- **Composition Phase**
  이 단계에서는 어떤 위젯을 그릴 것인지 선언합니다. 선언적 프로그래밍 패러다임에는 **지연 실행**이라는 개념이 있는데, 최대한 실제 실행을 뒤로 미루는 것을 말합니다.



### Base

#### @Composable

Functions with @Composable are contributed to building UI



#### @Model

Mark a class as a model for compoisiton

Composable functions will be recomposed whenever the data in the model changes.

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

LoginScreen is recomposed every time the username and password updates.



#### @Pivotal

Contributes to the identity of Composable function
When it changes, the composable starts a new life - as if it had been removed and recreated.

```kotlin
@Composable
fun UserList(userIds: List<Int>) =
    userIds.forEach { UserRow(it) }

@Composable
fun UserRow(@Pivotal userId: Int) {
    
}
```

Mark userId as Pivotal

-> When userId changes, existing UserRow is not used and new one is created.

#### Frame

프레임이 뭔지는 잘 모르겠는데 어쨌든 이 프레임을 화면에 그려주는 것 같음

-> 이 동작을 Commit이라고 하고..

#### FrameManager

FrameManager - 메인 쓰레드에서 priority frame을 관리한다고 함. FrameManager가 시작되면 메인 쓰레드에는 항상 **열려 있는 프레임**이 있음

어느 프레임에서든지 Model이 변경되면 FrameManager는 현재 프레임을 커밋하고 새로운 프레임을 연다.



### Composable Utilities

#### Component

Composable function과 기능은 같음. Function 대신 클래스로 작성할 수 있다는 점이 다르다.

compose() 함수를 재정의해서 Composition을 명시함

```kotlin
class Counter : Component {
    // This could be regarded as an internal state of Composable function.
    private var counter = 0 // Local state of composable
    var color = Color.RED // parameter of composable
    
    // Body of composable
    override fun compose() {
        Button(text="Increment", onClick={})
    }
}

// Usage
@Composable fun App() {
    MaterialTheme { Counter() }
}
```

#### Key

When given composable **executes more than once during composition**

선언 부분을 보면 Pivotal을 활용하는 걸 볼 수 있음

```kotlin
@Composable
fun Key(@Pivotal key: Any?, @Children children: () -> Unit)
```

예제는 너무 간단함

```kotlin
for (user in users) {
    Key(user.id) { UserPreview(user) }
}

for ((child, parent) in relationships) {
    Key(parent.id to child.id) {
        User(child)
        User(parent)
    }
}
```



### Predefined Effects

#### memo

**positionally memoize** the result of a computation

```kotlin
val focusManager = +memo { FocusManager() }

var key = 1
var calcuation = 1
val calculation = +memo(key) { 100 * ++calculation }

print(calculation) // 200
key = 2 // body of memo() re-invoked.
print(calculation) // 300
```

#### State

Based on **memo**
Locally mutable state - used in context of a composition

```kotlin
// 1 - Using a local immutable.
val count = +state { 0 } // Declaration
count.value // Query
count.value = 1 // Mutation

// 2 - Destructure the State object
val (count, setCount) = +state { 0 } // Declaration
count // Query
setCount(1) // Mutation

// 3 - Delegate to a local mutable variable.
var count by +state { 0 }
count // Query
count = 1 // Mutation
```

#### onActive

execute only once initially after **the first composition is applied** and then will not fire again.

```
@Composable
```

#### onDispose

used to schedule work to be done when the effect leaves the composition

```
으엉
```

#### onCommit

execute callback **every time the composition commits**

```
@Composable
fun Test() {
    log("test start")
    +onCommit { log("commit 1") }
    +onCommit { log("commit 1") }
    log("test end")
}
```

#### Ambient



### SFC - Stateless Function Components

##### KTX Tags

```kotlin
// Normal Invocation
CraneWrapper {
    Text(text="Hello World")
}

// KTX Tags
<CraneWrapper>
    <Text text="Hello World" />
<CraneWrapper/>
```

이게 머시여..? 리액트냐!? 제발 하지마 ㅠㅠ



















