#### 개요
Android 기기의 GPS 값을 가져와 MongoDB에 저장하는 과정에 대한 글이다.
Ubuntu 서버 내부에 MongoDB 데이터베이스를 구축하고, 서버 프로그램으로 반복 단일 요청에 강한 Node.js를 사용하였다.
#### Ubuntu - MongoDB 설치
우선, Ubuntu 22.04 LTS에 MongoDB를 설치한다.
```
// Ubuntu 버전 확인
cat /etc/lsb-release
// 기본 의존성 설치
sudo apt-get install gnupg curl

// mongoDB 정보 apt 등록하기
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update

// mongoDB 설치
sudo apt-get install -y mongodb-org
```

설치가 끝나고, mongoDB가 실행 중인지 테스트한다.
```
systemctl mongod status
```
![[스크린샷 2024-06-12 오후 4.21.16.png]]

정상적으로 실행되고 있다면, 데이터베이스를 구축하고, 컬렉션을 생성한다.
```
// mongoDB 쉘 연결 (--port 옵션으로 원하는 포트에서 가동 가능)
mongosh

// DB 목록 출력, DB 변경, 선택한 DB 내부 컬렉션 목록 출력
show databases
use api_test
show collections

// 컬렉션 생성
db.createCollection("api_test")

// 컬렉션 조회
db.api_test.find()
db.api_test.findOne({name : "cjchoi"})
```
![[스크린샷 2024-06-12 오후 4.24.03.png]]
컬렉션이 정상적으로 생성됨을 확인하고, Node.js 서버를 구축한다.
#### Ubuntu - Nodejs 설치 및 서버 구축
Node 최근 버전이 apt 내에 존재하지 않으므로 저장소를 수동 등록해야 한다.
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

node -v
npm -v
```

node가 정상적으로 설치되었다면, node.js 작업 폴더를 생성하고 파일 시스템을 구축한다.
```
sudo mkdir node_project
cd node_project

npm init
npm install express mongodb
```

마지막으로, 서버 파일을 작성하고 서버를 작동시킨다.
```
vi index.js
```

```
// index.js
// http 통신을 사용하기 위해 express 모듈 설치
const express = require('express');
const { MongoClient, ObjectID } = require('mongodb');

const app = express();
const port = 3000;

// mongodb 주소와 포트 입력
const url = 'mongodb://localhost:27017';
// 이전에 생성한 DB 이름과 일치해야 함
const dbName = 'api_test';
let db;

MongoClient.connect(url, { useNewUrlParser: true, useUnifiedTopology: true })
        .then(client => {
		        // DB 연결 확인을 위해 로그에 출력
                console.log('Connected to Database');
                db = client.db(dbName);
                app.listen(port, '0.0.0.0',() =>{
                        console.log(`Server is running on port ${port}`);
                });
        })
        .catch(error => console.error(error));

app.use(express.json());

app.post('/api_test', (req, res) => {
        const data = req.body;
        // api_test 컬렉션에 데이터 추가
        db.collection('api_test').insertOne(data)
        .then(result => res.json(result))
        .catch(error => res.status(500).send(error));
});

app.get('/api_test', (req, res) => {
        db.collection('api_test').find().toArray()
        .then(results => res.json(results))
        .catch(error => res.status(500).send(error));
});
```

```
node index.js
```
#### Android Retrofit API 통신
API에 연결하기 전에, 외부 인터넷에 접속할 수 있는 권한을 앱에 부여해야 한다.
res/xml/network_security_config.xml을 작성하고, AndroiodManifest.xml에 추가한다.
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" >

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <uses-permission android:name="android.permission.INTERNET" />

    <application
    <!-- 추가된 부분 -->
        android:networkSecurityConfig="@xml/network_security_config"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Gps_temp1"
         >
        <activity
            android:name=".MainActivity"
            android:exported="true" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

Retrofit API에 사용할 객체를 설정한다. 요청을 보내고 받을 때의 자료형을 고려하여 작성한다.
```
data class LocationRequest(val lat: Float, val lng: Float, val time: String)
data class ApiResponse(val success: Boolean, val message: String)

interface ApiService {
    @POST("api_test")
    fun location_test(@Body request: LocationRequest): Call<ApiResponse>

    @GET("api_test")
    fun location_get(): Call<List<LocationRequest>>

    @PUT("api_test")
    fun location_put(@Query("time") time: String, @Body request: LocationRequest): Call<ApiResponse>
}

```

API 주소와 기본 설정 객체의 인터페이스를 작성한다.(추후 Activity에서 사용하기 위함)
```
object RetrofitInstance {
// 로컬 환경에서 실험했기 때문에 URL 주소를 에뮬레이터를 실행하는 컴퓨터의 노드 서버 주소로 설정했음
    private const val BASE_URL = "http://10.0.2.2:3000/"

    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val api: ApiService by lazy {
        retrofit.create(ApiService::class.java)
    }
}

```

MainActivity와 연동하여, GPS 값이 화면에 표시될 때마다 서버에 POST 요청을 보내도록 한다.
```
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

                    try {
                        val request = LocationRequest(lat.toFloat(), lng.toFloat(), currentTime.toString())
                        RetrofitInstance.api.location_test(request).enqueue(object : Callback<ApiResponse> {
                            override fun onResponse(call: Call<ApiResponse>, response: Response<ApiResponse>) {
                                if (response.isSuccessful) {
                                    Log.d("MainActivity", "POST 성공: ${response.body()?.message}")
                                } else {
                                    Log.e("MainActivity", "POST 실패: ${response.errorBody()?.string()}")
                                }
                            }

                            override fun onFailure(call: Call<ApiResponse>, t: Throwable) {
                                Log.e("MainActivity", "POST 요청 실패", t)
                            }
                        })
                    } catch (e: Exception) {
                        Log.e("MainActivity", "예외 발생: ${e.message}", e)
                    }
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

![[스크린샷 2024-06-12 오후 5.00.34.png]]

앱을 실행하면 성공적으로 MongoDB에 GPS 값을 입력하게 된다. (계속 같은 값이 들어가는 것은 애뮬레이터 환경이기 때문)![[스크린샷 2024-06-12 오후 5.04.28.png]]