
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