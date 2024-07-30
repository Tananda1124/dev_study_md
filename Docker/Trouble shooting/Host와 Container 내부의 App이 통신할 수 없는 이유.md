![[Pasted image 20240412163710.png]]
# TLDR

Docker 를 사용할 때 서로 다른 Container 의 app 끼리 통신이 되지 않는 경우가 간혹 있었는데 그 이유를 알기 위해서는 docker network의 구성 방식에 대해서 알 필요가 있었다.

나의 경우 host 에서 docker container 에게 request를 보낼 때 port-forwarding을 하여서 localhost:PORT 와 같은 방식으로 처리를 하고 싶었는데 connection refused가 발생하는 문제가 있었다.

결과적으로 보면 굳이 network interface를 분리하지 말고 host network타입으로 구성하여 사용하거나, 해당 docker container 의 app 에서 0.0.0.0 에 대한 binding처리를 해줘서 모든 ip 값에 대해서 ( 혹은 해당 Container의 ip ) 요청을 받을 수 있도록 해줬어야 했다.

# Docker Network 구성

Docker의 초기 network 구성에는 bridge, host, none 3가지 network가 존재한다.

```bash
ubuntu@ip-***-***-***-***:~/test$ docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
0dbdaad214cb   bridge                         bridge    local
7e304153f372   host                           host      local
6fd3db0d0256   none                           null      local
```

Docker를 사용하게되면 default로 docker 네트워크를 host 네트워크와 분리시켜서 네트워크 환경을 구성한다.

따라서 Container를 만들 때 따로 설정을 해주지 않으면 bridge driver 타입의 bridge 네트워크로 할당되어 만들어지게 된다.

lo network interface 는 loopback 타입으로 127.0.0.1 자기 자신을 의미한다.

```bash
docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
80f4d8e7277c   test      "tail -f /dev/null t…"   21 minutes ago   Up 21 minutes             keen_johnson
9b973d995e13   test      "tail -f /dev/null"      22 minutes ago   Up 22 minutes             hungry_bhaskara


docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "0dbdaad214cb7d53e9d75437e380fc2fd1b3c65627d1a5cf9bbea2bce2c12323",
        "Created": "2022-11-15T07:08:26.48604911Z",
        "Scope": "local",
        "Driver": "bridge",
        ...
        "Containers": {
            "80f4d8e7277c68f36d1bf34ffc81b761137ed60a93c8af09ca0bc732f829d201": {
                "Name": "keen_johnson",
                "EndpointID": "9ed465c4a3e9ad1abf8404d0902e21b7c3bcb6f772184b6c955db0c41cd2ec00",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

## Docker network 구성도

위와 같이 2개의 Container가 구동돼있는 상태에서 host 에서 network interface 를 보게 되면 docker0, vethxxx, vethxxxx 가 형성되게 된다.

```bash
ubuntu@ip-***:~/test$ ifconfig

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 32000
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:70ff:feb1:bb3d  prefixlen 64  scopeid 0x20<link>
        ether 02:42:70:b1:bb:3d  txqueuelen 0  (Ethernet)
        RX packets 84589192  bytes 4971846816 (4.9 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 187633383  bytes 470947705206 (470.9 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 747065  bytes 3805910056 (3.8 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 747065  bytes 3805910056 (3.8 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth33105d6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::642b:11ff:fe6d:1420  prefixlen 64  scopeid 0x20<link>
        ether 66:2b:11:6d:14:20  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14  bytes 1076 (1.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

host의 network interface 형태는 위와 같이 나오고 각각의 container 에서는 lo(127.0.0.1), eth0 인터페이스를 갖고 있다.

```bash
root@80f4d8e7277c:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
        RX packets 15  bytes 1146 (1.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

정리하면 각각의 Container 들은 본인의 eth0 가 존재하고 이를 vethxxx 형태의 가상 네트워크 인터페이스로 연결짓게 되고 이를 docker0 bridge 가 관리하는 형태로 구성된다.

위 형태를 간단한 모식도로 보자면 아래와 같다.
![[Pasted image 20240412163729.png]]
## Host -> Conatiner server

그렇다면 내가 Container 내부에서 server 를 올린다고 할 때 docker 에서 다음과 같이 실행을 할 수 있다.

```null
docker run --name test_1 example
```

단순 container만 띄운 상태에서 host 에서 localhost:5000 ( 127.0.0.1:5000) 를 사용한다면 내가 원하는 Container 의 server로 요청이 들어갈까?
![[Pasted image 20240412163740.png]]
위와 같이 따로 조치를 취해주지 않는다면 단순 Host의 lo 에 요청을 날리는 것에 지나지 않아 connection refused error가 발생할 것이다.

## Port forwarding

Port forwarding을 사용하게 된다면 Container에 요청을 보낼 수 있게 된다.

```null
docker run -p 5000:5000 --name test_1 example
```

host lo 로 request 를 날리게 됐을 때 forwarding 설정한 모든 network interface의 port로 요청을 전달한다.
![[Pasted image 20240412163755.png]]
하지만 이렇게만 한다고 해서 문제가 완전히 해결되지는 않는다.  
나의 경우 container안의 server 가 127.0.0.1 (localhost:5000)을 binding 하였기 때문에 해당 ip, port 로 오는 요청이 아니면 받을 수가 없었기 때문에 아무런 응답을 받지 못하는 문제가 생겼었다.

따라서 나와같은 문제를 해결하기 위해서는 두 가지 정도 방법이 있을 것 같다.

1. server 에서 0.0.0.0 binding 을 하여 모든 ip 에서 들어오는 요청을 받을 수 있도록 변경 혹은 Container의 ip로 설정.
2. custom or default network를 사용하지 말고 host network로 설정하여서 network interface를 격리시키지 않고 사용하는 방법.

2번 같은 경우는 다음과 같은 방식으로 적용할 수 있다.

```bash
docker run --network host --name test_1 example
```

**Host network에 연결할 경우 따로 포트를 정해줄 필요가 없다 어차피 host 의 Interface를 공유하기 때문!**