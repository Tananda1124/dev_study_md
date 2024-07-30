이 글은 Ubuntu 환경에서 PostgreSQL과 pgvector를 설치하는 방법에 대한 것이다.
#### WSL로 Ubuntu를 실행한 이후, 다음 코드로 설치한다.
```
sudo apt update && sudo apt upgrade sudo apt install postgresql sudo apt install postgresql-server-dev-14 # 확장 설치에 필요 
sudo apt install clang
```
#### pgvector 설치
1. 임시 디렉토리로 이동한다.
2. pgvector를 클론한다.
3. 소스를 빌드하고 설치한다.
```
cd /tmp git clone --branch v0.7.0 https://github.com/pgvector/pgvector.git cd pgvector make sudo make install
```
#### 테스트 코드
PostgreSQL을 실행하고, 다음 코드로 테스트한다.
```
CREATE EXTENSION vector;

CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(3));
INSERT INTO items (embedding) VALUES ('[1, 2, 3]'), ('[4, 5, 6]'); SELECT * FROM items ORDER BY embedding <-> '[3, 1, 2]' LIMIT 5;
```

 `id | embedding  ----+-----------   1 | [1, 2, 3]   2 | [4, 5, 6]`

유사한 값부터 정상적으로 출력되었다면, 성공적으로 설치된 것이다.
#### 유사도 측정 알고리즘 설정
```
<+> : L2 Distance
<=> : Cosine Similarity
<#> : Dot Product
<-> : L1 Distance
```
각각 상황에 맞는 알고리즘을 골라 사용하도록 하자.