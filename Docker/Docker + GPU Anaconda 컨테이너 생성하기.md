Docker에서 이미지를 생성하면, 기존 호스트에서 사용하던 gpu를 컨테이너 내부에서 사용하지 못한다. 아래 명령어를 응용해서 컨테이너를 생성하면, nvidia 드라이버와 Cuda를 컨테이너 내부에서 그대로 사용할 수 있다.

```
sudo docker run --gpus all -it -p <포트 번호, -p옵션은 중복 사용하여 다중 포트 개방 가능> \
--name <새로 생성할 컨테이너 이름> <docker image 이름> /bin/bash
```

GPU가 연결되었는지 확인해 보자.
```
nvidia-smi
```

GPU가 성공적으로 연결된 것을 확인했다면, python 환경을 자유롭게 오가기 위해 Anaconda를 설치해야 한다.
```
# root 계정으로 가정하였음, root 계정이 아닐 경우 sudo 필요
apt update
apt install curl wget

#Anaconda 설치 파일 다운로드
curl --output <curl 받을 파일 경로, 파일 이름> https://repo.anaconda.com/archive/Anaconda3-2022.10-Linux-x86_64.sh
```

Anaconda 설치 파일을 다 받으면, 이것을 실행한다.
```
sha256sum <설치 파일>

bash <설치 파일>
```

Anaconda 실행 파일을 환경 변수에 추가하여, 언제나 사용할 수 있도록 만든다.
```
vi ~/.bashrc

export PATH=~/anaconda3/bin:~/anaconda3/condabin:$PATH

source ~/.bashrc
```

위의 과정이 완료되었다면, 버전 체크와 새로운 conda 환경 생성을 진행한다.
```
conda -V

conda create -n <새 가상환경 이름> python=<새 환경의 python 버전>

conda activate <새 가상환경 이름>
```