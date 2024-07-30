 도커 컨테이너는 가상의 프로세스다. 도커 컨테이너의 구동이 일반적인 프로세스와 구별되는 점 중 하나는 독립적인 네트워크를 가진다는 점에 있다. 각각의 컨테이너 마다 개별적인 네트워크 망을 지닌다는 말이다. 그런데 사실 이건 물리적으로 불가능한 일이다. 컨테이너가 어쨋든 프로세스 중의 하나라면 서버의 네트워크 망 안에 포함될 수밖에 없는 것이다. 그렇기에 엄밀히 말해 도커 컨테이너는 '독립적인 네트워크'를 가지기보다는 '분리된 네트워크'를 가진다. 그럼 어떻게 컨테이너는 분리된 네트워크를 지닐 수 있을까? 이 의문이 답이 곧 Bridge이다. 

 잠깐 덧붙이자면 앞서와 같이 분리된 네트워크를 가지는건 도커 컨테이너의 일반적인 설정이지 절대적인 성격은 아니다. 도커는 여러가지 종류의 네트워크 구성 옵션을 제공한다. 이 중에는 host 서버와 네트워크를 분리하지 않는 옵션도 있다. 여기서 말하는 Bridge는 컨테이너 네트워크의 기본 옵션이다. ([https://docs.docker.com/network/](https://docs.docker.com/network/))

 네트워크를 분리한다는 말을 아주 단순화하면 ip를 나눈다는 말이다. 그러니까 컨테이너가 분리된 네트워크를 가진다는 말인 즉 컨테이너가 host와는 다른 ip를 가진다는 말이다. 교각을 뜻하는 Bridge가 여기서 가지는 의미는 host와 컨테이너를 잇는 라우팅 경로다. 컨테이너의 ip는 실제 주소가 아닌 가상의 주소다. 컨테이너에 가상의 주소를 부여하는 원리는 솔직히 잘 모르겠다. 아마 공유기가 망에 속한 기기들에게 ip를 뿌리는 DHCP 기술과 엇비슷한 무엇이 아닐까 추측한다. 중요한 건 이렇게 공유기 역할을 하는 객체가 있다는 점이다. 

 도커 엔진에 등록된 네트워크 객체를 찾아보면 기본적으로 아래와 같은 객체들이 나온다. 여기서 bridge 드라이버로 표시된 객체가 가상의 ip를 부여하는 공유기의 역할을 한다. 모든 도커 컨테이너는 하나씩의 네트워크 객체를 가져야 한다. 아무런 옵션을 주지 않을 경우 아래의 bridge/bridge 객체가 부여된다. 

```
docker network ls

NETWORK ID     NAME          DRIVER    SCOPE
2e700696ccac   bridge        bridge    local
fca54011ceb9   host          host      local
d5a973f84ac1   none          null      local
```

 inspect는 도커 엔진이 가진 특정 객체의 설정을 조사하는 명령어다. inspect에 bridge를 던져보면 다음의 결과가 나온다. 다른건 몰라도 되고 저 아래의 Subnet만 주목하면 된다. 저 표시는 특정 IP의 표시가 아니라 172.17.0.0 ~ 172.17.255.255 이라는 IP 집합의 범위다. 기본 Bridge 모드로 컨테이너를 띄울 경우 이 65536 가지의 IP 중 하나가 컨테이너의 가상 IP로 할당된다. 

```
docker inspect bridge

[
    {
        "Name": "bridge",
        "Id": "2e700696ccac8b45ca040aeca60cefa7285c383ed3ea761d0c04be55a984f042",
        "Created": "2021-06-04T00:08:18.48663228Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"   <- 가상 IP 범위
                }
            ]
        },
 ...
```

 못 미덥다면 아무 컨테이너나 대충 만들어보고 그걸 inspect에 던져보면 된다. 놀랍게도 172.17.0.3 이라는 가상 주소가 컨테이너에 부여되었다!!! 사실 놀라운게 아니다. 짚고 넘어갈 점은 생겨나는 컨테이너마다 서로 다른 가상 IP를 가진다는 점이다. 즉, 컨테이너의 네트워크는 host와도 분리됨과 동시에 다른 컨테이너와도 분리된다. 

```
docker inspect container

...
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "2e700696ccac8b45ca040aeca60cefa7285c383ed3ea761d0c04be55a984f042",
                    "EndpointID": "31efa52464fb76892263defe0022a629d511fb5fbe274d8f3f9b6eb2f314c2f8",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",   <- 가상으로 할당된 IP
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

 여기까지는 기본 배경 지식이라고 하겠다. 멋쟁이 줄 하나 긋고 본격적인 이야기를 시작하겠다. 

---

### ■ 가상 IP를 가졌을 때 일어나는 일. 

 동일한 host 내 두 개의 프로세스 A, B 가 있다고 해보자. A 에서 B 로 http 호출을 보낸다고 했을 때, 이 패킷에는 A와 B가 공유하는 host의 IP가 적혀있을 것이다. 이 경우 호출은 컴퓨터가 기본적으로 내장한 별로의 router 프로세스를 통해 전달된다. A의 호출을 라우터가 읽고 그대로 B로 전달하는 셈이다.
 ![[Pasted image 20240412165216.png]]
 반면 가상의 IP를 부여받거나해서 A와 B가 서로 다른 IP를 가졌을 경우엔 호출 패킷엔 서로 다른 Origin/ Destination IP가 적혀진다. 이때의 호출은 라우터로만 처리할 수 없고 네트워크 시스템의 해석을 거쳐야 한다. 쉽게 말해 OSI의 아래 계층을 통과해야 한다.
 ![[Pasted image 20240412165228.png]]
  이 과정을 샅샅이 알 필요는 없다. 사실 나도 모른다. 알아둘 점은 하나다. 프로세스가 동일한 IP를 사용하는 전자의 경우엔 프로세스 간의 호출이 프로세스 단의 처리만으로 완료된다는 것이고, 프로세스가 서로 다른 IP를 사용하는 후자의 경우엔 프로세스 간의 호출이 네트워크 시스템 즉 하드웨어 단의 처리까지 필요로 한다는 점이다!
### ■ 도커 컨테이너 간 통신 체계. 

 이와 같은 이유로 하나의 host 내 구동되는 컨테이너들은 서로 분리된 망을 지닌다. 아래처럼 말이다!
 ![[Pasted image 20240412165249.png]]
   사실 이는 아주 암담한 광경이다. 컨테이너만 세 개 띄웠는데 거의 클러스터나 다름 없는 운영이 필요하기 때문이다. 서로 의존되는 컨테이너와 컨테이너를 일일이 연결해주는 일은 끔찍하게도 복잡한 일이다. 이런 문제를 해결하고자 기본적으로 같은 bridge를 사용하는 컨테이너들이 공유하는 gateway를 제공한다. 앞서의 컨테이너 조사 결과를 자세히 보면 ip 주소 위에 Gateway 주소도 있다. Gateway 주소는 관례상 subnet 범위의 첫번째 값을 사용한다. 달리 말해 x.x.0.1 의 값을 지닌다. 

```
docker inspect container

...
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "2e700696ccac8b45ca040aeca60cefa7285c383ed3ea761d0c04be55a984f042",
                    "EndpointID": "31efa52464fb76892263defe0022a629d511fb5fbe274d8f3f9b6eb2f314c2f8",
                    "Gateway": "172.17.0.1",     <- Gateway 주소
                    "IPAddress": "172.17.0.3",   <- 가상으로 할당된 IP
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

 암튼 이런 Gateway의 존재로 컨테이너간 통신을 아주아주 원활하게 관리할 수 있다. Gateway를 고려한 컨테이너 간 통신 체계는 다음과 같다. 컨테이너는 서로 직접 연결하는 것이 아니라 이처럼 Gateway 를 통해서 통신한다. 또한 host와 컨테이너간의 통신도 Gateway를 거친다. 컨테이너들이 서로 다른 주소를 사용하고 있음에도 port가 겹치면 안되는 이유가 여기에 있다. 또한 외부에서 컨테이너에 접근할 때 컨테이너의 주소가 아닌 host의 주소를 사용하는 이유도 여기에 있다. 외부에서 컨테이너에 직접 접근은 불가능하다.
 ![[Pasted image 20240412165304.png]]
  *이건 한 가지 팁이다. 컨테이너 간 통신을 할 때 종종 host의 IP를 그대로 사용하는 경우가 있다. 가령 container1이 백엔드 서버고 container 2가 DB 일 때, jdbc driver 경로를 ~192.x.x.x:8888/~ 이런 식으로 적는 경우 말이다. 이건 굉장히 좋지 않은 방법이다. 컨테이너 간 통신은 되도록 Gateway만 이용하는 것이 좋다. 말하자면 jdbc driver 경로를 ~172.17.0.1:8888/~ 이런 식으로 적는 것이다. 

 host 주소를 적는 경우의 문제점은 다음과 같다. 우선 host의 네트워크 처리를 발생시킨다는 비효율성 문제가 있다. Gateway의 경우엔 앞서의 라우터처럼 하드웨어 처리가 필요없는 프로세스 단의 처리만을 필요로 한다. 또한 host 네트워크를 통과하는 경우, host 방화벽에 막혀서 호출이 불가능할 수 있다. 가령 host에서 8888 포트를 막아놨을 경우, 두 컨테이너가 서버 내에 있음에도 서로 통신할 수 없는 어처구니 없는 상황이 일어날 수 있는 것이다. 

### ■ 공용 네트워크 생성

 여기서 말하는 공용 네트워크는 기본 bridge가 아닌 동일 서비스를 이루는 여러 컨테이너가 공유하는 네트워크를 말한다. 아주 쉽게 말하자면 Gateway를 서비스 별로 만든다고 보면 된다. 이유는 간단하다. 그렇게 하지 않을 경우 bridge 옵션으로 생성된 모든 컨테이너가 동일한 Gateway를 사용하게 되는 것이다. Gateway에 부하가 걸릴 수 있는 점과 서로 독립된 서비스 간에 API 호출이 가능해져 보안이 취약해진다는 점을 고려했을 때 이는 좋지 않은 설계다. 

 이러한 상황을 타개하는 가장 단순한 방법이 서비스 별 별도의 공용 네트워크를 만드는 방법이다. 절차는 다음과 같다. 

1. 네트워크 객체 생성
2. 컨테이너와 네트워크 객체 결합

#### 1 . 네트워크 객체 생성

 네트워크 객체를 만들 때는 Gateway 설정만 잘해줘도 기본적인 목적 달성엔 충분하다. 그런데 이 Gateway 설정을 위해선 subnet 설정도 해주어야 한다. 말만 어렵지 단순한 작업이다. 아래의 명령어와 옵션으로 객체를 생성하면 된다. 

 여기서의 subnet 정보는 말그대로 가상의 정보다. 그러니 다른 주소와 겹치지 않는 한에서 아무렇게나 주어도 상관없다. 그렇지만 도커의 기본 설정이 172.17.0.0 라는 점을 감안하여 이와 엇비슷한 172.~.0.0 의 주소를 사용하는 것이 좋겠다. 

```
docker network create \
  --driver=bridge \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  customNetwork
  
  ---
  
 docker network ls
  
  NETWORK ID     NAME            DRIVER    SCOPE
6b83ab4cd7ef   customNetwork   bridge    local
...

https://docs.docker.com/engine/reference/commandline/network_create/
```

#### 2 . 컨테이너와 네트워크 객체 결합

 결합은 두 가지 방법이 있다. 첫째는 구동 중인 컨테이너와 객체를 결합시키는 방법이고, 둘째는 컨테이너를 구동 시킬 때 객체를 결합하는 방법이다. 적용하는 명령어가 조금 다르다. 

```
# 구동 중인 컨테이너와 결합

docker network connect customNetwork container1

--

# 컨테이너 구동 시킬 때 결합

docker run --network customNetwork --name container1 containerImage:1

https://docs.docker.com/engine/reference/commandline/network_connect/
```

 이렇게 하면 서비스에 맞춤화된 별도의 공용 네트워크가 생성/적용 된다!!