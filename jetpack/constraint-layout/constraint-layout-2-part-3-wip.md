# Android Jetpack - ConstraintLayout 2.0 Part 3. WIP

> 이 시리즈에서는 MotionLayout을 제외한 ConstraintLayout 2.0의 기능을 소개합니다. ConstraintLayout 1.1에 대한 내용은 [이전 포스팅](https://velog.io/@tura/android-jetpack-constraint-layout)을 참고해주세요.

이번 포스팅에서는 다른 것들을 알아보겠습니다.

## ConstraintLayoutStates

여러 상태를 가진 레이아웃을 만들 수 있음

이 상태 간의 전환을 쉽게 할 수 있음

대부분의 레이아웃이 Loading - Init - End - Error 등의 상태를 가지고 있음

ConstraintLayoutState를 이용하면 이런 상태 간의 전환을 쉽게 할 수 있음.



1. 각 상태별로 Layout 파일 1개
2. ConstraintLayoutStates를 위한 XML 파일 하나
   `R.xml` 밑으로.



```
<?xml version="1.0" encoding="utf-8"?>
<ConstraintLayoutStates xmlns:android="http://schemas.android.com/apk/res/android"
                        xmlns:app="http://schemas.android.com/apk/res-auto">
    <State
            android:id="@+id/start"
            app:constraints="@layout/activity_cl_states_start"/>
    <State
            android:id="@+id/loading"
            app:constraints="@layout/activity_cl_states_loading"/>
    <State
            android:id="@+id/end"
            app:constraints="@layout/activity_cl_states_end"/>
</ConstraintLayoutStates>
```





 ### constraintProperties

기존에는 아래와 같이 `ConstraintLayout`으로부터 `ConstraintSet`을 받아 와서 그 안에 위치한 여러 가지 뷰들에 대한 `Constraint`를 설정했습니다.

![constraint-set-example](/Users/nayoonho/Documents/jetpack/constraint-layout/constraint-set-example.png)



2.0에서 새롭게 추가되는 `ConstraintProperties`를 이용하면 개별 뷰에 대한 `ConstraintProperties`를 이용하면 좀 더 

![constraint-properties-example](/Users/nayoonho/Documents/jetpack/constraint-layout/constraint-properties-example.png)