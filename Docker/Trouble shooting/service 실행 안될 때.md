
```
$ sudo service docker start  Redirecting to /bin/systemctl start docker.service  Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
```

실행하려니 실패.

```
$ systemctl status docker.service  ● docker.service - Docker Application Container Engine     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)     Active: failed (Result: exit-code) since 화 2022-04-05 12:17:01 KST; 12s ago       Docs: [https://docs.docker.com](https://docs.docker.com)    Process: 25074 ExecStart=/usr/bin/dockerd (code=exited, status=1/FAILURE)   Main PID: 25074 (code=exited, status=1/FAILURE)
```

상세 사유 없어서 확인이 안됨.

이럴땐 도커 데몬을 직접 실행해본다.

```
$ sudo dockerd --debug  Error starting daemon: Error initializing network controller: Error creating default "bridge" network: failed to allocate gateway (10.252.0.0): Address already in use

```

failed to start daemon: Error initializing network controller: error creating default "bridge" network: Failed to program NAT chain: INVALID_ZONE: docker


-------------------------

### 방화벽 설정

Found out that

```yaml
$ firewall-cmd --get-active-zones
FedoraWorkstation
  interfaces: ens4u1u2 wlp59s0
docker
  interfaces: br-48d7d996793a
libvirt
  interfaces: virbr0
trusted
  interfaces: docker0
```

the interface _docker0_ seems to be in the _trusted_ zone. But there's another zone called _docker_.

So I decided to give it a shot and add it to the _docker_ zone instead.

```yaml
$ sudo firewall-cmd --permanent --zone=docker --change-interface=docker0
$ sudo firewall-cmd --reload
```

Looks like this afterwards:

```yaml
$ firewall-cmd --get-active-zones
FedoraWorkstation
  interfaces: ens4u1u2 wlp59s0
docker
  interfaces: br-48d7d996793a docker0
libvirt
  interfaces: virbr0
```

Seems to work.  
Maybe someone can shed more light on this.

**Edit:** added `firewall-cmd --reload` as pointed out in the comments