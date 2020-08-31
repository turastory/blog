시리즈: 안드 개발자가 디자이너와 일하는 방법

# 타이포그래피 싱크하기

### 배경 - velog

안드로이드 앱을 개발하면 필연적으로 디자이너와 협력을 하게 됩니다. 앱에서 사용하는 색상, 아이콘, 텍스트부터 여러 UI 컴포넌트까지 협력이 필요한 요소는 무궁무진합니다.

그 중에서도 오늘은 텍스트, 즉 **타이포그래피(Typography)**에 관한 이야기를 해볼까 합니다. Sketch, Figma, Zepplin 등을 통해서 디자인을 전달 받고, 이를 실제로 구현해보면 다른 것들은 별 다른 문제가 없는데, 항상 어긋나는 게 하나 있습니다. 바로 **텍스트!!!** (~~아니 저는 그냥 문서에 나와있는대로 했다구요ㅠㅠ~~)

타이포그래피의 디자인과 실제 구현 사이의 동기화를 어떻게 할 수 있을까요? 우선 이 문제를 좀 더 파헤쳐봅시다..

### 문제

문제는 안드로이드가 텍스트를 렌더링할 때 사용하는 방식이, 디자이너분들에게 익숙한 방식과는 다르기 때문에 발생합니다.

[머테리얼 디자인의 Typography 섹션](https://material.io/design/typography/understanding-typography.html)을 보면, **Baseline**과 **Cap-Height**라는 개념을 알 수 있습니다. 이것을 토대로 순수한 텍스트의 높이를 Baseline to Cap-Height라고 하고, 패딩이 포함된 높이를 Line Height라고 부를 수 있죠.

![baseline-to-cap-height-picture](https://i.imgur.com/mlNVY7j.png)

디자이너분들은 이러한 개념을 가지고 텍스트를 만들어 주시는데, 안드로이드에서는 텍스트의 Line Height를 직접적으로 지정할 수 있는 방법이 없습니다. 때문에 줄 사이의 간격을 조절하는 `android:lineSpacingExtra`와 상하의 Padding을 사용해서 이를 표현해야 하죠.

![android-typo-grid-picture](https://i.imgur.com/9ug1GwC.png)

그러나 여기서 끝이 아닙니다. 그림을 보면 `android:height` 속성을 "wrap_content"로 설정했을 때 **기본적인 텍스트의 높이가 "Baseline to Cap-Height"와는 다른 것**을 확인할 수 있죠? 그렇기 때문에 단순히 Line Height에서 이 텍스트의 높이를 뺀 값을 가지고 Padding과 Line Spacing Extra를 설정하기에는 무리가 있습니다.

Figma의 타이포그래피 디자인을 안드로이드에서 동일하게 구현할 수 있는 방법은 정말 없는걸까요?

### 실험

우선은 TextView에서 사용하는 여러 수치들과 높이의 상관 관계를 알아보기로 했습니다. 이 관계를 밝혀내면 Figma에 정의되어 있는 값과 각각의 값들 사이의 관계식을 세울 수 있고, 이 관계식을 통해 Figma의 디자인을 안드로이드로 동일하게 그려낼 수 있습니다. 후보는 **Line Spacing Extra**, **Line Spacing Multiplier**, 그리고 **Text Size**입니다.

실험은 아래와 같은 샘플 프로젝트를 준비하고, Flipper에서 레이아웃 플러그인을 사용해서 수치를 얻었습니다. 아무래도 이 방법이 제일 편하더라구요.

**그리고 Font Padding으로 인한 수치의 변화를 막기 위해서 `android:includeFontPadding` 속성을 false로 설정했습니다.** 이렇게 하지 않으면 텍스트가 한 줄일 때의 높이와 여러 줄일 때의 높이 증가량이 일치하지 않고, Line Spacing Multiplier에 따른 높이의 변화 역시 비선형적으로 나타나는 등의 문제가 발생합니다. 그러니 꼭 `android:includeFontPadding` 플래그를 꺼주세요.

![experiments-using-flipper](https://i.imgur.com/5lkCGzP.png)

아래는 각각의 실험에서 공통적으로 사용하는 몇 가지 변수들입니다.

- __*d*__ - Device Density. 안드로이드는 다양한 기기가 존재하기 때문에, 픽셀 값을 기준으로 화면을 그리게 되면 기기에 따라서 뷰의 사이즈가 다르게 그려집니다. 때문에 이러한 문제를 해결하고 기기마다 다른 수치를 통일해서 사용하고자 도입한 개념입니다. 우리가 dp 혹은 sp로 정의한 값에 Density를 곱한 값이 실제로 그려지는 픽셀 값이 되죠.
- __*t*__ - Text Size. 안드로이드에서 dp 혹은 sp로 지정하는 텍스트 사이즈입니다.

#### Line Spacing Extra

`android:lineSpacingExtra` 속성은 TextView에서 줄 사이의 간격을 추가할 때 사용합니다. 

![line-spacing-extra-table](https://i.imgur.com/V3zbGDx.jpg)

테이블의 열은 **텍스트의 라인 수**, 행은 **Line Spacing Extra 값**을 의미합니다. 각 행과 열이 교차되는 지점의 값은 Flipper의 Layout Inspector 플러그인으로 확인한 **뷰의 실제 높이**입니다.

위 테이블을 통해서 확인할 수 있는 것들은 다음과 같습니다.

- 라인 수가 늘어남에 따라 높이가 일정한 비율로 증가합니다. 즉 **라인 수와 높이는 선형적으로 비례**합니다.
- Line Spacing Extra 값이 늘어남에 따라 높이가 일정한 비율로 증가합니다. 즉 **Line Spacing Extra와 높이 역시 선형적으로 비례**합니다.
- 라인 수가 1개일 때는 Line Spacing Extra의 영향이 없습니다.



#### Line Spacing Multiplier

`android:lineSpacingMultiplier` 속성은 `android:lineSpacingExtra`와 비슷하지만, 고정 값이 아니라 비율로 설정한다는 점이 다릅니다.

![line-spacing-multiplier-table](https://i.imgur.com/edV6Ayj.jpg)

테이블의 행이 **Line Spacing Multipler 값**에 해당한다는 것을 제외하고는 이전과 동일합니다.

위 그래프를 통해서는 다음과 같은 것들을 확인할 수 있습니다. (이전에 확인했던 것들은 제외했습니다.)

- Line Spacing Multiplier 값이 늘어남에 따라 높이가 일정한 비율로 증가합니다. 즉 **Line Spacing Multiplier와 높이는 선형적으로 비례**합니다.
- 라인 수가 1개일 때는 Line Spacing Multiplier의 영향이 없습니다.



#### Text Size

마지막으로 실험해볼 것은 `android:textSize` 속성입니다. 말 그대로 TextView의 텍스트 사이즈를 지정하는 속성이죠.

![text-size-table](https://i.imgur.com/en0c0ma.jpg)

이번에는 그동안 고정했던 Text Size에 따른 높이의 변화를 확인해보았습니다. 역시 다른 것들은 이전과 동일하고, 행 부분만 Text Size로 설정했습니다. (단위는 sp)

위 테이블에서는 다음과 같은 사실들을 확인할 수 있었습니다.

- Text Size가 변해도 라인 수와 Height 사이의 비례 관계는 변하지 않습니다.
- Text Size 값이 늘어남에 따라 높이가 일정한 비율로 증가합니다. 즉 **Text Size와 높이는 선형적으로 비례**합니다.



#### 종합

결과를 대략적으로 종합해보면 다음과 같습니다.

- Line Spacing Extra, Line Spacing Multiplier, Text Size 모두 뷰의 높이와 선형적으로 비례합니다.
  -> Figma에 정의되어 있는 **Line Height와, 상기한 여러 속성들 간의 관계식**을 만들 수 있습니다!

- Line Spacing Extra, Line Spacing Multiplier 만으로는 Text Size의 변화 없이 라인이 한 줄인 TextView의 높이를 변화시킬 수 없습니다.
  -> 이러한 속성들을 제외한 **다른 속성**을 활용해야 합니다. (앞서 이야기한 것처럼 Padding을 사용)



### 구현

#### 식 세우기

실험을 통해서 Text Size 등의 수치와 관계 없이 어떤 값을 사용하더라도 일정한 비율을 얻을 수 있다는 것을 확인했습니다. (높이와의 관계가 모두 선형적으로 비례하므로)

앞서 보았던 데이터를 가져오면, t = 40, d = 3일 때의 뷰의 높이는 177이었습니다. 이를 수식으로 나타내면 다음과 같습니다.

![40 \times 3 \times F_{font} = 177](https://i.imgur.com/ZeRz9H5.jpg)

이 방정식을 풀면 다음과 같은 결과를 얻을 수 있습니다.

![F_{font}  = 177 / 120 \approx  1.48](https://i.imgur.com/QRhH5e2.jpg)

몇 가지 폰트를 가지고 테스트해본 결과, 이 값은 폰트마다 다른 일종의 상수로 보입니다. (여기서는 [Spoqa Han Sans](https://spoqa.github.io/spoqa-han-sans/ko-KR/) 폰트를 사용했습니다.)



앞의 식을 좀 더 일반적인 형태로 작성해보면 다음과 같습니다. (h: height, t: text size, d: device density)

![h = F_{font}td](https://i.imgur.com/bKKzPiE.jpg)

Figma에 정의되어 있는 Line Height를 H_f라고 한다면, Figma와 Android의 Line Height 오차 e는 다음과 같이 정의할 수 있습니다.

![e = H_{f}d - F_{font}td](https://i.imgur.com/JhV5eAj.jpg)

또한 단위를 DP로 바꾸면 density는 제거할 수 있습니다.

![e = H_{f} - F_{font}t](https://i.imgur.com/i82EFzb.jpg)



#### 적용

이제 이렇게 찾은 식을 실제로 써보기 위해서 이전에 사용했던 그림을 다시 가져와볼게요.

![baseline-to-cap-height-picture](https://i.imgur.com/mlNVY7j.png)

![android-typo-grid-picture](https://i.imgur.com/9ug1GwC.png)

Figma의 Line Height는 **각 라인마다 위 아래 패딩을 기준**으로 정의되어 있고, 안드로이드에서는 이러한 속성이 없어 **Padding과 Line Spacing Extra를 활용**해야 합니다. (Line Spacing Multiplier는 활용하기 어려워 사용하지 않았습니다)

따라서 Padding과 Line Spacing Extra는 다음과 같이 정의할 수 있습니다.

![p_{bottom} = p_{top} = e / 2](https://i.imgur.com/RQ1bJIq.jpg)

![s=e](https://i.imgur.com/1mcCPtr.jpg)



위 내용을 실제 코드로 적용해보았습니다. 꽤나 괜찮은 결과가 나오는 것 같습니다 :)

```kotlin
fun TextView.syncLineHeight(figmaLineHeight: Float, factor: Float = 1.48f) {
    val lineSpacingExtra =
        figmaLineHeight.toDp(requireContext()) - factor * this.textSize

    val padding = lineSpacingExtra
        .takeIf { it != 0.0f }
        ?.div(2)
        ?.roundToInt()
        ?: 0

    setLineSpacing(lineSpacingExtra, 1f)
    updatePadding(
        top = padding,
        bottom = padding
    )
}
```

![line-height-demo](https://i.imgur.com/qabBVbm.gif)



### 정리

이렇게 큰 산 하나는 넘었지만 아직 해결해야 할 과제들은 많이 남아있습니다.

우선 Line Height를 설정하는 함수를 데이터 바인딩을 통해서 XML에서도 사용할 수 있게 만들어야 하고, 다양한 서비스 - 즉 모듈들에서 디자인 시스템을 사용할 것을 고려하면 이 바인딩 어댑터가 각 서비스 내에 존재하지 않고 공통된 모듈로 분리되어야 합니다.

그 다음 문제는 Padding과 관련된 문제입니다. 기존에 설정했던 Padding을 유지하면서 Line Height에 해당하는 Padding을 추가, 혹은 빼야할 필요도 있을테니까요.

그렇지만 이런 문제들은 시간을 조금 들이면 충분히 해결 가능한 성질의 문제들이라고 생각합니다. 어찌 되었든 디자이너분들이 Figma로 전달해주시는 것과 동일한 형태로 텍스트를 그릴 수 있게 되었으니, 이제 억울함을 풀 수 있겠네요! (~~아무튼 안드로이드가 문제야ㅡㅡ~~)

긴 글 읽어주셔서 감사합니다 :)

