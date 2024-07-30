MVVM 패턴 개발을 매끄럽게 하기 위하여, Jetpack Compose를 View 부분에 적극 사용하기로 하였다. Jetpack Compose를 이용하면, 기존 androidx의 xml 파일 없이 Kotlin으로 View를 구성할 수 있게 된다.
이렇게 구성한 View는 각각의 객체로 바인딩하여 ViewModel의 상태 관리를 받게 된다. 아래 글은 기본적인 UI 생성 및 각 요소를 꾸미는 방법이다.

#### Text
기존 Androidx의 TextView와 같은 UI 요소이다. 코드로 구현하면 다음과 같다.
```

```