
### 8.4.2. 상태 정보의 엔드포인트 

```
이 섹션은 Neo4j 인과관계 클러스터링 상태를 모니터링하기 위한 HTTP 엔드포인트를 나타냅니다.  
```

인과관계 클러스터는 클러스터의 상태 모니터링에 사용할 수 있는 HTTP 엔드포인트를 나타냅니다. 이 섹션에서는 이 엔드포인트를 나타내고 그 의미를 설명합니다. 

이 섹션에서는 다음을 포함합니다:

- 인과관계 클러스터링 엔드포인트 보안 적용 
- 엔드포인트 통합
- 코어 서버의 엔드포인트 
- 읽기 복제본의 엔드포인트 

#### 8.4.2.1. 인과관계 클러스터링 엔드포인트의 보안 설정 적용 

Neo4j에서 인증 및 권한 부여가 활성화되어 있다면 ([섹션 7.1.3, 인증 및 권한 부여](/security/authentication-authorization/enable.md)), 인과관계 클러스터링 엔드포인트는 인증 자격도 요구합니다. 일부 밸런서 로드 및 프록시 서버의 경우 이 옵션을 제공하는 것은 선택 사항이 아닙니다. 이 상황에서, [*neo4j.conf*](/configuration/file-locations.md)에서 ```dbms.security.causal_clustering_status_auth_enabled=false```을 설정하여 인과관계 클러스터링 엔드포인트 상태를 비활성화 해야합니다. 

#### 8.4.2.2. 통합된 엔드포인트

통합된 엔드포인트는 아래와 같이 코어 서버 및 읽기 복제본에 존재합니다. 

- ```/db/manage/server/causalclustering/writable``` — 특정 인스턴스에 ```write``` 트래픽을 지시할 때 사용합니다.

- ```/db/manage/server/causalclustering/read-only``` — 특정 인스턴스에 ```read``` 트래픽을 지시할 때 사용합니다.

- ```/db/manage/server/causalclustering/available``` — 임의의 요청 유형을 읽기 트랜잭션 처리에 사용할 수 있는 인스턴스로 전달할 때 사용합니다. 

**표 8.14. 통합된 HTTP 엔드포인트 응답**

| 엔드포인트                                     | 인스턴스 상태 | 반환된 코드     | 본문    |
| ---------------------------------------------- | ------------- | --------------- | ------- |
| `/db/manage/server/causalclustering/writable`  | Leader        | `200 OK`        | `true`  |
|                                                | Follower      | `404 Not Found` | `false` |
|                                                | Read Replica  | `404 Not Found` | `false` |
| `/db/manage/server/causalclustering/read-only` | Leader        | `404 Not Found` | `false` |
|                                                | Follower      | `true`          | `true`  |
|                                                | Read Replica  | `true`          | `true`  |
| `/db/manage/server/causalclustering/available` | Leader        | `200 OK`        | `true`  |
|                                                | Follower      | ```200 OK```    | `true`  |
|                                                | Read Replica  | ```200 OK```    | `true`  |


#### 8.4.2.3. 핵심 서버의 엔드포인트

상태 모니터링을 위해서 코어 서버는 다음 엔드포인트를 제공합니다. 

- ```/db/manage/server/core/writable``` — 특정 인스턴스에 ```write``` 트래픽을 지시할 때 사용합니다.

- ```/db/manage/server/core/read-only``` — 특정 인스턴스에 ```read``` 트래픽을 지시할 때 사용합니다.

- ```/db/manage/server/core/available``` — 임의의 요청 유형을 읽기 트랜잭션 처리에 사용할 수있는 인스턴스로 전달할 때 사용합니다.

**표 8.15. 핵심 HTTP 엔드포인트 응답**

| 엔드포인트                         | 인스턴스 상태 | 반환된 코드     | 본문    |
| ---------------------------------- | ------------- | --------------- | ------- |
| `/db/manage/server/core/writable`  | Leader        | `200 OK`        | `true`  |
|                                    | Follower      | `404 Not Found` | `false` |
| `/db/manage/server/core/read-only` | Leader        | `404 Not Found` | `false` |
|                                    | Follower      | `200 OK`        | `true`  |
| `/db/manage/server/core/available` | Leader        | `200 OK`        | `true`  |
|                                    | Follower      | ```200 OK```    | `true`  |


**예 8.13. 엔드포인트 모니터링에 인과관계 클러스터링 사용**

명령어 줄에서, 이 엔드포인트를 요청하는 방법은 ```curl```을 사용하는 것 입니다. 변수가 없으면, ```curl```은 주어진 URI에서 HTTP ```GET```을 수행하며, 존재한다면 분문을 출력합니다. 응답 코드를 얻고 싶다면 자세한 출력을 위해`-v` 플래그를 추가하십시오. 예시: 

- 상세 출력은 현재 리더로 선택된 코어 서버에서 ```writable``` 엔드포인트 요청:

```
#> curl -v localhost:7474/db/manage/server/core/writable
* About to connect() to localhost port 7474 (#0)
*   Trying ::1...
* connected
* Connected to localhost (::1) port 7474 (#0)
> GET /db/manage/server/core/writable HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libc url/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
> Host: localhost:7474
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Access-Control-Allow-Origin: *
< Transfer-Encoding: chunked
< Server: Jetty(6.1.25)
<
* Connection #0 to host localhost left intact
true* Closing connection #0
```

#### 8.4.2.4. 읽기 복제본 엔드포인트 

읽기 복제본은 상태 모니터링의 다음 엔드포인트를 제공합니다.

- ```/db/manage/server/read-replica/available``` - 임의의 요청 유형을 읽기 트랜잭션 처리에 사용하는 인스턴스로 전달할 때 사용합니다.

**표 8.16. HTTP 엔드포인트 읽기 복제본 응답**

| 엔드포인트                                 | 반환 코드 | 본문   |
| ------------------------------------------ | --------- | ------ |
| `/db/manage/server/read-replica/available` | `200 OK`  | `true` |