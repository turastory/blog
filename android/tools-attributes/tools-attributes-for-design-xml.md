# Tools Attributes for better editing

XML을 작성할 때, 종종 Preview가 제대로 나타나지 않거나, 혹은 원하는 대로 나타나지 않아서 불편할 때가 있습니다.

깊게 들어가지 않아도 당장 `RecyclerView`를 작성할 때 실제 개별 아이템에 대한 레이아웃은 다른 파일로 분리하게 되는데, 이렇게 되면 실제로 `RecyclerView`가 어떻게 보여지는지 알 수가 없습니다.

이런 경우 *tools attributes*를 유용하게 사용할 수 있습니다.



## Tools Attributes

`tools attributes`는 안드로이드 스튜디오가 `tools` 네임스페이스로 지원하는 여러 XML 속성들입니다.

이 속성들은 IDE에서 작업을 하는 동안 Inspection이나 Layout Preview 등에 활용되고, 실제 apk 빌드시에는 모두 제거됩니다.



### Namespace

`tools attributes` 네임스페이스를 정의하려면 최상위 XML 태그에 다음과 같이 선언하면 됩니다.

```xml
<RootTag xmlns:tools="http://schemas.android.com/tools">
```

안드로이드 스튜디오에서는 이를 간편하게 추가할 수 있는 Live Template을 지원하고 있습니다. XML 태그 내에서 `toolsNs`를 입력하면 됩니다.

![toolsns-live-template](/Users/nayoonho/Documents/android/tools-attributes/toolsns-live-template.png)



여기서는 몇 가지 유용한 `tools attributes`를 소개합니다.



### Design

#### RecyclerView

RecyclerView를 사용할 때 어떤 아이템이 보여질지 Preview에서 확인하고 싶다면 `listitem`을 사용해서 보여줄 레이아웃을 지정할 수 있습니다.

`itemCount`는 

```xml
tools:listitem="@layout/item_sample"
tools:itemCount="5"
```







## Further Resources

보다 자세한 tools attributes에 대한 참조는 다음 문서를 확인하세요.

https://developer.android.com/studio/write/tool-attributes#toolsshrinkmode