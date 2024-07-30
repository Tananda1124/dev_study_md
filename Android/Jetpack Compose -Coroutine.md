
# Jetpack Compose - Coroutine

LifecycleScope와 LaunchedEffect의 사용 시점과 용도를 정리한 내용이다.

### LifecycleScope
- **사용 시점**: `LifecycleScope`는 Android 구성 요소(예: Activity, Fragment)의 생명주기와 연동할 때 사용한다.
- **용도**: Activity나 Fragment의 생명주기 상태에 따라 코루틴 작업을 안전하게 실행 및 관리하기 위해 사용한다. 예를 들어, `onCreate`, `onStart`, `onResume` 등 생명주기 콜백 내에서 비동기 작업을 수행할 때 적합하다.
- **예제**:
  ```kotlin
  class MyActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)

          lifecycleScope.launch {
              // 네트워크 요청 등 비동기 작업 수행
              val data = fetchData()
              updateUI(data)
          }
      }

      private suspend fun fetchData(): String {
          // 비동기 작업 수행
          delay(1000L)
          return "Hello World"
      }

      private fun updateUI(data: String) {
          // UI 업데이트 작업
          findViewById<TextView>(R.id.textView).text = data
      }
  }
  ```

### LaunchedEffect
- **사용 시점**: `LaunchedEffect`는 Jetpack Compose의 `@Composable` 함수 내부에서 컴포저블의 생명주기와 연동할 때 사용한다.
- **용도**: 특정 상태나 키 값의 변경에 따라 비동기 작업을 수행하고, 컴포저블이 컴포지션에서 벗어나거나 상태가 변경될 때 코루틴을 안전하게 취소하기 위해 사용한다. 주로 컴포저블의 초기화 작업이나 상태 변경에 따른 비동기 작업에 적합하다.
- **예제**:
  ```kotlin
  @Composable
  fun MyComposable() {
      var data by remember { mutableStateOf("Loading...") }

      LaunchedEffect(Unit) {
          // 네트워크 요청 등 비동기 작업 수행
          data = fetchData()
      }

      Text(text = data)
  }

  private suspend fun fetchData(): String {
      // 비동기 작업 수행
      delay(1000L)
      return "Hello Compose"
  }
  ```

### 요약
- **LifecycleScope**: 앱의 생명주기(`onCreate`, `onStart`, `onResume` 등)와 연동하고 싶을 때 사용한다. 주로 Activity나 Fragment에서 비동기 작업을 수행할 때 사용한다.
- **LaunchedEffect**: Compose의 컴포저블과 연동하고 싶을 때 사용한다. 컴포저블이 처음 렌더링될 때, 또는 특정 상태가 변경될 때 비동기 작업을 수행한다.

두 가지를 적절히 사용하면 앱의 생명주기와 UI 상태 변화에 따라 비동기 작업을 효과적으로 관리할 수 있다.
