
```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   633  100   633    0     0   2434      0 --:--:-- --:--:-- --:--:--  2434
100 11.6M  100 11.6M    0     0  5360k      0  0:00:02  0:00:02 --:--:-- 8785k
```

docker-compose를 curl로 다운받습니다.

```
$ sudo chmod +x /usr/local/bin/docker-compose
```

실행 권한을 추가합니다.

```
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

사용자 바이너리 디렉토리로 실행파일을 심볼릭링크를 생성하면서 완료합니다.