
# 브로드캐스트 리시버와 ViewModel을 이용한 Compose 뷰 업데이트 예제

이 예제에서는 Android Jetpack Compose에서 ViewModel과 BroadcastReceiver를 이용하여 뷰를 업데이트하는 방법을 다룬다. 배터리 상태를 브로드캐스트로 받아서 Compose UI에 표시하는 간단한 예제이다.

## 브로드캐스트 리시버 사용법

1. **브로드캐스트 리시버에서 ViewModel을 사용할 수 있도록 인자로 ViewModel을 주어 할당하고 변수를 업데이트하도록 엮는다.**
2. **브로드캐스트 리시버를 MainActivity에서 선언해 활성화한다. 또한, onCreate 등에서 ViewModel을 인자로 주어 ViewModel의 변수를 변경할 수 있도록 선언한다.**
3. **@Composable View에 ViewModel 변수를 설정해 감시하도록 한다.**

## 1. 필요한 종속성 추가

먼저, `build.gradle` 파일에 Compose와 관련된 종속성을 추가한다:

```groovy
dependencies {
    implementation "androidx.compose.ui:ui:1.2.0"
    implementation "androidx.compose.material:material:1.2.0"
    implementation "androidx.compose.ui:ui-tooling-preview:1.2.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.4.1"
    implementation "androidx.activity:activity-compose:1.5.0"
}
```

## 2. ViewModel 설정

배터리 상태를 저장할 ViewModel을 설정한다.

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class BatteryViewModel : ViewModel() {

    private val _batteryLevel = MutableLiveData<Int>()
    val batteryLevel: LiveData<Int> = _batteryLevel

    fun setBatteryLevel(level: Int) {
        _batteryLevel.value = level
    }
}
```

## 3. BroadcastReceiver 설정

배터리 상태 변경 브로드캐스트를 수신하는 BroadcastReceiver를 설정한다.

```kotlin
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewmodel.compose.viewModel

class BatteryReceiver(private val viewModel: BatteryViewModel) : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val level = intent.getIntExtra("level", 0)
        viewModel.setBatteryLevel(level)
    }
}

fun registerBatteryReceiver(context: Context, viewModel: BatteryViewModel): BroadcastReceiver {
    val receiver = BatteryReceiver(viewModel)
    val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
    context.registerReceiver(receiver, filter)
    return receiver
}
```

## 4. Compose UI 설정

Compose UI에서 ViewModel을 관찰하여 UI를 업데이트한다.

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel

class MainActivity : ComponentActivity() {
    private lateinit var batteryReceiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel: BatteryViewModel = viewModel()
            batteryReceiver = registerBatteryReceiver(this, viewModel)

            BatteryStatusScreen(viewModel)
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(batteryReceiver)
    }
}

@Composable
fun BatteryStatusScreen(viewModel: BatteryViewModel) {
    val batteryLevel by viewModel.batteryLevel.observeAsState(0)

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        Text(text = "Battery Level: $batteryLevel%", style = MaterialTheme.typography.h4)
    }
}
```

이렇게 하면 배터리 상태가 변경될 때마다 Compose UI가 자동으로 업데이트된다.
