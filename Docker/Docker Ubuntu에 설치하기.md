Ubuntu에 Docker를 설치하는 방법은 다음과 같습니다. 이 단계들을 따라 Docker를 설치하고 설정할 수 있습니다.

기존 Docker 설치 제거 (선택 사항):
기존에 Docker가 설치되어 있다면 이를 제거합니다.

bash
코드 복사
sudo apt-get remove docker docker-engine docker.io containerd runc
필수 패키지 설치:
apt 패키지 관리자가 HTTPS를 통해 저장소를 사용할 수 있도록 필수 패키지를 설치합니다.

bash
코드 복사
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
Docker의 공식 GPG 키 추가:

bash
코드 복사
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
Docker 저장소 설정:

bash
코드 복사
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
Docker 엔진 설치:

bash
코드 복사
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
Docker 서비스 시작 및 자동 실행 설정:

bash
코드 복사
sudo systemctl start docker
sudo systemctl enable docker
설치 확인:
Docker가 정상적으로 설치되었는지 확인합니다.

bash
코드 복사
sudo docker run hello-world
현재 사용자에게 Docker 권한 부여 (선택 사항):
Docker 명령을 실행할 때마다 sudo를 사용하지 않도록 현재 사용자를 docker 그룹에 추가합니다.

bash
코드 복사
sudo usermod -aG docker $USER
변경 사항을 적용하려면 로그아웃 후 다시 로그인합니다.

이제 Ubuntu 시스템에 Docker가 성공적으로 설치되었습니다. docker run hello-world 명령이 성공하면 Docker가 제대로 설치된 것입니다.