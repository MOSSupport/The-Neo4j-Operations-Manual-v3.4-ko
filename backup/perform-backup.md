
## 6.2. 백업 실행 

```
이 섹션에서는 Neo4j 데이터베이스를 백업하는 방법에 대해 알아봅니다.
```

Neo4j는 전체 및 증가하는 백업을 모두 지원합니다. 

백업 절차는 독립 데이터베이스, 고-가용 클러스터 및 Casual 클러스터 모두 동일합니다. 

백업은 Neo4j 서버를 실행할 때 데이터베이스 서버 또는 네트워크에서 수행됩니다.결과 파일은 로컬로 작성되거나 백업을 수행 서버에 있는 네트워크 마운트 디렉토리에서  작성됩니다.

### 6.2.1. 설정 옵션

아래 표는 백업과 관련된 변수 설정을 나타냅니다:

| 변수명                   | 기본 값                                                      | 설명                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `dbms.backup.enabled`    | `true`                                                       | 온라인 실행 백업을 지원합니다.                               |
| `dbms.backup.address`    | `127.0.0.1:6362-6372` : 고가용 클러스터 멤버 및 독단 서버 `127.0.0.1:6362` : Casual 클러스터 멤버 | 온라인 백업을위한 리스닝 서버.                               |
| `dbms.backup.ssl_policy` |                                                              | 백업 포트에서 사용되는 SSL 정책. 이것은 Casual 클러스터 멤버에서만 사용할 수 있습니다. |

### 6.2.2. 백업 명령어 

Neo4j 데이터베이스는 ````neo4j-admin````의 ```backup```명령어를 이용해서 온라인 모드에서 사용할 수 있습니다. 

**구문**

```
neo4j-admin backup --backup-dir=<backup-path> --name=<graph.db-backup>
                    [--from=<address>] [--protocol=<any|catchup|common>]
                    [--fallback-to-full[=<true|false>]]
                    [--pagecache=<pagecache>]
                    [--timeout=<timeout>]
                    [--check-consistency[=<true|false>]]
                    [--additional-config=<config-file-path>]
                    [--cc-graph[=<true|false>]]
                    [--cc-indexes[=<true|false>]]
                    [--cc-label-scan-store[=<true|false>]]
                    [--cc-property-owners[=<true|false>]]
                    [--cc-report-dir=<directory>]
```


**옵션**

| 옵션                  | 기본 값        | 설명                                                         |
| --------------------- | -------------- | ------------------------------------------------------------ |
| --backup-dir          |                | 백업을 저장할 디렉토리 입니다.                               |
| --name                |                | 백업 명. 이미 있으면 증가하는 백업이 실행됩니다.             |
| --from                | localhost:6362 | Neo4j의 호스트 및 포트                                       |
| --protocol            | any            | 백업을 수행할 프로토콜 입니다. 실행할 때 `any`, `catchup` 를 먼저 수행합니다. 실패한다면, `common`으로 돌아갑니다. 이것을 설정할 필요는 없지만, 백업 시작이 오래 걸리거나 백업이 실패하기를 원하면 인과 관계 클러스터 백업에 캐치 업하도록 설정하거나 HA / 단일 인스턴스 백업에 공통으로 설정할 수 있습니다. |
| --fallback-to-full    | true           | 만약 증가하는 백업이 실패한다면, 백업은 예전 백업을 <name>.err.<N>에 옮겨 전체 백업으로 대체할 것 입니다. |
| --pagecache           | 8M             | 백업 진행에 사용할 페이지 캐쉬 사이즈 입니다.                |
| --timeout             | 20m            | 형식이  <time>[ms\|s\|m\|h]인 타임 아웃 기본 단위는 초 입니다.  이것은 Neo4j 전문 서비스가 지시 한 경우에만 사용해야하는 디버깅 옵션입니다. |
| --check-consistency   | true           | 일관성을 검사할 경우 사용합니다.                             |
| --additional-config   |                | 추가 구성을 제공하는 구성 파일입니다. 이 인수는 더 이상 사용되지 않습니다. |
| --cc-graph            | true           | 노드, 관계, 속성, 형식 및 토큰간 검사를 수행합니다.          |
| --cc-indexes          | true           | 인덱스 체크를 수행합니다.                                    |
| --cc-label-scan-store | true           | 레이블 스캔 저장소에서 검사를 수행합니다.                    |
| --cc-property-owners  | false          | Perform additional checks on property ownership. This check is **very** expensive in time and memory. 소유권에 대한 추가 점검을 수행합니다. 이 검사는 시간과 메모리가 많이 소요됩니다.비 |
| --cc-report-dir       | .              | 일관성 검사가 쓰여지는 디렉터리 입니다.                      |

