필요한 패키지 설치:
OpenSSL을 컴파일하기 위해 필요한 도구와 라이브러리를 설치합니다.

sh
코드 복사
sudo apt-get update
sudo apt-get install -y build-essential checkinstall zlib1g-dev
OpenSSL 소스 파일 다운로드:
주어진 링크에서 OpenSSL 소스 파일을 다운로드합니다.

sh
코드 복사
wget https://www.openssl.org/source/openssl-3.0.14.tar.gz
압축 해제:
다운로드한 tar.gz 파일을 압축 해제합니다.

sh
코드 복사
tar -xzvf openssl-3.0.14.tar.gz
cd openssl-3.0.14
설치 디렉토리 생성 및 설정:
OpenSSL을 설치할 디렉토리를 설정합니다. 일반적으로 /usr/local/ssl에 설치합니다.

sh
코드 복사
sudo mkdir -p /usr/local/ssl
OpenSSL 컴파일 및 설치:
OpenSSL 소스 파일을 컴파일하고 설치합니다.

sh
코드 복사
./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared zlib
make
sudo make install
환경 변수 설정:
새로 설치된 OpenSSL을 사용하도록 시스템의 환경 변수를 설정합니다. ~/.bashrc 또는 ~/.profile 파일을 편집하여 다음을 추가합니다.

sh
코드 복사
export PATH="/usr/local/ssl/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/ssl/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/ssl/lib/pkgconfig:$PKG_CONFIG_PATH"
설정을 적용하기 위해 쉘을 다시 시작하거나 아래 명령어를 실행합니다.

sh
코드 복사
source ~/.bashrc
설치 확인:
OpenSSL이 제대로 설치되었는지 확인합니다.

sh
코드 복사
openssl version

apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-
dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev git

설정 파일 업데이트:
다음 설정을 ~/.bash_profile, ~/.profile 또는 ~/.bashrc에 추가합니다. 일반적으로 모두에 추가해 두는 것이 좋습니다.

sh
코드 복사
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
pyenv-virtualenv 설정:
pyenv-virtualenv를 사용하려면, 추가 설정을 ~/.bashrc에 추가해야 합니다.

sh
코드 복사
eval "$(pyenv virtualenv-init -)"
설정 적용:
쉘을 다시 시작하거나, 아래 명령어를 사용해 설정을 적용합니다.

sh
코드 복사
source ~/.bash_profile
source ~/.profile
source ~/.bashrc

pyenv install 3.11.9

pyenv global 3.11.9

python --version