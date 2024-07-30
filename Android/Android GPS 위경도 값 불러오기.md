안드로이드 앱에서 실시간 GPS 값을 불러와 사용하는 예제 코드에 대한 문서이다.
#### Permission
```AndroidManifest.xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
우선, Manifest 부분에 사용자의 정확한 위치를 사용해도 되는지 권한 체크를 해야 한다.

#### MainActivity
```MainActivity.kt
package com.example.gps_temp1

class MainActivity : AppCompatActivity() {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var locationButton: Button
    private lateinit var locationTextView: TextView
    private var locationJob: Job? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 뷰 초기화
        locationButton = findViewById(R.id.locationButton)
        locationTextView = findViewById(R.id.locationTextView)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

        // 위치 권한 요청
        val requestPermissionLauncher = registerForActivityResult(
            ActivityResultContracts.RequestPermission()
        ) { isGranted: Boolean ->
            if (isGranted) {
                // 권한이 허용되었을 때
                startLocationUpdates()
            } else {
                // 권한이 거부되었을 때
                locationTextView.text = "위치 권한이 필요합니다."
            }
        }

        // 버튼 클릭 리스너
        locationButton.setOnClickListener {
            if (ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_COARSE_LOCATION
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                // 위치 권한이 없으면 권한 요청
                requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
                return@setOnClickListener
            }
            // 권한이 있을 때 위치 정보 업데이트 시작
            startLocationUpdates()
        }
    }

    private fun startLocationUpdates() {
        locationJob?.cancel() // 기존 작업 취소
        locationJob = CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                updateLocation()
                delay(2000) // 2초마다 업데이트
            }
        }
    }

    private suspend fun updateLocation() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            // 권한이 없으면 return
            return
        }

        fusedLocationClient.lastLocation
            .addOnSuccessListener { location: Location? ->
                if (location != null) {
                    val lat = location.latitude
                    val lng = location.longitude
                    val currentTime = System.currentTimeMillis()
                    locationTextView.text = "현재 위치: 위도 $lat, 경도 $lng\n측정 시간: $currentTime"
                    Log.d("t-dib", "lat : $lat , lng : $lng , time : $currentTime")
                } else {
                    locationTextView.text = "위치 정보를 가져올 수 없습니다."
                }
            }
    }

    override fun onDestroy() {
        super.onDestroy()
        locationJob?.cancel() // 액티비티가 파괴될 때 코루틴 작업 취소
    }
}
```
위의 코드는 코루틴을 이용하여 2초마다 위도와 경도 값을 가져오는 코드이다. 버튼이 눌리면 코루틴이 실행되고, 앱이 종료되면 코루틴을 중지하는 작업이 포함되어 있다. 이러한 방식을 Retrofit2와 연동하면 시간에 따라 사용자가 어느 위치에 있는지 서버에서 추적할 수 있다.

![[Pasted image 20240611160830.png]]

또한, `LocationListener`를 사용하여 위치 변화를 감지할 수 있다. 이를 통해 위치가 변경될 때마다 서버로 위치 정보를 전송할 수 있다. 예를 들어, 최소 50미터 이상 이동했을 때 위치 정보를 업데이트하고 서버에 전송하도록 설정할 수 있다.
아래는 `LocationListener`를 사용하는 예제 코드이다:
#### MainActivity(2)
```MainActivity.kt
class MainActivity : AppCompatActivity(), LocationListener {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var locationManager: LocationManager
    private lateinit var locationButton: Button
    private lateinit var locationTextView: TextView
    private var locationJob: Job? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 뷰 초기화
        locationButton = findViewById(R.id.locationButton)
        locationTextView = findViewById(R.id.locationTextView)

