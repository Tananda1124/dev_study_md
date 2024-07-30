# **Docker 설치**
yum-utils 설치
```bash
# yum-utiles 패키지 설치
sudo yum -y update
sudo yum install -y yum-utils
```
Docker Engine을 설치할 수 있도록 저장소를 추가합니다.

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

도커 리포지토리가 사용되도록 설정되었으므로 다음을 입력하여 yum을 사용하여 최신 버전의 도커 CE(Community Edition)를 설치합니다.

```
sudo yum install docker-ce docker-ce-cli containerd.io -y
```

# **Docker 버전 확인**

```
# 버전확인 
docker -v
```

# 방화벽 설정

firewall-cmd 방화벽이 활성화가 되어있을 경우 예외를 등록해줍니다.

```
$ sudo firewall-cmd --permanent --zone=trusted --change-interface=docker0

>>
success

$ sudo firewall-cmd --zone=public --add-masquerade --permanent

>>
success

$ sudo firewall-cmd --reload

>>
success
```

# Docker 실행
```bash
sudo systemctl start docker
```
- Docker 실행 테스트
    - hello-world 예시 이미지 띄우기

```bash
sudo docker run hello-world
```

- Docker 실행 여부 확인

```bash
$ systemctl status docker

● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since 화 2023-03-14 11:00:53 KST; 3min 26s ago
     Docs: https://docs.docker.com
 Main PID: 14151 (dockerd)
    Tasks: 8
   Memory: 27.3M
   CGroup: /system.slice/docker.service
           └─14151 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
...
Hint: Some lines were ellipsized, use -l to show in full.
```

# 부팅 시 Docker 자동 시작 설정
- `systemctl` 명령어를 사용하여 부팅 시 docker가 자동으로 실행되도록 설정 가능

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

- 자동 실행 설정을 해제하려면 `disable` 명령어 사용

```bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```
# root 권한이 아닌 일반 사용자로 Docker 관리하기
- Docker 데몬은 항상 `root` 사용자로 실행됨
- `docker` 명령어 사용 시 `sudo`를 사용하지 않으려면 `docker` 그룹을 생성하고 사용자 추가 필요

```bash
# docker 그룹 생성 (Docker 설치 시 자동으로 생성되어있을 수 있음)
sudo groupadd docker

# docker 그룹에 사용자 추가
sudo usermod -aG docker $USER

# 그룹 변경 사항 활성화
newgrp docker
```

- sudo 명령어 없이 docker 명령어를 사용할 수 있는지 테스트

```bash
# 실행 중인 컨테이너 확인
docker container ls

# docker 이미지 목록 확인
docker images
```
# Uninstall
```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```
- `/var/lib/docker/`에 저장된 이미지, 컨테이너, 볼륨, 네트워크는 자동으로 지워지지 않음. 수동으로 지우는 것 필요 (131서버는 /data/docker-images)

# 참고
- [https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)
- [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)