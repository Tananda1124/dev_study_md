### OpenWebui
OpenWebui는 언어 모델을 사용하기 위해 ChatGPT와 같은 GUI 환경을 제공해 준다. 단순 GUI 뿐 아니라 User 등록, 대화 세션 저장, 새 모델 테스트 등의 활동을 쉽게 해 볼 수 있다.
물론, ChatGPT API를 이용해서 GPT 서버에 대화 기록이 남지 않도록 할 수도 있다. 지식 베이스를 이용한 대화 또한 지원한다.

### 설치 및 주요 기능
#### 설치
OpenWebui는 Ollama 및 ChatGPT API와 연계할 수 있도록 설계되었다. Ollama가 이미 설치되어 있다면, 바로 Docker image로 다운받아 연결하면 된다.
```Docker
docker run -d -p <실행할 포트 번호>:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```
위의 코드를 이용해 host.docker.internal로 기본 Ollama 주소가 설정되었지만, 나중에 변경할 수 있으니 크게 신경쓸 필요는 없다.

접속 방법은 매우 간단하다. 
http://<docker 컨테이너를 실행하고 있는 ip 주소>:<실행한 포트 번호>
이 주소로 접속하면 된다.

#### 접속 및 Admin 패널
접속해서 처음 계정을 생성하면, 자동으로 Admin 계정으로 등록된다. 그 이후에 생성하는 계정은 pending 상태로 admin이 허락해 주어야만 활동할 수 있으니 주의해야 한다.
![[스크린샷 2024-06-27 오후 4.35.29.png]]
계정 이름 부분을 누르면, 위 이미지와 같은 메뉴가 나온다. 이 중, 'Admin Panel' 부분에서 전반적인 서비스 관리가 가능하다.

#### Openai, Ollama 서버 연결 설정
Admin Panel은 여러 가지를 활용할 수 있지만, 가장 기본이 되는 것은 언어 모델이 작동하는 서버와 연결하는 것이다.
Admin Panel - Connections 메뉴에서 이것을 설정할 수 있다.
![[스크린샷 2024-06-27 오후 4.42.38.png]]
OpenAI API는 키를 발급받을 경우 바로 사용이 가능하다. 키를 등록하는 순간 ChatGPT와 완전히 동일한 기능을 사용할 수 있다고 보면 된다. 지원하는 모델은:
- GPT-3.5-turbo
- GPT-4-turbo
- GPT-4o
등과 같은 Openai의 거의 모든 모델을 사용 가능하다.

Ollama API 부분은 좀 복잡한데, 아래 스니펫으로 정리해 놓았다.
###### Ollama가 호스트 Windows, MacOS, Ubuntu에서 실행 중일 때
```
http://host.docker.internal:<Ollama 포트 번호, 보통 11434>
```

###### Ollama가 같은 Ubuntu의 Docker에서 실행 중일 때
이 때는, 외부 ip 주소를 하나 더 만드는 것은 비효율적이다. 따라서, 컨테이너 간 브릿지 통신을 이용해야 한다.
아래 명령어로 컨테이너의 브릿지 ip 정보를 가져온다.
```
sudo docker network inspect bridge
```
![[스크린샷 2024-06-27 오후 4.50.18.png]]
예시 이미지는 open-webui 컨테이너지만, 실 사용시 ollama가 작동하는 컨테이너의 브릿지 ip 주소를 알아내야 한다.
이렇게 주소를 알아낸 이후, Connection 부분에 Sync를 진행한다.
```
http://<docker 브릿지 ip>:11434
```

###### Ollama가 아예 외부 서버에서 실행 중일 때
DHCP, 포트 포워드가 완료된 서버가 실행 중이라면 간단하다.
```
http://<Server IP>:<Port>
```

#### 언어모델 추가하기
##### Ollama에 등록된 모델
![[스크린샷 2024-06-27 오후 5.19.49.png]]
Admin Panel의 Pull model 부분에서 검색해 추가하면 된다.
##### Ollama에 등록되지 않은 모델
Ollama에 등록되지 않은 모델도 추가할 수 있다.
우선, 모델의 .gguf 파일 다운로드 링크가 있어야 한다. huggingface 등의 커뮤니티에서 gguf라고 제목이 붙어있는 모델이 있는데, 예시로 eeve:10.8B를 다운받아 보자.

https://huggingface.co/heegyu/EEVE-Korean-Instruct-10.8B-v1.0-GGUF
모델 카드 페이지로 들어가서, 'Files and versions' 탭을 클릭한다. 파일 목록은 다음과 같은데, 마우스 우클릭으로 이미지에 표시된 부분의 다운로드 링크를 복사한다.
![[스크린샷 2024-06-27 오후 5.27.07.png]]
Q4와 같은 표현은 어떤 양자화 알고리즘 모델인지 표시이다. 성능은 Q5가 보통 좋다.

링크를 복사했다면, Ollama가 있는 환경으로 돌아와 모델을 다운로드 받고, 확장자를 .gguf로 만든다.
```Linux
sudo curl https://huggingface.co/heegyu/EEVE-Korean-Instruct-10.8B-v1.0-GGUF/resolve/main/ggml-model-Q4_K_M.gguf?download=true

mv 'ggml-model-Q4_K_M.gguf?download=true' eeve.gguf
```

이제, 모델에 대한 기본 설정 부분인 ModelFile을 작성하면 된다. 같은 경로에 ModelFile이라는 이름의 파일을 만든다.
```Linux
touch Modelfile
vi Modelfile
```

다음과 같은 정보를 ModelFile에 입력한다.
간혹 Modelfile이 없는 경우가 있는데, 파인튜닝 이전 원본 모델의 Modelfile을 이용해도 된다. (Llama3 Modelfile 등 검색)
```ModelFile
FROM eeve.gguf

TEMPLATE """{{- if .System }}
<s>{{ .System }}</s>
{{- end }}
<s>Human:
{{ .Prompt }}</s>
<s>Assistant:
"""

SYSTEM """A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions."""

PARAMETER stop <s>
PARAMETER stop </s>
```

이제 설정을 마쳤으니, Ollama에 이 모델을 등록한다.
```Ollama
ollama create EEVE:10.8B -f ModelFile
ollama list
```
![[스크린샷 2024-06-27 오후 5.34.11.png]]
위 이미지와 같이 출력되었다면, 정상적으로 모델이 실행 가능하게 된 것이다.

이제, Admin Panel의 다음 부분을 클릭해 모델 리스트를 새로고침하면 정상적으로 채팅할 수 있다.![[스크린샷 2024-06-27 오후 5.34.32.png]]