### 개요
이 문서는 Android App에서 RestAPI를 사용하기 위해 Retrofit2 라이브러리를 이용하는 기본적 방법이다.
### 프로젝트 설정
API를 주고받기 위해, 인터넷 연결이 필요하다. AndroidManifest.xml 부분에 permission을 허용하도록 하자.
```
// 인터넷 권한 설정
<uses-permission android:name="android.permission.INTERNET" />
```

권한 설정이 완료되었다면, Gradle Script에 Retrofit2를 선언한다.
```
// Gradle 8 이상일 경우
// Retrofit2 - RestAPI
implementation(platform(com.squareup.retrofit2:retrofit2:2.9.0))
implementation(platform(com.squareup.retrofit2:converter-gson:2.9.0))

// Gradle 8 이하일 경우  
implementation 'com.squareup.retrofit2:retrofit:2.9.0'  
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

### 데이터 객체 설정
API를 주고받기 위해, 줄 데이터와 받을 데이터의 Data Class를 생성한다. 이 데이터 형식과 라우팅에 따라 request-response를 진행하게 된다.
```LoginService.kt
// Request - Response로 주고받을 데이터 클래스 생성
data class LoginRequest(val id: String, val pw: String)  
data class LoginResponse(val success: Boolean, val message: String)  

//API 선언
interface LoginService {  
    @POST("login")  
    fun login(@Body request: LoginRequest): Call<LoginResponse>  
}
```

### Retrofit 인스턴스 생성
다른 Activity에서 API를 호출하는 객체를 생성하고 조작할 수 있도록 하기 위해, 인스턴스를 정의해야 한다.
```RetrofitInstance.kt
object RetrofitInstance {
	// 로컬 환경에서 실험했기 때문에 URL 주소를 에뮬레이터를 실행하는 컴퓨터의 노드 서버 주소로 설정했음
    private const val BASE_URL = "http://10.0.2.2:3000/"

    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val api: LoginService by lazy {
        retrofit.create(LoginService::class.java)
    }
}
```

### Activity와 연동
위에서 설정한 API를 실제로 호출하고 실행하는 코드는 Activity 내에서 실행되어야 한다. 이제 이러한 값을 이용해 서버와 여러 값을 주고받고, 객체를 저장하는 등의 작업과 연계할 수 있다.
아래는 서버에서 설정한 id와 비밀번호 값에 따라서 Node 서버와 API 통신하여 로그인 여부를 Toast로 알려주는 간단한 예시이다. 실제 프로젝트에서는 Toast 부분을 서비스 로직으로 수정해야 한다.
```MainActivity.kt
class MainActivity : AppCompatActivity() {  
    private lateinit var binding : ActivityMainBinding  
  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        binding = ActivityMainBinding.inflate(layoutInflater)  
        setContentView(binding.root)  
  
        setButton()  
    }  
  
    private fun setButton(){  
        binding.goButton.setOnClickListener {  
            val id = binding.idEditText.text.toString()  
            val pw = binding.pwEditText.text.toString()  
  
            val request = LoginRequest(id, pw)  
            RetrofitInstance.api.login(request).enqueue(object : Callback<LoginResponse> {  
                override fun onResponse(call: Call<LoginResponse>, response: Response<LoginResponse>) {  
                    if (response.isSuccessful) {  
                        val loginResponse = response.body()  
                        if (loginResponse != null) {  
                            if (loginResponse.success) {  
                                Toast.makeText(this@MainActivity, "Login successful", Toast.LENGTH_SHORT).show()  
                            } else {  
                                Toast.makeText(this@MainActivity, "Login failed: ${loginResponse.message}", Toast.LENGTH_SHORT).show()  
                            }  
                        }  
                    } else {  
                        Toast.makeText(this@MainActivity, "Response error: ${response.code()}", Toast.LENGTH_SHORT).show()  
                    }  
                }  
  
                override fun onFailure(call: Call<LoginResponse>, t: Throwable) {  
                    Toast.makeText(this@MainActivity, "Network error: ${t.message}", Toast.LENGTH_SHORT).show()  
                }  
            })  
        }  
    }  
}
```

### Nodejs 서버
위 코드와 연계하여 사용한, 임시 Nodejs 서버이다. 이 서버는 미리 선언된 딕셔너리 값과 request로 전달된 값이 일치하면 success 여부와 message를 앱으로 응답한다.
```server.js
const express = require('express');
const bodyParser = require('body-parser');
const morgan = require('morgan');

const app = express();
const port = 3000;

app.use(morgan('dev'));  // 로그 설정 추가
app.use(bodyParser.json());

// Simple in-memory user store
const users = [
  { id: 'user1', pw: 'password1' },
  { id: 'user2', pw: 'password2' },
];

// Login endpoint
app.post('/login', (req, res) => {
  const { id, pw } = req.body;

  console.log(`Received login request for id: ${id}, pw: ${pw}`);  // 입력된 데이터를 로그로 출력
  
  const user = users.find(u => u.id === id && u.pw === pw);
  
  if (user) {
    console.log('Login successful');  // 성공 로그
    res.json({ success: true, message: 'Login successful' });
  } else {
    console.log('Login failed: Invalid id or password');  // 실패 로그
    res.json({ success: false, message: 'Invalid id or password' });
  }
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```