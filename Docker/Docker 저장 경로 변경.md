## 개요

도커 관련 개발을 하다보면 도커 컨테이너 이미지들이 잦은풀링으로 인해 많이 쌓여나간다. 신경안쓰고 있다가 어느날 보면 스토리지가 많이 차 있는 것을 볼 수 있다. 특히 본인은 NVIDIA 관련 NGC 컨테이너 이미지들을 사용하는 경우가 많아 이런 이미지들은 수 기가에서 20GB가 넘는 이미지 용량을 볼수도 있다. 그래서 이러한 도커 이미지들의 저장 공간을 여유있는 스토리지로 바꾸는 방법을 알아본다.

### 테스트 환경

- Ubuntu 18.04.6 LTS
- Docker Version: 20.10.17

## 기존 도커 저장 경로 확인

```bash
sudo docker info | grep "Docker Root Dir"

>>> 
 Docker Root Dir: /var/lib/docker
```
따로 변경한 적이 없다면 기본 경로는 `/var/lib/docker` 이다.

## 도커 서비스 중지

돌아가고 있는 도커 서비스를 중지한다. 서비스 중지 시 현재돌고 있는 컨테이너가 모두 종료됨에 주의하자.

```bash
sudo systemctl stop docker.service
sudo systemctl stop docker.socket
```

## 도커 디렉토리 복사 또는 새로운 디렉토리 생성

기존에 가지고 있던 도커 이미지를 보존하기 위해 그대로 디렉토리를 새로운 경로에 복사하려면 다음 명령어를 사용한다.

```
$ sudo rsync -aP /var/lib/docker {새로운 도커 저장경로}
```

기존 이미지가 없거나 새롭게 도커 디렉토리를 만들어 주려면 그냥 생성하면 된다.

```
$ sudo mkdir -p {새로운 도커 저장경로}
```

## 도커 설정에 새로운 저장경로 연결

`/etc/docker/daemon.json` 파일을 수정하여 새로운 경로를 연결한다. 만약 파일이 없다면 생성하자

```
$ sudo vi /etc/docker/daemon.json

>>>
{
    "data-root": "새로운 경로"  # ex) "/data/docker"
}
```

여러 블로그나 다른 해외의 사이트를 검색하다보면 `data-root` 대신 `graph` 로 적는 경우도 있다. 예전 도커 버전은 해당 키로 저장경로를 설정했다하는데 검색해보니 도커 17.05 버전 이후로는 deprecate 되었으니 `data-root` 키로 저장경로를 설정하자

## 도커 서비스 재시작 및 확인

다시 도커 서비스를 재시작하고 우리가 `data-root`로 설정한 경로로 가보면 새로 디렉토리를 만들었을 경우 기존 도커 경로에 있던 파일 디렉토리가 유사하게 생성된 것을 확인할 수 있다.

```
$ sudo systemctl start docker

$ ls {새로운 경로}

>>>
builder   containers  network   plugins   swarm  trust
buildkit  image       overlay2  runtimes  tmp    volumes
```

새롭게 변경 된 도커 경로에서 테스트 이미지 실행 및 기능 실행에 문제가 없다고 판단되면 기존 도커 저장 경로(ex. `/var/lib/docker`)는 삭제해서 저장 공간을 확보한다.