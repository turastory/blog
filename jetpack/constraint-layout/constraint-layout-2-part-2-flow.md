# ConstraintLayout 2.0 Part 2. Flow

> 이 시리즈에서는 MotionLayout을 제외한 ConstraintLayout 2.0의 기능을 소개합니다.
> ConstraintLayout 1.1에 대한 내용은 [이전 포스팅](https://velog.io/@tura/android-jetpack-constraint-layout)을 참고해주세요.

지난 포스팅에서 ConstraintLayout 2.0의 기본적인 개념을 살펴보았습니다. 이번 포스팅에서는 이러한 기본적인 개념을 숙지한 상태로, 새롭게 추가된 **Flow**에 대해서 알아보겠습니다.



### Adding Depdencies

```groovy
dependencies {
    implementation "androidx.constraintlayout:constraintlayout:2.0.0-beta1"
}
```

AndroidX를 사용하지 않는 분들은 아래 링크에서 Dependency를 추가해주세요.
https://androidstudio.googleblog.com/2019/05/constraintlayout-200-beta-1.html



## Flow

**Flow**는 *CSS의 FlexBox*와 비슷한 형태의 동작을 가능하게 하는 **VirtualLayout**입니다.
ViewGroup이 아니라 VirtualLayout이기 때문에 뷰의 계층과는 무관하게 작동하고, 이로 인해 다양한 것들이 가능해졌습니다. 하나씩 살펴보실까요!



### Basics

Flow의 핵심은 기존의 ConstraintLayout에서 다소 복잡한 형태로 적용되었던 **Chain의 간소화**입니다.
기존의 Chain을 보면, 사실 그 어디에도 chain이라는 단어는 없습니다. (style을 지정하지 않았을 때는요!) 더군다나 chain의 스타일을 설정하고 싶을 때는 어디에 설정해야 하는지 헷갈리기도 하죠.

```xml
<Button
  android:id="@+id/button1"
  app:layout_constraintHorizontal_chainStyle="packed"
  app:layout_constraintLeft_toRightOf="@+id/button2"
  app:layout_constraintRight_toRightOf="parent"/>

<!-- button2만 놓고 보면 여기에 chain이 걸려있는지 도무지 알 방법이 없다. -->
<Button
  android:id="@+id/button2"
  app:layout_constraintRight_toLeftOf="@+id/button1"
  app:layout_constraintLeft_toLeftOf="parent"/>
```



Flow에서는 원하는 Chain의 형태를 지정해서 이전과 비교했을 때 보거나 쓰기가 편합니다.

```xml
<androidx.constraintlayout.helper.widget.Flow
  android:layout_width="0dp"
  android:layout_height="0dp"
  android:orientation="vertical"
  app:constraint_referenced_ids="button_one,button_two,button_three"
  app:flow_horizontalAlign="start"
  app:flow_verticalStyle="packed"
  app:flow_verticalGap="16dp"
  app:layout_constraintBottom_toBottomOf="parent"
  app:layout_constraintEnd_toEndOf="parent"
  app:layout_constraintStart_toStartOf="parent"
  app:layout_constraintTop_toTopOf="parent" />
```



#### 방향 설정

Chain의 방향을 설정할 때는, LinearLayout의 방향을 설정하는 것과 동일하게 `android:orientation`을 이용합니다.

```xml
android:orientation="horizontal|vertical"
```



#### Chain 스타일 지정

Chain의 스타일에는 세 가지가 있었죠. Flow에서도 이 세 가지 스타일을 지원합니다.

```xml
app:flow_verticalStyle="packed|spread|spread_inside"
app:flow_horizontalStyle="packed|spread|spread_inside"
```



#### Bias

Chain의 스타일을 `packed`로 했을 경우, Chain 내에서의 위치를 bias로 설정할 수 있습니다. 기존의 bias와 비슷하지만, 체인의 형태를 유지하면서 bias가 적용된다는 점이 다릅니다. 마치 레이아웃으로 묶인 것처럼 적용되죠.

```xml
app:flow_verticalBias="float"
app:flow_horizontalBias="float"
```

![constraint-layout-bias](https://i.imgur.com/Yvy6yUV.jpg)



#### Gap

LinearLayout, RelativeLayout 등에서 Element 사이의 거리를 둘 때는 `margin`을 사용했습니다. Flow에서는 개별 뷰에 대해서 margin을 설정하는 것은 불가능하고, 각각의 뷰 사이의 `gap`만을 지정할 수 있습니다. (혹시 가능하다면 알려주시기 바랍니다)

```xml
app:flow_verticalGap="dimension"
app:flow_horizontalGap="dimension"
```

![constraint-layout-gap](https://i.imgur.com/t3MZfLH.jpg)



#### Align

LinearLayout 등에서는 뷰들을 어떻게 정렬시킬지를 `gravity`로 지정했습니다. Flow에서도 그와 같은 기능을 지원합니다. 아래 예시와 같이 뷰의 크기가 다르거나, 가변적으로 바뀌어야할 때 유용하게 사용할 수 있습니다.

```xml
app:flow_horizontalAlign="start|end|center"
app:flow_verticalAlign="top|bottom|center|baseline"
```

![constraint-layout-align](https://i.imgur.com/lE1zp2M.jpg)



### WrapMode

Flow는 참조하는 뷰들을 어떻게 다루는지에 따라 세 가지 방식으로 동작합니다. 이 모드를 통해서 CSS의 FlexBox와 비슷한 동작이 가능해졌다고 볼 수 있습니다. 다음과 같은 방법으로 wrapMode를 적용할 수 있습니다.

```xml
app:flow_wrapMode="none|chain|aligned"
```



#### wrapMode: none

Flow의 기본 동작이고, 단순히 참조하는 뷰들 사이에 Chain을 만듭니다. 앞서 보여드렸으니 따로 설명하지는 않겠습니다.



#### wrapMode: chain

기본 모드와 비슷하지만 공간이 부족할 경우 여러 줄에 걸쳐서 뷰를 배치합니다. CSS FlexBox의 **flex-wrap: wrap**과 같은 동작이라고 볼 수 있습니다.



##### First style/bias

Chain 모드에서 첫 번째 체인의 배치를 정의할 수 있는 속성들이 존재합니다.

```xml
app:flow_firstHorizontalStyle="packed|spread|spread_inside"
app:flow_firstVerticalStyle="packed|spread|spread_inside"
app:flow_firstHorizontalBias="float"
app:flow_firstVerticalBias="float"
```

![constraint-layout-first-style](https://i.imgur.com/OxqXoJf.jpg)

이 속성들을 어떤 경우에 활용할 수 있을지는 아직 잘 모르겠습니다. 첫 줄만 다르게 스타일을 지정해야 하는 경우가 무엇이 있을까요?



##### Max elements

각 줄마다 표시할 뷰의 개수를 지정하고 싶을 경우도 있을텐데, 그럴 때는 다음 속성을 사용하면 됩니다.

```xml
app:flow_maxElementsWrap="integer"
```

![constraint-layout-max-elements-wrap](https://i.imgur.com/UGKJuPV.jpg)



#### wrapMode: aligned

Aligned 모드에서는 아이템들이 마치 테이블처럼 정렬되어서 나타납니다. 다른 것들은 Chain 모드와 동일합니다. 이제 테이블 형식의 레이아웃을 짜고 싶을 때 훨씬 더 편하게 만들 수 있겠군요.

![constraint-layout-mode-aligned](https://i.imgur.com/NbbakhM.jpg)



## Downsides

Flow는 계층적인 뷰 구조를 만들기 위해서 사용했던 기존의 ViewGroup을 대체할 수 있습니다. 실제로 이 포스트를 작성하면서 사용한 여러 예시들 역시 Flow만을 사용해서 작성했습니다. 하지만 개인적으로 Flow를 사용하면서 불편한 점이 몇 가지 있었는데, 여기에 한 번 적어볼까 합니다.

- **XML을 보기가 상대적으로 어렵습니다.**
  이건 순전히 제 개인적인 의견이긴 합니다만, Flat Hierarchy로 되어있다보니 실제로 레이아웃을 수정할 때는 어디를 수정해야할 지 살짝 헷갈리곤 합니다. 다양한 ConstraintHelper, VirtualLayout을 적용할 수 있다는 점은 좋지만, 반대로 XML의 가독성에는 그다지 좋지는 않은 것 같습니다.

- **섬세한 레이아웃 작업에는 적합하지 않습니다.**
  Gap과 같은 기능을 보면 아시겠지만, 참조하고 있는 **개별 뷰에 대한 마진을 설정하는 것이 불가능**합니다. 즉, 모든 뷰가 같은 Gap을 가져야 하기 때문에, 레이아웃 전체를 Flow로 구성하는 것은 조금 어려울 수도 있습니다.
  기존에 ViewGroup으로 묶어서 처리하고 있는 부분들만 먼저 Flow로 바꿔보는 것을 권해드립니다.

- **IDE 지원이 아직 미흡합니다.**
  Flow와 같은 VirtualLayout을 사용할 때는, 개별 뷰에서 constraint를 적용하지 않는데 이 때 Android Studio 3.4에서는 다음과 같이 warning을 표시합니다. 지원이 미흡하다기보다는 **정적 분석이 조금 아쉽다**고 하는게 더 맞겠네요.

  ![constraint-layout-lint-warning](https://i.imgur.com/YUSMrMi.jpg)



## Summary

뭐 이런 불편한 점은 차치하고, 확실히 유용한 기능임에는 틀림 없습니다. 새로 만드는 앱에서 뿐만 아니라 기존 앱에서도 한 번쯤 시도해볼만 합니다 :)

다음 포스팅에서는 아직 다루지 않은 ConstraintLayout 2.0의 다른 기능들을 소개할 예정이니 기대해주세요!