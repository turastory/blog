# Android Tools - Flipper로 디버깅 편하게 하기

GDG Pangyo 2019에서 Flipper라는 디버깅 도구를 알고 한 번 써봤는데, 정말 괜찮은 것 같아서 여기서 한 번 소개해보려고 합니다.



## Flipper

[Flipper](https://github.com/facebook/flipper)는 모바일 앱 디버깅을 위해 Facebook이 개발한 라이브러리입니다.

PC에서 사용하는 클라이언트와 모바일 플랫폼 SDK로 이루어져있습니다. 클라이언트는 Electron으로 만들었고, Android와 iOS, React Native를 모두 지원한다고 합니다.



## vs Stetho

안드로이드 개발을 조금 해보신 분들이라면 Stetho라는, 크롬 기반의 디버깅 도구가 있다는 걸 알고 계실거에요. Stetho를 써보면 유용한 부분들이 많지만 아무래도 그 형식을 웹 개발 도구에 맞추려다보니 부자연스럽기도 합니다.

[관련 문서](https://github.com/facebook/flipper/blob/master/docs/stetho.md)에서도 이러한 내용을 확인할 수 있는데요. Stetho가 안드로이드 디버깅 툴임에도 불구하고 웹 개발자들을 위한 Chrome DevTools를 기반으로 하기 때문에 굉장히 제한적인 부분이 많았고, 이러한 문제를 해결하기 위해 Flipper를 만들게 되었다고 합니다.

Stetho와 비교했을 때 Flipper가 가지는 이점들을 정리해봤습니다.

- Android뿐만 아니라 iOS도 지원합니다.
- Chrome DevTools가 아니라 Electron으로 만들어진 웹앱을 클라이언트로 사용하기 때문에 기능이 좀 더 자연스럽고 안정적입니다.
- 플러그인이 CLI가 아닌 GUI 기반으로, 개발이 좀 더 어려워진 대신 깔끔하고 접근성이 좋은 GUI의 강점을 활용할 수 있습니다.



#### Stetho는 어떻게 되나요?

Flipper가 Stetho를 계승하기는 하지만, Stetho에 대한 개발이나 지원을 중단하는 것은 아니라고 합니다. 몇 가지 이슈도 있고, Flipper가 오픈 소스로 공개된지 얼마 되지 않았기 때문에 기존에 쓰시던 Stetho를 선호한다면 계속 사용해도 문제는 없을 것 같습니다.



## Plugins

Flipper는 Plugin 형식으로 얼마든지 새로운 기능을 추가할 수 있도록 만들어졌습니다. 물론 개발자들이 일일히 필요한 것들을 구현하기는 어렵기도 하고 번거롭기 때문에.. 기본적으로 제공하는 몇 가지 플러그인들이 있습니다.

### Layout Inspector

가장 기본적인 플러그인입니다. 처음에는 Android Studio의 Layout Inspector를 생각했었는데, 그보다는 크롬 개발자 도구로 웹 사이트 구조를 살펴보는 것에 좀 더 가깝더군요.

기본적으로 뷰의 구조를 살펴보는 것은 물론, 아래와 같이 즉각적으로 View의 attribute를 변경하는 것도 가능합니다.

(오른쪽의 화면은 [scrcpy](https://github.com/Genymobile/scrcpy)라는 스크린 미러링 도구를 사용했습니다.)

![flipper-layout-inspector](/Users/nayoonho/Documents/flipper/flipper-layout-inspector2.gif)



### Navigation 

네비게이션 이벤트를 수집해서 보여주는 플러그인입니다. 애널리틱스의 이벤트 트래킹과 비슷한 기능인 것 같습니다. 아래와 같이 Extension을 하나 만들어 놓고, 몇몇 액티비티의 onCreate()에서 호출하도록 해봤습니다.

```kotlin
fun Activity.sendNavigation(uri: String = this::class.java.simpleName) {
    NavigationFlipperPlugin.getInstance().sendNavigationEvent(
        uri,
        this::class.java.simpleName,
        null
    )
}
```

![flipper-navigation](/Users/nayoonho/Documents/flipper/flipper-navigation.png)

시간 순서대로 보여주는 건 좋은데, 다른 데이터를 담을 수 없다는게 조금 아쉽네요. 그래도 플러그인 구조가 굉장히 단순해서, override해서 얼마든지 원하는 대로 수정할 수 있을 것 같습니다.



### Sandbox

Sandbox 플러그인의 기능은 정말 간단합니다. 클라이언트로부터 앱으로 어떤 데이터를 보내는 단 하나의 기능만을 가지고 있습니다. 테스트용으로 엔드포인트를 바꾸거나, AB 테스트 플래그를 변경하거나 할 때 쓰면 좋겠는데, 여러 값을 한 번에 보낼 수는 없는 게 좀 아쉽네요.

받은 값을 로그로 찍는 기능을 간단하게 만들어봤습니다. 

```kotlin
val client = AndroidFlipperClient.getInstance(this)
val echoStrategy = object : SandboxFlipperPluginStrategy {
    override fun getKnownSandboxes(): MutableMap<String, String>? = null

    override fun setSandbox(sandbox: String?) {
        sandbox ?: return
        Log.d("sandbox", "$sandbox")
    }
}
client.addPlugin(SandboxFlipperPlugin(echoStrategy))
```

여담으로 Navigation 플러그인과 비슷하게 구조가 굉장히 단순해서, 커스텀 플러그인을 만들 때 참고로 삼으면 좋을 것 같아요.



### LeakCanary

메모리 누수를 탐지하는 LeakCanary를 지원한다고 합니다. 들어보기만 하고 제대로 써본 적은 없었는데, 이참에 한 번 써보자 싶어서 찾아봤습니다.

![flipper-leak-canary](/Users/nayoonho/Documents/flipper/flipper-leak-canary.png)

과연 얼마나 나올까 했는데.. 시작부터 말도 안되게 나오네요 :sob::sob::sob:



각 플러그인을 설정하는 방법에 대해서는 [공식 문서](https://fbflipper.com/docs/setup/layout-plugin.html)에 잘 설명이 되어 있어서, 그 쪽을 참고하시면 될 것 같습니다. 이 외에도 Network, Database, Shared Preferences 등의 플러그인들도 있는데, 설정도 간단하고 기능도 예상하는 딱 그대로인지라 따로 적지는 않았습니다.



## Wrap up

새롭고 좋은 라이브러리인 것은 분명하지만, 아직 충분히 성숙하지 않은 만큼 이슈도 많은 것 같습니다. Galaxy S6, Nexus 6P 등 몇 가지 기종에서 연결이 안된다는 이슈도 있는 것 같고, Android App Bundle과 호환이 안된다는 이야기도 들립니다. 더불어서 기본으로 제공하는 플러그인 중 iOS를 지원하지 않는 플러그인들도 꽤나 있는 것 같아 iOS에서는 아직 사용하기 어려울 수도 있겠습니다.

일단 저는 프로덕션에서 apk를 쓰고 있지 않아서 한 번 도입해보고 지켜보려고 합니다. 커스텀 플러그인으로 만들만한 유스 케이스도 떠오르는 게 하나 있어서, 다음 포스팅에서는 그와 관련된 이야기를 해보면 좋을 것 같네요.

보다 자세한 정보는 [GitHub Repo](https://github.com/facebook/flipper)와 [공식 문서](https://fbflipper.com/)에서 확인하실 수 있습니다.