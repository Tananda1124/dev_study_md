### Ollama
Ollama란, 다양한 LLM 모델을 최소한의 코드만으로 작동시킬 수 있도록 만든 프레임워크다. Huggingface의 Transformers보다 사용하기 쉽고, 빠르게 여러 모델을 사용해 볼 수 있는 장점이 있다.

### Ollama 서비스 시작하기
Ollama 서비스를 실행시키는 방법은 여러 가지가 있다. 개인 pc에서 가볍게 사용할 목적이라면 Windows, MacOS, Ubuntu에서 직접 다운받아 실행하는 것을 추천한다.
하지만, 업무 환경이라면 빠른 테스트와 서버의 유지보수를 위해 Docker 컨테이너를 활용하는 쪽을 더 권장한다.

우선, 최신 버전의 ollama를 설치한다.
```
sudo docker pull ollama/ollama
```

nvidia GPU 사용 설정이 기존에 완료되었다면, 서비스를 시작하는 부분으로 바로 넘어간다.
```
nvidia-smi
```
이 명령어를 실행했을 때, 그래픽 카드 정보가 나타나지 않는다면 GPU를 사용할 수 있도록 세팅해야 한다.
###### GPU 세팅
GPU를 세팅하기 위해 리포지토리 등을 등록하고 설치하는 부분인데, 쭉 따라하면 보통 잘 실행된다.
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \ | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \ | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \ | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list sudo apt-get update

curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo \ | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

sudo yum install -y nvidia-container-toolkit

sudo nvidia-ctk runtime configure --runtime=docker sudo systemctl restart docker
```

###### ollama 실행
이렇게 가져온 docker 이미지의 기본 서비스 주소는 11434이므로, 이 이미지를 원하는 포트에 매핑하여 실행한다.
```
sudo docker run -d --gpus=all -v ollama:/root/.ollama -p <원하는 포트 번호>:11434 --name ollama ollama/ollama
```