## Motion Layout

### Motion Layout

#### What is it

Property Animation + TransitionManager + CoordinatorLayout



[http://pluu.github.io/blog/android/2019/04/07/droidknights-motionlayout/](http://pluu.github.io/blog/android/2019/04/07/droidknights-motionlayout/)



- 선언적임
  모든 transition은 XML로만 표현 가능
- IDE 지원
  안드로이드 스튜디오에서 Motion Layout 



#### Where to use it

**유저와의 상호작용**이 발생하는 곳에 사용.

GIF, lottie animation 등 유저가 직접적으로 상호작용하지 않는 부분에는 필요 없음



#### Dependencies

5월 9일 alpha 스테이지가 끝나고 beta 버전이 릴리즈 되었음

버전 릴리즈는 [이곳](https://androidstudio.googleblog.com/2019/05/constraintlayout-200-beta-1.html)을 참조

```
dependencies {
    implementation 'androidx.constraintlayout:constraintlayout:2.0.0-beta1'
}
```





그냥 자유롭게 끄적여보자



MotionLayout의 상세 스펙은 MotionScene에 있음.

MotionLayout은 View에만 집중

position이나 movement는 MotionScene으로 delegate



ConstraintSet - Layout의 모든 positioning 정보를 캡슐화

-> 여러 개의 ConstraintSet을 이용해서 뷰를 다시 만들지 않고 언제든지 Position과 Dimension을 바꿀 수 있음.

TransitionManager와 같이 사용해면 상대적으로 쉽게 애니메이션을 만들 수 있음. MotionLayout은 이런 아이디어에서 출발해서, 좀 더 확장한 형태



MotionScene - A Specification for MotionLayout

`res/xml` 에 저장됨. 

![img](https://cdn-images-1.medium.com/max/1600/1*CwwhnKdE7CpV_txpQ7pP6Q.png)

이런 것들을 포함

- the ConstraintSets used
- the transition between those ConstraintSets
- keyframes, touch handling, etc.





1. Reference Existing Layout

![img](https://cdn-images-1.medium.com/max/1600/1*6iOoPkHl13ap6cluzoYKmg.gif)

##### 이런걸 만들어보자!

먼저 시작 상태와 끝 상태에 해당하는 2개의 레이아웃 정의.



ConstraintLayout - 2개의 레이아웃으로부터 ConstraintSet을 만든 다음 적용

-> 이 방법은 Transition이 시작되면 중간에 멈출 방법이 없음.

-> 게다가 미리 정해놓은 것만 가능. 유저의 액션에 반응할 수가 없음.



MotionLayout

그냥 뷰 아무데나 하나 만들고 다음 속성 정의. 어차피 위치는 MotionScene에서 결정함.

```xml
app:layoutDescription="@xml/scene_01"
```

여기서 `scene_01`이 우리의 MotionScene이 담긴 파일.

덤으로 `showPaths` 속성을 사용하면 경로가 보임!!

```xml
app:showPaths="true"

<!-- For Preview -->
tools:showPaths="true"
```



옛다 샘플 코드

```xml
<MotionScene
    xmlns:motion="http://schemas.android.com/apk/res-auto">

    <Transition
        motion:constraintSetStart="@layout/motion_01_cl_start"
        motion:constraintSetEnd="@layout/motion_01_cl_end"
        motion:duration="1000">
        <OnSwipe
            motion:touchAnchorId="@+id/button"
            motion:touchAnchorSide="right"
            motion:dragDirection="dragRight" />
    </Transition>

</MotionScene>
```

여기서 `motion` 네임스페이스는 별 다른거 없고 그냥 `app`이라고 보면 됨. MotionLayout이라는 걸 강조하려고 motion이라고 쓴 듯.



Transition의 정의. 시작과 끝 레이아웃, 그리고 시간을 지정하는 듯 함.

Transition이라 함은 말 그대로 한 상태에서 다른 상태로의 전이를 나타내니까 뭐..

```xml
<Transition
    motion:constraintSetStart="@layout/layout_start"
    motion:constraintSetEnd="@layout/layout_start"
    motion:duration="1000">
```



일단 이 녀석의 이름은 OnSwipe Handler.

우리의 손가락으로 Transition을 컨트롤할 수 있게 해주는 역할이라는데..

```xml
<OnSwipe
    motion:touchAnchorId="@+id/button"
    motion:touchAnchorSide="right"
    motion:dragDirection="dragRight" />
```



`touchAnchorId`: 대상 뷰 ID

`touchAnchorSide`: 내 손가락을 따라올 대상 뷰의 Side? 어느 쪽으로 움직일 것인지 말하는 것 같은데.. (`right / left / top / bottom`)

`dragDirection`: 모션의 방향? 이 속성이 progress 값이 어떻게 설정될지를 결정짓는다고 함.

-> 좀 테스트를 해보다가 발견한건데 dragRight - 화면을 오른쪽으로 움직이면 progress가 증가함. 0 -> 1로 Progress를 증가시키는 방향을 의미하는 듯



OnSwipe Handler.. 이게 좀 수상한데..

소스를 까서 보니 일단 실제로 CustomViewr가 있는 건 아니고 아래와 같이 Styleable로 정의되어 있는 형태. -> 실제 View가 없어도 Styleable이 정의되어 있으면 XML에서 Reference가 가능하다??

```xml
<declare-styleable name="OnSwipe">
    <attr name="dragScale" format="float" />
    <attr name="maxVelocity" format="float" />
    <attr name="maxAcceleration" format="float" />
    <attr name="dragDirection" />
    <attr name="touchAnchorId" />
    <attr name="touchAnchorSide" />
    <attr name="touchRegionId" format="reference" />
    <attr name="moveWhenScrollAtTop" format="boolean" />
    <attr name="onTouchUp" format="enum">
        <enum name="autoComplete" value="0" />
        <enum name="autoCompleteToStart" value="1" />
        <enum name="autoCompleteToEnd" value="2" />
        <enum name="stop" value="3" />
        <enum name="decelerate" value="4" />
        <enum name="decelerateAndComplete" value="5" />
    </attr>
</declare-styleable>
```

참고: onTouchUp은 터치가 끝났을 때 어떻게 동작할지를 결정지음. 기본은 autoComplete인 것 같은데, stop으로 하면 멈춤.

혹시나 해서 찾아봤는데, MotionScene도 있음.

```xml
<declare-styleable name="MotionScene">
    <attr name="defaultDuration" />
</declare-styleable>
```

직접 만들어봤는데 아무거나 되는 건 아니고 뭔가 설정이 필요한 듯?



MotionScene 소스를 보면, 자체적으로 Tag를 파싱하는 코드가 있는데 여기에서 String을 비교해서 OnSwipe에 해당할 경우 TouchResponse라는 클래스로 XML을 넘김. 느낌이.. 코드 정리가 안된건지 뭔지.. 하 이거 코드 공개될때까지 기다려야 하나

어쨌든 MotionScene 자체는 View도 아니고 그냥 일반적인 클래스



2. Self contained

1번은 기존에 있는 뷰들과 MotionScene을 연결하는 거였다면, ConstraintSet을 MotionScene 안에 직접 넣을 수도 있음.

이러면 몇 가지 장점이 있다고 하는데,

- 여러 ConstraintSet을 한 파일에서 관리
- 기존의 ConstraintSet에 비해 몇 가지 추가된 속성이 있음??
- 앞으로 나올 안드로이드 스튜디오의 MotionEditor가 self-contained MotionScene만 지원할 것 같다고..?



Interpolated Attributes

일단 MotionScene 내부의 ConstraintSet은 뭔가 더 가능함.

다음 attribute가 알아서 interpolation이 된다고 함.

```
alpha
visibility
elevation
rotation, rotation[X/Y]
translation[X/Y/Z]
scaleX/Y
```



self-contained니까 레이아웃을 따로 정의할 필요는 없음. MotionScene 안에 모든 걸 작성.

```xml
<?xml version="1.0" encoding="utf-8"?>
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:motion="http://schemas.android.com/apk/res-auto">

    <Transition
        motion:constraintSetStart="@+id/start"
        motion:constraintSetEnd="@+id/end"
        motion:duration="1000">
        <OnSwipe
            motion:touchAnchorId="@+id/button"
            motion:touchAnchorSide="right"
            motion:dragDirection="dragRight" />
    </Transition>

    <ConstraintSet android:id="@+id/start">
        <Constraint
            android:id="@+id/button"
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:layout_marginStart="8dp"
            motion:layout_constraintBottom_toBottomOf="parent"
            motion:layout_constraintStart_toStartOf="parent"
            motion:layout_constraintTop_toTopOf="parent" />
    </ConstraintSet>

    <ConstraintSet
        android:id="@+id/end"
        motion:deriveConstraintsFrom="@id/start">

        <Constraint android:id="@id/button">
            <Layout
                android:layout_width="64dp"
                android:layout_height="64dp"
                android:layout_marginEnd="8dp"
                motion:layout_constraintEnd_toEndOf="parent" />
        </Constraint>
    </ConstraintSet>

</MotionScene>
```

관찰할 수 있는, 예제 1과 비교했을 때의 몇 가지 차이점

1. MotionScene 내부에 ConstraintSet 선언 및 id 정의. 이 값이 Transition 속성에서 사용됨.
2. 기본적으로 ConstraintSet은 기존에 설정되어 있는 값을 전부 클리어하고 새로운 Constraint를 적용하는 방식으로 동작함. 두 번째 ConstraintSet은 `motion:deriveConstraintsFrom`을 통해서 첫 번째 ConstraintSet의 Constraint들을 가져왔음. 이에 따라서 두 번째 ConstraintSet의 Constraint는 많은 것들이 생략되어 있음.
   -> 왜 Constraint를 직접 건들지 않고 Layout을 따로 만들어서 설정하는걸까? `deriveConstraintsFrom`의 범위가 어디까지인가… 개별 속성일까 아니면 각각의 태그일까? - 지금 상태로 봐서는 후자가 맞는 것 같기는 함
3. ConstraintSet은 뷰에 대한 정보는 전혀 가지고 있지 않음. 따라서 View 태그를 정의하는 대신 Constraint 태그를 통해서 id를, Layout 태그를 통해서 위치와 크기를 정의.

이로부터 알아낼 수 있는 것들

1. ConstraintSet을 활용하는 모습을 통해서 일단 기존의 ConstraintLayout이 View + ConstraintSet이라고 생각할 수 있을 것 같음.
   MotionLayout에서는 확실하게 View와 ConstraintSet을 나누는 것 같음.
2. 이렇게 나눴기 때문에, 어떤 Widget에 애니메이션을 적용할지 선택하는 것도 간단함. 필요한 것만 MotionScene에 선언하면 됨



MotionLayout 속성들

`app:layoutDescription=”reference”` 이전에 설명. scene 파일을 가리킴

`app:applyMotionScene=”boolean”` MotionScene 적용 여부를 결정함. 껏다 켰다 하면 애니메이션같은 걸 조절할 수 있을 듯.

`**app:showPaths=”boolean”**` 이전에 설명. 경로를 보여줌.

`**app:progress=”float”**` Transition의 Progress 설정

`**app:currentState=”reference”**` 특정한 ConstraintSet을 현재 상태로 설정'



몇 가지 의문

- Progress에 대한 이야기를 하는데.. 아마 MotionLayout은 내부적으로 0~1 사이의 Progress를 통해서 Transition을 관리하는 것 같음.





3. Custom Attributes

원하는 속성을 지정할 수도 있음.

```xml
<Constraint>
    <CustomAttribute
        motion:attributeName="backgroundColor"
        motion:customColorValue="#D81B60"/>
</Constraint>
```

가령 backgroundColor를 쓰면 색이 변함

CustomAttribute의 동작 방식

- getter/setter가 존재해야 함.
  get{Name} / set{Name}
- 값도 정의해야 함
  custom{Something}Value
  customDimension
  ...

한 쪽에 쓰면 다른 한 쪽에도 써야 한다고 합니다..

-> 이걸 이용하면 뷰의 거의 대부분의 상태 정보를 MotionLayout에서 MotionScene으로 분리할 수 있음.

다만 목적 자체가 Transition을 위함이니 만큼, 변경하고 싶은 속성만 그때그때 하는게 바람직한 것 같음.

















