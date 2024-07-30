### 아래 오류가 나타나면 해볼 수 있는 작업
- Error creating default "bridge" network: cannot create network (docker0): conflicts with network (docker0): networks have same bridge name
- failed to start daemon: Error initializing network controller: error creating default "bridge" network: cannot create network 91dc612009a3cdcef196443e0e23d5e283cb6a40bf3231bae2729b9158761404 (docker0): conflicts with network 4b921bb6b47a6df2ae457ec658b29acb35b7279537a4c264986331a23c284bfe (docker0): networks have same bridge name

## Solution1
For me I downgraded my OS (Centos Atomic Host in this case) and came across this error message. The docker of the older Centos Atomic was 1.9.1. I did not have any running docker containers or images pulled before running the downgrade.

**_I simply ran the below and docker was happy again:_**

```
#docker 네트워크 폴더 경로 확인할 것 (/data/docker-images)
#아래는 기본경로
sudo rm -rf /var/lib/docker/network
sudo systemctl start docker
```

[More info.](https://bugzilla.redhat.com/show_bug.cgi?id=1399398)
## Solution2
The Problem seems to be in `/var/docker/network/`. There are a lot of sockets stored that reference the bridge by its old id. To solve the Problem you can delete all sockets, delete the interface and then start docker **but all your container will refuse to work since their sockets are gone.** In my case I did not care about my stateless containers anyway so this fixed the problem:

```
ip link del docker0
rm -rf /var/docker/network/*
mkdir -p /var/docker/network/files
systemctl start docker
# delete all containers
docker ps -a | cut -d' ' -f 1 | xargs -n 1 echo docker rm  -f
# recreate all containers
```