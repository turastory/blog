[Google I/O 2019](https://events.google.com/io/)가 시작되면서, Jetpack에도 많은 변화가 생겼습니다.

그래서 새로 추가된 CameraX, SavedState for ViewModel 등과 앞으로 추가될 Jetpack Compose와 더불어, 지금까지 나온 Jetpack의 여러 컴포넌트들을 리뷰하는 시간을 가져보려고 합니다.



## What's new in Android Jetpack

이번 글에서는 I/O에서 Jetpack의 변경된 사항들과, 새롭게 공개된 컴포넌트들을 간략하게 알아보겠습니다.

### Preferences

앱의 기능과 동작을 수정할 수 있는 설정 화면을 만들고 싶을 때 사용하는 `Preference` API가 Android Q에서부터 deprecated 처리되어서, 이제 Jetpack의 Preference Library를 사용해야 합니다.

[릴리즈 노트](https://developer.android.com/jetpack/androidx/releases/preference)에서 현재 버전을 확인할 수 있습니다. Preference에 대한 자료는 [ADS '18](https://www.youtube.com/watch?v=PS9jhuHECEQ)을 참고하세요.



### SavedState for ViewModel

현재 화면 회전과 같은 Configuration Changes는 ViewModel이 처리할 수 있지만, 시스템에 의해서 프로세스가 죽는 경우에는 Activity의 `onSaveInstanceState()` 콜백을 사용할 필요가 있습니다.

이를 위해서 Activity와 ViewModel 사이의 추가적인 작업이 필요했는데, 이 모듈을 사용하면 생성자를 통해서 savedState에 접근할 수 있습니다.

[릴리즈 노트](https://developer.android.com/jetpack/androidx/releases/savedstate)에서 현재 버전을 확인할 수 있습니다.



### Benchmark

Benchmark는 코드의 성능을 측정하고 결과를 확인할 수 있는 라이브러리입니다. 앱에서 자주 사용되는 CPU 작업에 벤치마크를 적용하면 좋다고 하는데, 자세한 건 직접 테스트해봐야 할 것 같습니다.

Benchmark는 일반적인 Instrumentation Test와 동일합니다. `BenchmarkRule`을 선언하고 필요한 테스트 코드를 작성하면 됩니다. 다음 Code Snippet을 참고하세요.

```kotlin
@RunWith(AndroidJUnit4::class)
class ViewBenchmark {
    @get:Rule
    val benchmarkRule = BenchmarkRule()

    @Test
    fun simpleViewInflate() {
        val context = ApplicationProvider.getApplicationContext()
        val inflater = LayoutInflater.from(context)
        val root = FrameLayout(context)

        benchmarkRule.keepRunning {
            inflater.inflate(R.layout.test_simple_view, root, false)
        }
    }
}
```

[릴리즈 노트](https://developer.android.com/jetpack/androidx/releases/benchmark)에서 현재 버전을 확인할 수 있고, [튜토리얼](https://developer.android.com/studio/profile/benchmark.md)과 [샘플 코드](https://github.com/googlesamples/android-performance)도 있습니다.



### CameraX

CameraX는 Jetpack의 일부로써, 카메라를 좀 더 쉽게 사용할 수 있도록 해주는 라이브러리입니다. 

롤리팝까지의 Backward compatibility와 기기 간 호환성에 초점을 맞춰서 기존에 개발자들이 겪고 있던 카메라 호환 이슈를 많이 해결한 것으로 보입니다.

[릴리즈 노트](https://developer.android.com/jetpack/androidx/releases/camerax)에서 현재 버전을 확인할 수 있습니다. [튜토리얼](https://developer.android.com/jetpack/androidx/releases/camerax)과 [코드랩](https://codelabs.developers.google.com/codelabs/camerax-getting-started/#0)을 확인해보세요.



### Jetpack Compose

개인적으로 가장 기대하고 있는 것 중 하나입니다. 안드로이드의 뷰 시스템은 직관적이지 못한 점이 많습니다. 뷰를 코드가 아닌 XML로 작성해야 한다는 것도 불편한 점 중 하나입니다. 작년에 공개한 Material Design 2.0에 대한 API 지원이 `Flutter`에 비해서 아직도 부족한 부분이 많이 있다는 점만 봐도 그렇죠.

이런 문제를 해결하기 위해서 MVVM, Data Binding 등을 사용하고 있긴 하지만 근본적인 문제를 해결해주지는 않습니다.

`Vue.js`,  `Flutter` 등 선언적으로 UI를 작성할 수 있는 패러다임을 따라 안드로이드에서도 Kotlin으로 UI를 작성할 수 있도록 해주는 `Jetpack Compose` 프로젝트를 진행합니다.


![jetpack-compose.png](https://images.velog.io/post-images/tura/6d586bb0-71a3-11e9-bb4e-7b3f80e8f91c/jetpack-compose.png)

*From Google I/O 2019, What's new in Anrdoid*



`Jetpack Compose`를 사용하면 XML이나 Data Binding과 같은 것들을 사용할 필요가 없다고 합니다. `findViewById`라던지 annotation processing 역시 불필요해집니다.

역시 테스트를 해봐야 알겠지만, Compose를 사용했을 때 Flutter같이 애니메이션이나 Material Component에 대한 지원이 정말 잘 됬으면 좋겠습니다.. 

현재 [공식 문서](https://developer.android.com/jetpack/compose)가 올라와 있고, AOSP의 일부로 [소스](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/ui/README.md)를 확인해볼 수 있습니다. 아직 개발 초기 단계이기 때문에, 실제 Production에서는 쓰지 말라고 하네요!

## Summary

정말 간단하게 소개했는데, 이후 포스팅에서 각각의 주제에 대해서 좀 더 자세하게 알아보겠습니다 :)

Google I/O 세션을 직접 보고싶은 분들은 [여기](https://events.google.com/io/schedule/events/b15cf123-8048-492f-b108-45024581999a)로.