**예시 6.1. 데이터베이스 백업**

이 예에서는 [메모리 사용량을 통제](/backup/backup-introduction.md)하기 위해 환경 변수를 설정합니다. 

명령 행 옵션 ```--pagecache```를 사용하여 페이지 캐시를 정의합니다.

나아가, ```HEAP_SIZE``` 환경 변수는 백업에 할당된 최대 힙(Heap) 사이즈를 명시할 것 입니다. 

이제 모든 백업을 수행할 수 있습니다:

```
$neo4j-home> export HEAP_SIZE=2G
$neo4j-home> mkdir /mnt/backup
$neo4j-home> bin/neo4j-admin backup --from=192.168.1.34 --backup-dir=/mnt/backup --name=graph.db-backup --pagecache=4G
Doing full backup...
2017-02-01 14:09:09.510+0000 INFO  [o.n.c.s.StoreCopyClient] Copying neostore.nodestore.db.labels
2017-02-01 14:09:09.537+0000 INFO  [o.n.c.s.StoreCopyClient] Copied neostore.nodestore.db.labels 8.00 kB
2017-02-01 14:09:09.538+0000 INFO  [o.n.c.s.StoreCopyClient] Copying neostore.nodestore.db
2017-02-01 14:09:09.540+0000 INFO  [o.n.c.s.StoreCopyClient] Copied neostore.nodestore.db 16.00 kB
...
...
...
```


/mnt/backup의 디렉토리 리스팅을 하면 ```graph-db.backup```라고 불리는 Neo4j 백업을 확인할 수 있습니다. 

### 6.2.3. 증가하는 백업

증가하는 백업은 현존하는 백업 디렉토리가 지정되고트랜잭션 로그가 마지막 백업 이후에 있을 때마다 실행됩니다. 그 후, 백업 명령어는 Neo4j에서 모든 새로운 트랜잭션을 복사하고 백업에 적용시킵니다. 결과는 현재 서버 상태와 동일하게 업데이트된 백업일 것 입니다. 

**예시 6.2. 증가하는 백업 실행**

이 예에서는 앞의 예제에 따라 전체 백업을 수행했다고 가정합니다. 이전과 같이 메모리 사용을 통제합니다. 

증가하는 백업을 실행하려면, 이전 백업 위치를 명시해야 합니다. 

```
$neo4j-home> export HEAP_SIZE=2G
$neo4j-home> bin/neo4j-admin backup --from=192.168.1.34 --backup-dir=/mnt/backup --name=graph.db-backup --fallback-to-full=true --check-consistency=true --pagecache=4G
Destination is not empty, doing incremental backup...
Backup complete.
```

존재하는 데이터가 바른 백업과 ```--fallback-to-full=false```을 포함하지 않는다면 증가하는 백업은 실패할 것 입니다. 또한, 필요한 트랜잭션 로그가 제거되고 ```--fallback-to-full=false```로 설정되어도 실패할 것 입니다. ```--fallback-to-full=true``` 설정은 증가하는 백업이 수행되지 않을 때 수행되는 보호장치 입니다. 

```--check-consistency```는 기본적으로 ```true```로 설정되어 있습니다. 빠르게 증가하는 백업을 하려면 ```--check-consistency=false``` 및 ```--fallback-to-full=false```로 설정하면 됩니다. 
 

미해결 트랜잭션을 복사 할 때 서버는 트랜잭션 로그에 액세스해야합니다.

이 로그는 Neo4j에 의해서 유지되며 일정 기간 후에 ```dbms.tx_log.rotation.retention_policy``` 정책에 따라 자동적으로 제거됩니다. 

백업 전략을 세울 때, 트랜잭션 로그가 증가하는 백업 사이에 있도록 ```dbms.tx_log.rotation.retention_policy```을 설정하는 것이중요합니다.  

### 6.2.4. 코드 종료 

`neo4j-admin backup`는 성공 또는 에러를 기반의 다른 코드로 종료됩니다. 

에러 케이스의 경우, 이것은 어떤 에러를 직면했는지 포함합니다. 

**표 6.1. Neo4j 관리자 백업 종료 코드**

| 코드 | 설명                                                         |
| ---- | ------------------------------------------------------------ |
| `0`  | 성공.                                                        |
| `1`  | 백업 실패.                                                   |
| `2`  | 백업은 성공했지만, 일관성 체크는 실패.                       |
| `3`  | 백업은 성공했지만, 일관성 체크는 Backup succeeded but consistency check found 불일치. |