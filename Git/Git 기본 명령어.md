### 개요
Git에서 자주 사용하는 기초 명령어를 모은 문서이다.
### Git 프로젝트 설정
```
# 현재 연결된 원격 저장소 출력
git remote -v

# 새로운 원격 저장소(origin) 추가
git remote add origin <추가할 원격 저장소 주소, https 혹은 ssh>

# 기존에 'origin' 이름의 원격 저장소가 존재할 때
git remote add <새로 연결할 원격 저장소 별명> <추가할 원격 저장소 주소>
```
### Git 사용자 설정
```
# 사용자 이름 설정
git config --global user.name "cjchoi"

# 사용자 이메일 설정
git config --global user.email "cjchoi@triniti.kr"
```
### Git 브랜치 변경
```
# 새로운 브랜치 생성 후 이동
git checkout -b <새 브랜치 이름>

# 기존 브랜치로 이동
git checkout <기존 브랜치 이름>

# 브랜치 이름 변경
git branch -m <기존 브랜치> <새 브랜치>
```
### Git 파일 push
```
# git 원격 저장소에 올릴 파일 추가
git add *.<파일 형식>
git add .

# add한 파일 확정(커밋)
git commit -m "커밋 이름, 메모"

# 원격 저장소와 동기화
git push -u <원격 저장소 별명> <브랜치 이름> # <브랜치 이름>과 자동으로 연결, 이후에 'git push'만 진행해도 자동 연결

git push <원격 저장소 별명> <브랜치 이름>

# 강제로 푸시 (주의 요함)
git push --force <원격 저장소 별명> <브랜치 이름>

# 모든 브랜치 푸시
git push --all <원격 저장소 별명>

# 태그 푸시
git push --tags
```
### Git 프로젝트 복사
```
# 대표 브랜치 내려받기
git clone <프로젝트 주소>
```

### 기타 유용한 명령어
```
# 원격 저장소의 변경 사항을 가져오고 로컬 브랜치와 병합
git pull <원격 저장소 별명> <브랜치 이름>

# 상태 확인 (추적되지 않은 파일, 변경된 파일 등)
git status

# 로그 확인 (커밋 히스토리)
git log

# 특정 파일의 변경 내용 확인
git diff <파일 이름>
```