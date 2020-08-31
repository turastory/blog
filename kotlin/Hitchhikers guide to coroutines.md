# 더 이상 콜백 쓰지 마세요. 코루틴이 있잖아요.

저는 C언어로 개발을 시작했습니다. 처음에는 주로 콘솔에 뭔가를 찍어가면서 프로그래밍이라는 것을 익혔고, 이후에는 임베디드 프로그래밍을 주로 했습니다. Java를 접한 건 지금으로부터 3년 전이고, 코틀린을 접한 것은 2년이 되었지만 제대로 사용하기 시작한 건 아직 1년이 되지 않았어요.

이 글은 이러한 배경을 가진 안드로이드 개발자가 바라본, 코틀린의 코루틴과 동시성에 대한 이야기입니다. 자바스크립트나 C#를 해보셨던 분들은 Promise, async/await에 익숙하실거라고 생각합니다만, 여기서는 코루틴을 제외한 다른 방법에 대한 이야기는 하지 않으려고 합니다. 글이 너무 길어질 것 같기도 하고, 무엇보다 제가 그러한 개념을 잘 알지 못합니다.

글을 작성하기 위해 참고한 글들은 말미에 정리해두었습니다. 여유가 되신다면 꼭 한 번씩 읽어보시기를 권합니다.



### Structured Concurrency 소개

지난 몇 달 간 코루틴을 공부하면서 나온 결론은, 더 이상 콜백을 통해서 비동기 처리를 하면 안된다는 생각이었습니다. 그 중심에는 Structured Concurrency라는 개념이 있습니다.



###### <구조적 프로그래밍의 중요성 강조 - 책: 클린 아키텍처>

우리 말로 "구조적 동시성" 정도로 번역할 수 있는 이 개념은, 다소 익숙한 "구조적 프로그래밍"과 크게 맞닿아 있습니다.

객체지향, 함수형 프로그래밍 등에 익숙한 우리에게 구조적 프로그래밍은 다소 구시대적인 것이 아닌가, 하고 생각할 수 있겠습니ek

밥 아저씨로 널리 알려져 있는 Robert C. Martin의 "클린 아키텍처" 라는 책에서 

















### Problem

























### Solution: Structured Concurrency



코루틴

- 코루틴 이전에는 
  - Callback - executable code passed to other code that is expected to call-back (execute)
    - 다른 함수로 전달되는, 실행가능한 코드
  - Future, Promise - A proxy for a result that is initially unknown, but may known in the future
    - 미래의 값을 표현하는 수단
- Beyond asynchronous
- 
- ddasdasd

흠..



### References

- [Coroutine design proposal](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md)
  코틀린 코루틴의 기원이 되는 design proposal 문서입니다. 구조적 동시성에 대한 내용은 없지만, 코루틴의 기반이 되는 다양한 개념들에 대해서 설명이 아주 잘 되어 있어요. 코루틴 라이브러리 소스와 같이 보면 더 좋습니다 :)
  이후에 기회가 된다면 CoroutineContext, ContinuationInterceptor 등 코루틴 구현 디테일에 대한 글도 작성해볼까 합니다.

- [Wikipedia](https://en.wikipedia.org/wiki/Structured_concurrency)
  구조적 동시성에 대한 소개와 설명이 아주 잘 되어있습니다.
  
- [Martin Sústrik on Structured Concurrency](http://250bpm.com/blog:71)
  구조적 동시성에 관한 기념비적인 저작이라고 생각합니다. 이 글뿐만 아니라 굉장한 통찰력을 엿볼 수 있는 글들이 많아요. 특히 이 글의 경우 이전 포스트인 [Getting rid of state machines](http://250bpm.com/blog:69) 시리즈를 함께 읽어야 이해가 쉽습니다.

- [Notes on structured concurrency, or: Go statement considered harmful](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/)
  저도 정독을 하지는 못했는데요, [번역 버전](https://muchtrans.com/translations/notes-on-structured-concurrency-or-go-statement-considered-harmful.ko.html)이 있어서 가볍게 훑고 지나갔던 것 같습니다. 그림이 많고 구조적 동시성과 구조적 프로그래밍의 관계를 암시적으로 조금이나마 알 수 있었던 글입니다.

- [Roman Elizarov on Structured Concurrency](https://medium.com/@elizarov/structured-concurrency-722d765aa952)
  코루틴 라이브러리 팀의 리드인 Roman Elizarov의 글입니다. 글이 재미있기도 하고, 개념을 짧게 간추려놓아서 Structured Concurrency에 대한 입문으로도 좋고 나중에 읽었을 때 또 다른 느낌이 듭니다.