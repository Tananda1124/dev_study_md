### Jetpack Compose를 왜 사용해야 하는가?
Jetpack Compose는 Android에서 제공하는 최신 UI 구성 및 조작 툴킷이다.
xml 형식으로 잘 정의해 왔던 View를 왜 Kotlin 선언형인 Compose로 바꿔야 하는지 의문이 들어, 다음에 이유를 정리해 보았다.

1. 기존 Android 프로그래밍에서는 View를 수동으로 조작해야 한다.(FindViewById, binding 등의 메소드를 이용해 경우에 따라 수동 조작해 왔다.) 수동으로 조작하게 될 경우, 여러 뷰를 수동으로 업데이트해야 하기 때문에 기능 설정을 잊어버리거나 오류가 발생할 위험이 있다.
2. 업계 트렌드가 선언형 UI로 전환되었고, 다른 여러 플랫폼에서 이미 UI 구성 방식으로 선언형을 사용하고 있다. 그에 따른 UI구성,업데이트,유지-보수 등 측면의 엔지니어링의 간소화는 덤이다.
3. Compose는 UI를 전부 다시 그리기보다는 다시 그릴 부분만 선택하여 로딩한다. 이에 따라 리소스 관리에 용이하다.

compose는 영어 사전에서 '요소' 라는 뜻이다. 이러한 정의와 같이, 화면의 요소를 기준으로 View를 구성하는 방법이라고 할 수 있다.
### Compose를 사용하는 법
Compose는 Kotlin 코드 내에서 UI를 선언하는 View 구성 방법이다. 사용 방법은 다음과 같다.
```Kotlin
@Composable
fun Greeting(name : String){
	Text("Hello, $name")
}
```
- @Composable 주석으로 함수를 지정하여 사용해야 한다.
- 함수는 데이터(매개변수)를 받을 수 있다. 이러한 매개변수를 이용해 앱 로직을 구성한다.
- 함수는 MainActivity 등에서 사용될 뿐, 다른 반환형을 가지지 않는다.️
###### 재구성 프로세스
선언형 위젯은 객체를 노출하지 않고, 상태를 제공한다. 이에 따라 ViewModel과 연동하여 앱의 상태를 UI로 변환할 수 있다.![[mmodel-flow-data.png]]![[mmodel-flow-events.png]]
위 그림과 같이, 사용자가 화면에서 상호작용을 하게 되면, 해당 상호작용의 데이터가 앱 로직에 전달된다. 앱 로직은 적절한 이벤트를 반환하여 UI와 서비스 로직을 제공하게 된다.

###### 동적 콘텐츠
이처럼 이벤트로 작동하는 함수는 Kotlin으로 작성되기 때문에 동적으로 선언할 수 있다. 다음과 같이 반복해서 UI를 생성하는 작업도 가능하다.
```Kotlin
@Composable  
fun Greeting(names: List<String>) {
	for (name in names) {
		Text("Hello $name")
	}  
}
```
위 Composable을 이용하면, Text 요소 여러 개를 오류 없이 간단히 반복해 생성할 수 있다.

###### UI 재구성 with MVVM
UI를 선언하며 onClick 등의 이벤트 리스너를 설정하는 것도 가능하다.
```Kotlin
@Composable  
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
	Button(onClick = onClick) {
		Text("I've been clicked $clicks times")
	}  
}
```
위 코드는 원시적 형태이고, MVVM 아키텍쳐에서는 아래와 같이 ViewModel을 설정하고 UI의 상태를 관리하는 등의 작업이 필요하다.

```Kotlin
class ClickCounterViewModel : ViewModel() {
    private val _clicks = MutableLiveData(0)
    val clicks: LiveData<Int> = _clicks

    fun onClick() {
        _clicks.value = (_clicks.value ?: 0) + 1
    }
}

@Composable
fun ClickCounterApp(viewModel: ClickCounterViewModel = viewModel()) {
    val clicks by viewModel.clicks.observeAsState(0)

    ClickCounter(clicks = clicks, onClick = viewModel::onClick)
}

@Composable️
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}

@Preview(showBackground = true)
@Composable
fun ClickCounterAppPreview() {
    ClickCounterApp()
}

```
ViewModel에 UI의 상태를 감시하는 LiveData 형식의 변수를 선언하고, ClickCounter와 같이  UI 상태에 따라 로직을 설정한다. 이 두 로직을 연결하는 부분도 필요한데, 예시 코드에서는 ClickCounterApp이 그 역할을 한다.
위 과정과 같이 ViewModel과 View가 각자 책임을 분리하고, 상태에 따른 UI 업데이트를 효율적으로 구성할 수 있다.️