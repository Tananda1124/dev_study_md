
# Net-Tools 필수 명령어

## ifconfig
- Description: 네트워크 인터페이스 설정 및 보기
- Usage: `ifconfig [interface] [options]`
- Common Options:
  - `up`: 인터페이스 활성화
  - `down`: 인터페이스 비활성화
  - `inet addr`: IP 주소 설정
  - `netmask`: 서브넷 마스크 설정

## route
- Description: IP 라우팅 테이블 보기 및 설정
- Usage: `route [add|del] [-net|-host] target [netmask N] [gw G] [metric M]`
- Common Options:
  - `add`: 라우트 추가
  - `del`: 라우트 삭제
  - `-net`: 네트워크 라우트 지정
  - `-host`: 호스트 라우트 지정
  - `gw`: 게이트웨이 설정

## netstat
- Description: 네트워크 연결, 라우팅 테이블, 인터페이스 통계, masquerade 연결, 멀티캐스트 멤버십 보기
- Usage: `netstat [options]`
- Common Options:
  - `-a`: 모든 소켓 표시
  - `-r`: 라우팅 테이블 표시
  - `-i`: 네트워크 인터페이스 통계 표시
  - `-t`: TCP 연결만 표시
  - `-u`: UDP 연결만 표시

## arp
- Description: ARP 캐시 관리 및 보기
- Usage: `arp [-v] [-H type] [-i if] [-a [hostname]] [hostname]`
- Common Options:
  - `-a`: 모든 항목 표시
  - `-d`: 항목 삭제
  - `-s`: 정적 ARP 항목 설정
  - `-i`: 인터페이스 지정

## hostname
- Description: 호스트 이름 보기 및 설정
- Usage: `hostname [name]`
- Common Options:
  - `-i`: IP 주소 표시
  - `-d`: 도메인 이름 표시
  - `-f`: 정규화된 도메인 이름 표시

## mii-tool
- Description: 네트워크 인터페이스의 링크 상태 점검 및 설정
- Usage: `mii-tool [options] [interface]`
- Common Options:
  - `-v`: 자세한 정보 표시
  - `-r`: 링크 다시 설정
  - `-w`: 자동 감지 활성화

## ipmaddr
- Description: 멀티캐스트 주소 관리
- Usage: `ipmaddr [options] [interface]`
- Common Options:
  - `-a`: 멀티캐스트 주소 추가
  - `-d`: 멀티캐스트 주소 삭제
  - `-l`: 멀티캐스트 주소 목록 표시

## nameif
- Description: 네트워크 인터페이스의 이름 변경
- Usage: `nameif [-c configfile] [name] [address]`
- Common Options:
  - `-c`: 설정 파일 지정
