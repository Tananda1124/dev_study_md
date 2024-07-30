### 개요
Firebase와 Android App 간의 연결 및 푸시 메세지를 받아 Toast로 표시하는 과정을 작성하였다.
### 기본 설정
Firebase 기능을 사용하기 위해, Firebase 홈페이지에서 프로젝트를 만들고 json 파일을 프로젝트에 포함시켜야 한다.
json 파일을 프로젝트에 성공적으로 가져왔다면, Manifest와 Gradle을 다음과 같이 수정한다.

```AndroidManifest.xml
<!-- FCM -->  
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```
```build.gradle  
implementation(platform("com.google.firebase:firebase-bom:33.1.0"))
```
```settings.gradle  
id("com.google.gms.google-services") version "4.4.2" apply false
```

### Firebase 콘솔 연결
기본 권한 설정이 끝나면, Firebase에 연결하고 App의 토큰 값을 가져와야 한다. 이 토큰 값을 이용해야 관리자는 사용자에게 푸시 메세지를 전송할 수 있다.

```AndroidManifest.xml
<service  
    android:name=".FCM.MyFirebaseMessagingService"  
    android:exported="false">  
    <intent-filter>        <action android:name="com.google.firebase.MESSAGING_EVENT" />  
    </intent-filter></service>
```

위 Manifest의 android:name 부분은 FCM과 연동할 서비스 클래스를 미리 정의해야 한다.

```MyFirebaseMessagingService.kt
package com.example.temp1.FCM  
  
import android.content.Intent  
import android.util.Log  
import com.example.temp1.MainActivity  
import com.google.firebase.messaging.FirebaseMessagingService  
import com.google.firebase.messaging.RemoteMessage  
  
class MyFirebaseMessagingService : FirebaseMessagingService() {  
  
    override fun onMessageReceived(remoteMessage: RemoteMessage) {  
        super.onMessageReceived(remoteMessage)  
  
        remoteMessage.notification?.let {  
            sendNotification(it.title, it.body)  
        }  
    }  
  
    private fun sendNotification(title: String?, message: String?) {  
        val intent = Intent(this, MainActivity::class.java).apply {  
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK)  
            putExtra("title", title)  
            putExtra("message", message)  
        }  
        startActivity(intent)  
    }  

	// Logcat에 토큰 값 출력, 이 값을 이용해 FCM 메세지 대상 기기로 등록
    override fun onNewToken(token: String) {  
        super.onNewToken(token)  
        Log.d("FCM", "Refreshed token: $token")  
    }  
}
```

FCM의 기본 채널을 설정한다.
또한, String과 Metadata를 연결한다.

```AndroidManifest.xml
<meta-data  
    android:name="com.google.firebase.messaging.default_notification_channel_id"  
    android:value="@string/default_notification_channel_id" />
```
```string.xml
<resources>  
    <string name="app_name">temp1</string>  
    <string name="default_notification_channel_id">default_channel</string>  
    <string name="default_notification_channel_name">Default Channel</string>  
</resources>
```

위와 같이 기본 설정이 완료되면, Intent를 이용하여 FCM 메세지를 처리하는 코드를 작성한다.

```MainActivity.kt
package com.example.temp1

import android.Manifest
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.provider.Settings
import android.widget.Toast
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.google.firebase.messaging.FirebaseMessaging

class MainActivity : AppCompatActivity() {

    private val multiplePermissionsLauncher =
        registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { permissions ->
            val deniedPermissions = permissions.filter { !it.value }
            if (deniedPermissions.isEmpty()) {
                // 모든 권한이 허용된 경우
                Toast.makeText(this, "All permissions granted.", Toast.LENGTH_SHORT).show()
            } else {
                // 권한이 거부된 경우 안내 다이얼로그 팝업 표시
                showPermissionDeniedDialog(deniedPermissions.keys.joinToString(", "))
            }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        // 안드로이드 버전 체크 후 권한 요청 시작
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            requestPermissions()
        }

        // Firebase 메시징 설정
        FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
            if (!task.isSuccessful) {
                println("Fetching FCM registration token failed: ${task.exception}")
                return@addOnCompleteListener
            }
            // FCM 등록 토큰 가져오기
            val token = task.result
            println("FCM registration token: $token")
        }

        // FCM 메시지 처리
        intent?.extras?.let {
            val title = it.getString("title")
            val message = it.getString("message")
            if (title != null && message != null) {
                Toast.makeText(this, "$title: $message", Toast.LENGTH_LONG).show()
            }
        }
    }


    private fun requestPermissions() {
        val permissionsToRequest = mutableListOf<String>()

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.POST_NOTIFICATIONS)
        }

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_SCAN) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.BLUETOOTH_SCAN)
        }

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.BLUETOOTH_CONNECT)
        }

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.ACCESS_FINE_LOCATION)
        }

        if (permissionsToRequest.isNotEmpty()) {
            multiplePermissionsLauncher.launch(permissionsToRequest.toTypedArray())
        }
    }

    private fun showPermissionDeniedDialog(deniedPermissions: String) {
        AlertDialog.Builder(this)
            .setTitle("어플리케이션 권한 필요")
            .setMessage("기기 접근 권한이 거부되었습니다: $deniedPermissions. 앱이 종료됩니다.")
            .setPositiveButton("설정으로 이동") { _, _ ->
                // 사용자를 권한 설정 화면으로 이동
                val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
                    data = Uri.fromParts("package", packageName, null)
                }
                startActivity(intent)
            }
            .setNegativeButton("앱 종료") { _, _ ->
                // 권한 설정 거부 시 앱 종료
                finish()
            }
            .show()
    }
}

```

이후, App과 FCM을 연결하기 위해 App을 작동시킨 뒤 Logcat 부분에서 Firebase 토큰 값을 가져와 FCM에 저장한다. 그리고, FCM 콘솔에 들어가 토큰 내용을 입력한다.

![[KakaoTalk_Photo_2024-06-07-17-59-00.png]]

<테스트 메세지 전송> 버튼을 누르면, <기기 미리보기> 부분에 나와 있는 것과 같은 메세지를 FCM -> APP으로 발송한다. 에뮬레이터에서 메세지(Intent로 받아온 Toast)가 정상적으로 출력되면 연결에 성공한 것이다. 아래 사진은 각각 인앱의 Toast 표시, 인앱이 아닐 경우의 알림 표시이다. 

![[KakaoTalk_Photo_2024-06-07-17-58-48.jpeg]]
![[KakaoTalk_Photo_2024-06-07-17-58-42.png]]

FCM 콘솔에서 특정 이벤트를 위해 메세지를 예약할 수도 있다. 예약 기능을 사용할 경우, FCM 서버에서 대상 기기로 지정한 시점에 일괄적으로 메세지를 발송한다. 하지만, 실험 결과 정확한 타이밍에 푸시 메세지를 보내는 것이 아니라 약 1분 정도의 간격이 발생하는 것을 확인하였다.

![[KakaoTalk_Photo_2024-06-07-17-59-05.png]]

#### 주의할 점
기기 별로 액세스 토큰이 다르게 발급되기 때문에, 앱 작동 및 기사 로그인 시 액세스 토큰을 비롯한 정보를 서버에 전달하여 FCM과 연계하도록 설정해 놓을 필요가 있다.