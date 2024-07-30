Continuous Integration : 지속적 통합
Continuous  Deployment : 지속적 배포

CI/CD란, 위 용어처럼 소프트웨어를 개발하며 프로젝트를 지속적으로 통합-배포하는 일련의 과정이라고 할 수 있다.

#### Git- Jenkins
	 git commit
	 jenkins hook 및 자동 pull
	 jenkins 자동 빌드 및 배포
	 이 때, jenkins는 오류가 발생하는 파일은 자동 빌드 목록에서 제외

#### Git - Jenkins 프로젝트 연동
	새로운 아이템 -> Free Style 설정
	소스코드 관리 부분 gitlab 설정 및 gitlab의 ssh 주소 설정
	gitlab에서 jenkins user 생성 및 permission 부여
	gitlab의 webhook 설정 및 jenkins의 secret token 연동
	.sh 파일을 작성하여 자동 빌드 설정 및 로깅