        // FusedLocationProviderClient 초기화
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)
        // LocationManager 초기화
        locationManager = getSystemService(LOCATION_SERVICE) as LocationManager

        // 위치 권한 요청을 위한 런처 설정
        val requestPermissionLauncher = registerForActivityResult(
            ActivityResultContracts.RequestPermission()
        ) { isGranted: Boolean ->
            if (isGranted) {
                // 권한이 허용되었을 때 위치 업데이트 시작
                startLocationUpdates()
            } else {
                // 권한이 거부되었을 때 사용자에게 알림
                locationTextView.text = "위치 권한이 필요합니다."
            }
        }

        // 버튼 클릭 리스너 설정
        locationButton.setOnClickListener {
            // 위치 권한 확인
            if (ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_COARSE_LOCATION
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                // 위치 권한이 없으면 권한 요청
                requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
                return@setOnClickListener
            }
            // 권한이 있을 때 위치 정보 업데이트 시작
            startLocationUpdates()
        }
    }

    // 위치 업데이트 시작 함수
    private fun startLocationUpdates() {
        locationJob?.cancel() // 기존 작업 취소
        // 코루틴을 사용하여 주기적으로 위치 업데이트 요청
        locationJob = CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                updateLocation()
                delay(2000) // 2초마다 업데이트
            }
        }
        
        // 위치 권한 확인
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            return
        }

        // 위치 업데이트 요청 설정 (LocationListener 사용)
        locationManager.requestLocationUpdates(
            LocationManager.GPS_PROVIDER,
            2000L,  // 최소 시간 간격 (밀리초)
            50f,    // 최소 거리 간격 (미터)
            this,
            Looper.getMainLooper()
        )
    }

    // 위치 정보 업데이트 함수 (코루틴 내부에서 호출)
    private suspend fun updateLocation() {
        // 위치 권한 확인
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            // 권한이 없으면 return
            return
        }

        // 마지막 위치 가져오기
        fusedLocationClient.lastLocation
            .addOnSuccessListener { location: Location? ->
                if (location != null) {
                    val lat = location.latitude
                    val lng = location.longitude
                    val currentTime = System.currentTimeMillis()
                    // 위치 정보 업데이트
                    locationTextView.text = "현재 위치: 위도 $lat, 경도 $lng\n측정 시간: $currentTime"
                    Log.d("t-dib", "lat : $lat , lng : $lng , time : $currentTime")

                    // 서버로 위치 정보 전송
                    sendLocationToServer(lat, lng, currentTime)
                } else {
                    locationTextView.text = "위치 정보를 가져올 수 없습니다."
                }
            }
    }

    // LocationListener 인터페이스의 onLocationChanged 구현
    override fun onLocationChanged(location: Location) {
        val lat = location.latitude
        val lng = location.longitude
        val currentTime = System.currentTimeMillis()
        // 위치 정보 업데이트
        locationTextView.text = "현재 위치: 위도 $lat, 경도 $lng\n측정 시간: $currentTime"
        Log.d("LocationListener", "위도: $lat, 경도: $lng, 시간: $currentTime")

        // 서버로 위치 정보 전송
        sendLocationToServer(lat, lng, currentTime)
    }

    // 서버로 위치 정보 전송 함수
    private fun sendLocationToServer(lat: Double, lng: Double, time: Long) {
        // Retrofit 설정 및 서버로 데이터 전송 로직 추가
        val retrofit = Retrofit.Builder()
            .baseUrl("https://yourserver.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        val service = retrofit.create(LocationService::class.java)
        val locationData = LocationData(lat, lng, time)

        // 코루틴을 사용하여 서버로 위치 데이터 전송
        CoroutineScope(Dispatchers.IO).launch {
            try {
                val response = service.sendLocation(locationData)
                if (response.isSuccessful) {
                    Log.d("LocationListener", "서버 전송 성공")
                } else {
                    Log.d("LocationListener", "서버 전송 실패: ${response.code()}")
                }
            } catch (e: Exception) {
                Log.e("LocationListener", "서버 전송 중 오류", e)
            }
        }
    }

    // LocationListener 인터페이스의 onStatusChanged 구현
    override fun onStatusChanged(provider: String?, status: Int, extras: Bundle?) {
        // 위치 제공자의 상태가 변경될 때 호출됩니다.
    }

    // LocationListener 인터페이스의 onProviderEnabled 구현
    override fun onProviderEnabled(provider: String) {
        // 위치 제공자가 사용 가능해질 때 호출됩니다.
    }

    // LocationListener 인터페이스의 onProviderDisabled 구현
    override fun onProviderDisabled(provider: String) {
        // 위치 제공자가 사용 불가능해질 때 호출됩니다.
    }

    // 액티비티가 파괴될 때 호출되는 함수
    override fun onDestroy() {
        super.onDestroy()
        locationJob?.cancel() // 코루틴 작업 취소
        locationManager.removeUpdates(this) // 위치 업데이트 중지
    }
}

// Retrofit 인터페이스 설정
interface LocationService {
    @POST("location/update")
    suspend fun sendLocation(@Body locationData: LocationData): retrofit2.Response<Unit>
}

// 위치 데이터 클래스 정의
data class LocationData(
    val latitude: Double,
    val longitude: Double,
    val timestamp: Long
)
```

이런 방식으로 사용자의 위치가 50미터 이상 이동할 때마다 위치 정보를 업데이트하고 서버에 전송할 수 있다. 
Retrofit 인스턴스를 정의하고 보내는 함수, LocationService post 요청 부분, 위치 데이터 클래스 LocationData 정의 부분을 적절히 활용하면 된다. (이 코드는 포그라운드 실행이므로, 코루틴을 이용해 백그라운드 실행으로 적절히 수정해야 한다.)
이를 통해 효율적으로 위치 정보를 관리하고 사용할 수 있다. (Retrofit을 위해 build.gradle에 의존성을 추가해야 한다.)