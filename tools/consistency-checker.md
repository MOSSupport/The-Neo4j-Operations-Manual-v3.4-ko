
## 10.2. 일관성 검사기 

```
이 절에서는 Neo4j 일관성 검사기에 대해 다룹니다. 
```

```neo4j-admin```을 이용하여 데이터베이스 및 백업에 대한 일관성을 검사합니다.

### 10.2.1. 데이터베이스 및 백업에 대한 일관성 검사 

```neo4j-admin``` 도구는 ```bin``` 디렉터리에 위치합니다. 데이터베이스의 일관성을 체크하기 위해서는 ```check-consistency``` 매개변수를 사용합니다.

**문법**

```
neo4j-admin check-consistency [--database=<name>] [--backup=</path/to/backup>] [--verbose[=<true|false>]] [--report-dir=<directory>] [--additional-config=<config-file-path>] [--check-graph[=<true|false>]] [--check-indexes[=<true|false>]] [--check-label-scan-store[=<true|false>]] [--check-property-owners[=<true|false>]]
```

**옵션**

| 옵션                     | 디폴트      | 설명                                                         |
| ------------------------ | ----------- | ------------------------------------------------------------ |
| --database               | graph.db    | 데이터베이스 명                                              |
| --backup                 |             | 일관성을 검사하기위한 백업 경로. --database와 함께 사용 불가. |
| --additional-config      |             | 추가 구성을 제공하는 구성 파일. 이 변수는 더 이상 사용되지 않습니다. |
| --verbose                | 거짓(false) | 자세한 출력 여부를 지정.                                     |
| --report-dir             | .           | 보고서 파일을 출력할 디렉터리.                               |
| --check-graph            | 참(true)    | 노드, 관계, 속성, 유형, 토큰 간의 체크 수행.                 |
| --check-indexes          | 참(true)    | 인덱스의 체크 수행.                                          |
| --check-label-scan-store | 참(true)    | 레이블(label) 스캔 저장소 체크 수행.                         |
| --check-property-owners  | 거짓(false) | 속성 소유권에 관한 추가 체크 수행. 이 체크는 시간-메모리가 매우 소비됩니다. |


**제한 사항**

현재 사용중인 데이터베이스는 일관성을 체크할 수 없습니다. 만약 실행중인 데이터베이스 검사를 수행하면 실행은 안되고 오류를 보여줍니다.

**출력**

일관성 검사기가 오류를 찾지 못하면 오류없이 종료하며 보고서를 생성하지 않습니다. 일관성 검사 프로그램에서 오류를 발견하면 종료 코드 ```1```로 종료하고 ```inconsistencies-YYYY-MM-DD.HH24.MI.SS.report``` 형식의 이름을 가진 보고서 파일을 작성합니다. 보고서 파일의 위치는 현재 작업 디렉토리이거나 ```report-dir``` 옵션으로 지정됩니다.

**예제 10.8. 일관성 검사기 실행**

데이터베이스 일관성 체크에는 ```--database``` 옵션울 사용합니다. 먼저 데이터베이스의 실행을 중지한 뒤 실행할 것을 기억하십시오. 

```
$neo4j-home> bin/neo4j stop
$neo4j-home> bin/neo4j-admin check-consistency --database=graph.db

2017-02-02 09:45:31.719+0000 INFO  [o.n.k.i.s.f.RecordFormatSelector] Selected RecordFormat:StandardV3_0[v0.A.7] record format from store /Users/maria/neo4j-enterprise-3.2.0-alpha03/data/databases/graph.db
2017-02-02 09:45:31.719+0000 INFO  [o.n.k.i.s.f.RecordFormatSelector] Format not configured. Selected format from the store: RecordFormat:StandardV3_0[v0.A.7]
2017-02-02 09:45:31.963+0000 INFO  [o.n.m.MetricsExtension] Initiating metrics...
....................  10%
....................  20%
....................  30%
....................  40%
....................  50%
....................  60%
....................  70%
....................  80%
....................  90%
...................Checking node and relationship counts
....................  10%
....................  20%
....................  30%
....................  40%
....................  50%
....................  60%
....................  70%
....................  80%
....................  90%
.................... 100%
```

백업 일관성 체크는 ```--backup``` 옵션을 사용합니다.

```
bin/neo4j-admin check-consistency --backup backup/graph.db-backup
```