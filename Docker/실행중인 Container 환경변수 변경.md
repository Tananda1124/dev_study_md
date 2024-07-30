1. container id 알아내기

```bash
docker inspect _my_container_  
```

2. config 파일 변경
```bash
vi /var/lib/docker/containers/_{container-id}_/config.v2.json
```

3. docker 컨테이너 재시작
```bash
docker restart _my_container_
```