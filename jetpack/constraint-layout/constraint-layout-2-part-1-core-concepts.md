# Android Jetpack - ConstraintLayout 2.0 Part 1. Core Concepts

> 이 시리즈에서는 MotionLayout을 제외한 ConstraintLayout 2.0의 기능을 소개합니다.
> ConstraintLayout 1.1에 대한 내용은 [이전 포스팅](https://velog.io/@tura/android-jetpack-constraint-layout)을 참고해주세요.

Google I/O 2019와 함께 ConstraintLayout 2.0.0의 베타 채널이 배포되었습니다.
베타 릴리즈가 되면 기존의 함수 이름이 더 이상 바뀌지 않습니다. 즉 실제 프로덕션 환경에서 충분히 사용할만한 수준이 되었다는 뜻이지요.

2.0의 새로운 기능들을 살펴보기 전에, 먼저 핵심적인 개념 몇 가지를 정리해보았습니다.



## VirtualLayout

그 동안 ConstraintLayout에서 추구했던 것 중 하나가 "뷰의 구조를 플랫하게 만드는 것"이었는데, 실제로 구현할 때는 매번 Constraint를 설정하는게 귀찮아서 오히려 계층적으로 구현하는게 편한 경우도 있었습니다.

VirtualLayout은 이런 문제를 해결함과 동시에 평면적인 UI 구조를 유지할 수 있게 하고, 더불어서 기존의 UI 시스템으로는 불가능한 다양한 기능을 제공합니다. VirtualLayout을 알아보기 전에, ConstraintHelper에 대해서 짧게 이야기해볼게요.



### ConstraintHelper

ConstraintHelper는 UI 계층에 포함되지 않으면서, 다른 뷰들에 특정한 동작을 적용할 수 있습니다. ConstraintLayout 1.1에서는 *Guideline*과 *Barrier*, *Group*이 바로 Helper였습니다.

ConstraintHelper를 사용하면 다음과 같은 장점이 있다고 합니다. (구글의 말에 따르면 말이죠..)

- 뷰의 구조를 평면적으로 유지
- 한 뷰에 대해 여러 Helper를 적용 가능



예시를 들기 위해서, 다음과 같은 간단한 동작을 하는 Helper를 직접 만들어보았습니다.

![constraint-layout-helper-example](https://i.imgur.com/9BWmH9B.gif)

그리기 전에 뷰들을 보이지 않도록 설정을 하고, 1초 뒤에 Scale 효과와 함께 보이게 하는 코드입니다.

```kotlin
class TestHelper : ConstraintHelper {
    /* secondary constructors omitted */
    
    private var container: ConstraintLayout? = null

    override fun updatePreLayout(container: ConstraintLayout?) {
        super.updatePreLayout(container)

        getViews(container)?.forEach { view ->
            view.visibility = View.INVISIBLE
        }
    }

    override fun updatePostLayout(container: ConstraintLayout?) {
        super.updatePostLayout(container)

        this.container
            ?.let { this.container = container }
            ?: run {
                getViews(container)?.forEach { view ->
                    view.revealAfter(1000L)
                }
            }
    }
}
```

> revealAfter()는 직접 만든 확장 함수입니다. 중요한 부분은 아니라서 생략했습니다.

위와 같이 ConstraintHelper는 몇 가지 함수를 제공해서 뷰의 그리는 과정을 간섭할 수 있도록 합니다.

- `updatePostMeasure(ConstraintLayout)`
- `updatePreLayout(ConstraintLayout)`
- `updatePostLayout(ConstraintLayout)`
- `updatePostConstraints(ConstraintLayout)`

이렇게 나뉘어있긴 하지만, 일반적으로 `updatePostLayout()`만으로도 충분한 것 같습니다.




### So what is VirtualLayout?

2.0에서는 이러한 Helper의 개념을 확장해서 VirtualLayout을 내놓았습니다. ConstraintHelper와의 차이점이 무엇인지 궁금해서 문서를 조금 찾아봤는데, 명확하게 설명되어 있는 자료가 없어서 소스 코드와 Google I/O 등을 보고 추측을 해보았습니다.

먼저 VirtualLayout은 개념적으로 ConstraintHelper와 동일합니다. 즉, UI 계층에 포함되지 않으면서 다른 뷰들에 영향을 줍니다. 여기에 더불어 다음과 같은 함수를 통해 **뷰의 위치를 정하는 과정을 간섭**할 수 있습니다.

```java
// VirtualLayout.java
public void onMeasure(androidx.constraintlayout.solver.widgets.VirtualLayout layout, int widthMeasureSpec, int heightMeasureSpec)
```

VirtualLayout 중 하나인 *Flow* 역시 이 함수를 오버라이딩하고 있습니다.

```java
// Flow.java
@Override
public void onMeasure(androidx.constraintlayout.solver.widgets.VirtualLayout layout, int widthMeasureSpec, int heightMeasureSpec) {
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    if (layout != null) {
    layout.measure(widthMode, widthSize, heightMode, heightSize);
    this.setMeasuredDimension(layout.getMeasuredWidth(), layout.getMeasuredHeight());
    } else {
    this.setMeasuredDimension(0, 0);
    }
}
```



결론적으로 ConstraintHelper가 뷰의 동작을 캡슐화한다면, VirtualLayout은 **뷰의 위치에 대한 다양한 기능을 캡슐화**한다고 볼 수 있겠습니다.



## Summary

간단하게 정리를 해보면 다음과 같습니다.

- ConstraintHelper는 뷰에 대한 특정 동작을 캡슐화한 것으로, 여려 뷰에 적용될 수 있다.
- VirtualLayout은 ConstraintHelper를 확장한 것으로, 뷰의 위치를 잡는 것을 도와준다.