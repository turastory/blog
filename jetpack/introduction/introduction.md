# Android Jetpack - Introduction

![jetpack-image](/Users/nayoonho/Documents/jetpack/introduction/jetpack-image.png)



2018년 5월, Android Jetpack이 공개되었습니다. 당시 안드로이드 개발을 할 때는 MVVM, Data Binding 등의 개념도 생소한터라 Jetpack에 대해 깊게 파볼 생각을 하지 못했습니다.



그 후로 1년이 지난 지금, Google I/O에서 다시 한 번 Jetpack의 큰 변화가 있었습니다. 새롭게 몇 가지의 컴포넌트가 추가되었고, 특히나 **Kotlin First**라는 슬로건을 내걸면서 코틀린에 대한 지원을 전폭적으로 강화했죠.



Jetpack을 전반적으로 리뷰할 시기가 바로 지금이 아닐까 생각이 들어, **"Android Jetpack의 모든 것을 파헤친다"** 라는 주제로 시리즈를 기획했습니다.




### Rule

Android Jetpack 시리즈에서 **다루는 컨텐츠**는 다음과 같습니다.

- Jetpack 컴포넌트들의 개념 및 사용법
- Jetpack 컴포넌트들의 내부 소스와 구현
- KTX 라이브러리, Coroutine 등 코틀린 관련 기능

Android Jetpack 시리즈에서 **다루지 않는 컨텐츠**는 다음과 같습니다.

- View, 안드로이드 프레임워크. 다만 Jetpack 컴포넌트를 설명하기 위해서 필요한 경우는 언급할 수도 있습니다.
- RxJava, Retrofit 등의 서드 파티 라이브러리.
- MVP, MVVM 등의 아키텍처 패턴. 시리즈의 모든 코드는 MVVM을 베이스로 하고 있고, 읽으실 때 이에 대한 기본적인 지식이 있다고 가정합니다.

또한 Android Jetpack 시리즈는 **매주 일요일 10시에 업로드**할 예정입니다.




### Track

Android Jetpack 시리즈는 두 가지 트랙으로 구성됩니다.



#### Introduction

기술의 개요를 살펴보면서 어떻게 사용할 수 있는지 알아보는 기초 세션입니다. 주로 다음과 같은 것에 중점을 둡니다.

- 빠르게 써먹을 수 있는 방법
- 사용 관점에서 기술에 대한 큰 그림과 개요
- 80% 만큼 써먹을 수 있는 20%의 유용한 지식



#### Inside

내부 구현 및 소스를 살펴보면서 기술의 내부에서 어떤 일이 벌어지고 있는지 알아보는 심층 세션입니다. 주로 다음과 같은 것에 중점을 둡니다.

- 더 잘 써먹을 수 있는 방법
- 구현 관점에서 기술에 대한 큰 그림과 디테일에 대한 요약
- 100%를 채워줄 수 있는 80%의 전문 지식



지식은 나누면 나눌수록 가치가 커진답니다. 저 역시 Jetpack을 탐구하면서 작성하는 글이고, 따라서 도중 어딘가 잘못된 부분이 있을 수 있습니다. 이런 것들을 발견하면 언제든 피드백을 주시기 바랍니다. 본인에게도, 저에게도 좋을거라고 생각합니다!

~~하찮은 초보 안드롱 개발자를 도와주세요..~~

