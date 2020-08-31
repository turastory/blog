## 좋은 점?

1. XML 기능 확장
   `Set method`나 `@BindingMethod`를 써서 숨겨져있는 여러 함수나 프로퍼티를 간단하게 XML에서 사용할 수 있다

2. Model 부분에서 View에 대한 의존성을 제거
   View -> Model로 변화를 받아야 할 때 유용하다 (2 way binding)

## 3 Ways to configure data binding

[공식 문서](https://developer.android.com/topic/libraries/data-binding/binding-adapters)

### Priority

- Binding Adapter
- Binding Method
- Set Method

### Set method (Automatic method selection)

In XML...

```xml
<!-- main_activity.xml -->
<TextView
  android:id="@+id/sample_text"
  android:enabled="@{true}" />

<CustomView
  android:id="@+id/sample_custom_view"
  app:goneIf="@{false}"/>
```

```kotlin
// CustomView.kt
var goneIf: Boolean
```

```java
// MainActivityBinding.java
this.sampleText.setEnabled(true);
this.sampleCustomView.setGoneIf(false);
```

#### 결론

xml에서 명시한 이름과 signature에 해당하는 함수를 찾아서 연결시켜줌
아주 간단한 수준의 바인딩


### Binding Adapter (Provide custom logic)

```java
@BindingAdapter(
	values = ["hello", "isUppercase"],
    requireAll = false
)
public static void bindTextView(TextView textView, int val1, boolean val2) {
    // 구현
}
```

같은 코드를 코틀린으로 하면...
(리시버를 활용하면 이 함수를 다른 곳에서도 유용하게 쓸 수 있음)

```kotlin
@BindingAdapter(
	values = ["bind:hello", "isUppercase"],
    requireAll = false
)
fun TextView.bindTextView(val1: Int, val2: Boolean) {
    // 구현
}
```

여기서

1. `@BindingAdapter`는 바인딩 어댑터를 선언하는 데 사용.
   여기서 values가 이 바인딩 어댑터가 처리하는 attributes에 해당. (e.g. `bind:hello, app:isUppercase`)
   `bind:hello`와 같이 namespace를 명시적으로 지정할 수도 있고, 지정하지 않으면 `app`으로 설정됨.
   
2. 함수의 첫번째 파라미터는 바인딩할 대상 View를 나타냄
   언급한 것처럼 코틀린에서 Receiver로 활용할 수 있음.

3. 함수의 나머지 파라미터는 어노테이션에서 지정한 각 values에 대응됨.
   이 때 둘의 개수가 정확히 일치하지 않으면 에러가 발생함.

이름이나 함수 몸체 등은 중요하지 않음

#### 결론

모든 부분을 커버할 수 있는 유니버셜한 바인딩
조금 복잡함


### Binding Method (Specify a custom method name)

```kotlin
@BindingMethods(
  value = BindingMethod(
    type = View::class,
    attribute = "android:onClick",
    method = "setOnClickListener"
  )
)
```

Set method와 비슷하지만 attribute이름을 가지고 method를 찾는게 아니라
함수 이름을 따로 명시한다는게 다름

#### 결론

거의 쓸 일 없다


## Two-way binding

### InverseBinding

Model -> View가 Binding이라면
View -> Model이 InverseBinding임

```kotlin
@InverseBindingAdapter(attribute = "android:text", event = "textChanged")
fun TextView.getTextInverseBinding(): String {
    return text.toString()
}
```

여기서 `event`는 **InverseBinding이 동작해야 하는 시점**을 결정짓는다.
그런데 이 event를 발생시키기 위한 BindingAdapter가 하나 더 필요하다.

```kotlin
@BindingAdapter(value = ["textChanged"])
fun TextView.setTextChanged(inverseBindingListener: InverseBindingListener) {
    addTextChangedListener(object : SimpleTextWatcher() {
        override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
            inverseBindingListener.onChange()
        }
    })
}
```

`InverseBindingListener`는 안드로이드의 바인딩 시스템에게 값의 변화를 알리고, 그 결과 textChanged라는 이벤트가 발생하게 되어서 InverseBinding이 동작하는 것.


### Prevent infinite loop

그런데 이렇게 바인딩을 만들면,

1. InverseBinding을 통해서 값이 뷰에서 모델에 전달됨
2. Binding을 통해서 값이 모델에서 뷰로 전달됨.
3. 이 변화가 `textChanged` 이벤트를 발생시킴
4. 반복...

이렇게 무한 반복이 일어나는데.. 이걸 막기 위해서는 다음과 같이 값을 비교하는 로직을 `BindingAdapter` 쪽에 작성해주어야 함

```kotlin
@BindingAdapter(value = ["android:text"])
fun TextView.setTextPreventOverwrite(text: String) {
    if (isSame(this.text.toString(), text)) return;
    setText(text)
}
```

아주 조심해야 한다..





### Some mistakes

#### apply()

apply()를 사용할 때, 다음과 같은 실수를 했음.

```kotlin
private val vm by lazy {}

fun onCreate() {
    binding = bind(R.layout.activity_main).apply {
        this.vm = vm
    }
}
```

원래 의도는 Activity의 vm을 binding의 vm과 연결하는 것이었는데, 여기서 this.vm과 vm은 같은 것으로 취급됨.

제대로 동작하려면 다음과 같이 scope를 명시해주거나

```kotlin
binding.apply {
    this.vm = this@MainActivity.vm
}
```

아예 따로 작성하면 됨

```
binding = bind()
binding.vm = vm
binding.lifecycleOwner = this
```

아니면 혼동을 피하기 위해서 이름을 다르게 하는게 제일 좋을 것 같긴 함

```kotlin
binding.viewModel = vm
```









