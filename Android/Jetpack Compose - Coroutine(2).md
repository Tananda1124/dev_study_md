
# Jetpack Compose - Coroutine(2)

LifecycleScope와 LaunchedEffect을 제외한 코루틴의 사용 시점과 용도를 정리한 내용이다.
Jetpack Compose에서는 `LaunchedEffect` 외에도 다양한 방식으로 코루틴을 사용할 수 있다. 이 글에서는  Compose와 관련된 코루틴 사용 방법 몇 가지를 소개한다.
### 1. rememberCoroutineScope

`rememberCoroutineScope`는 컴포저블 함수 내에서 코루틴 스코프를 제공하며, 이를 통해 코루틴을 시작할 수 있다. 이는 컴포저블의 생명주기에 따라 자동으로 취소되지 않기 때문에, 수동으로 취소하거나 관리할 필요가 있다.

#### 사용 예

```kotlin
@Composable
fun MyComposable() {
    val coroutineScope = rememberCoroutineScope()

    Button(onClick = {
        coroutineScope.launch {
            // 코루틴 내부에서 비동기 작업 수행
            val data = fetchData()
            println(data)
        }
    }) {
        Text("Fetch Data")
    }
}

private suspend fun fetchData(): String {
    delay(1000L)
    return "Hello, World!"
}
```

### 2. SideEffect

`SideEffect`는 컴포저블 함수 내에서 상태가 변경될 때 실행되어야 하는 작업을 수행할 때 사용한다. 이는 주로 Compose 외부의 상태와 동기화할 때 유용하다.

#### 사용 예

```kotlin
@Composable
fun MyComposable(value: Int) {
    SideEffect {
        // Compose 외부의 상태를 동기화하는 작업 수행
        println("Value changed to $value")
    }

    Text("Value: $value")
}
```

### 3. DisposableEffect

`DisposableEffect`는 컴포저블 함수 내에서 초기화 작업과 정리 작업을 모두 수행할 수 있는 API이다. 예를 들어, 리소스를 할당하고 해제하는 작업을 관리할 때 유용하다.

#### 사용 예

```kotlin
@Composable
fun MyDisposableComposable() {
    DisposableEffect(Unit) {
        // 초기화 작업
        println("DisposableEffect started")

        onDispose {
            // 정리 작업
            println("DisposableEffect disposed")
        }
    }

    Text("DisposableEffect Example")
}
```

### 4. produceState

`produceState`는 Compose 내에서 상태를 생성하고 관리하는 데 사용된다. 이는 상태를 생성하기 위해 코루틴을 사용하며, 해당 상태가 변화할 때 UI를 업데이트한다.

#### 사용 예

```kotlin
@Composable
fun MyProduceStateComposable() {
    val data by produceState(initialValue = "Loading...") {
        value = fetchData()
    }

    Text("Data: $data")
}

private suspend fun fetchData(): String {
    delay(1000L)
    return "Hello, ProduceState!"
}
```

### 5. derivedStateOf

`derivedStateOf`는 다른 상태에 의존하는 상태를 생성하는 데 사용된다. 이는 상태가 변경될 때 파생 상태를 자동으로 업데이트한다.

#### 사용 예

```kotlin
@Composable
fun MyDerivedStateComposable() {
    var count by remember { mutableStateOf(0) }
    val isEven by derivedStateOf { count % 2 == 0 }

    Column {
        Text("Count: $count")
        Text("Is even: $isEven")

        Button(onClick = { count++ }) {
            Text("Increase Count")
        }
    }
}
```

### 요약

Jetpack Compose에서는 `LaunchedEffect` 외에도 다양한 코루틴 및 상태 관리 API가 제공된다. `rememberCoroutineScope`, `SideEffect`, `DisposableEffect`, `produceState`, `derivedStateOf` 등 다양한 API를 사용하여 Compose와의 상태 관리 및 비동기 작업을 효과적으로 수행할 수 있다. 이를 통해 더욱 유연하고 강력한 UI 컴포저블을 작성할 수 있다.
