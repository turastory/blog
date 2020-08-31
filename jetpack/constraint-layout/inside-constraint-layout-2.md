# Android Jetpack - Inside ConstraintLayout

`ConstraintLayout`이 무엇이고, 어떻게 사용하는지 궁금한 분들은 [ConstraintLayout](https://velog.io/@tura/android-jetpack-constraint-layout), ConstraintLayout 2.0 포스팅을 참고하세요.

이번 글에서는 `ConstraintLayout` 내부 구현을 살펴봅니다.



https://youtu.be/29gLA90m6Gk?t=281



### How

먼저 구글이 아직 2.0에 대한 소스를 공개하지 않았기 때문에 내부 구현을 알아보기 위해서 디컴파일링한 소스를 분석해야 했습니다.

또한 실제 사용 예를 담은 [예제 코드](https://github.com/googlesamples/android-ConstraintLayoutExamples)와 [릴리즈 노트](https://androidstudio.googleblog.com/2019/05/constraintlayout-200-beta-1.html)의 이야기들을 다수 참고했습니다.



### Overview

전체적인 ConstraintLayout의 구조는 다음과 같습니다.



Solver?

