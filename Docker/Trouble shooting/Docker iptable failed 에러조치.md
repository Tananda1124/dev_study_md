# Docker iptable failed 에러조치 (docker0: iptables: No chain/target/match by that name

## 개요

docker run 명령어 실행 시 iptable 관련 오류를 발생시키는경우에 대한 trouble 슈팅

## Docker iptable failed

docker run 명령어 실행 시 아래와 같은 오류를 확인했다.
```
...
docker0: iptables: No chan/target/match by that name.
```

```
/usr/bin/docker-current: Error response from daemon: driver failed programming external connectivity on endpoint tender_mestorf (62c1781b3d1f93ab06131f9a88bc038d6dbf8c677c5aca8830c8481ec5a06821):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8080 -j DNAT --to-destination 172.17.0.2:8080 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1)).
```

뭔가 iptable이 손상된것 같은 느낌인데 안그래도 구글링해보니 docker를 사용하던 중 방화벽을 잘못건드려서 발생하는 문제라고 한다. 
- 출처([https://data-newbie.tistory.com/479)](https://data-newbie.tistory.com/479)

해결방법은 다음과 같이 진행하면 된다.

```
$ iptables -t filter -F  
$ iptables -t filter -X 
$ systemctl restart docker
```