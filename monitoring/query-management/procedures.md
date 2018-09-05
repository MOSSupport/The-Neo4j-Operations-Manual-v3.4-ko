
## 8.3.2. 쿼리 관리 절차

```
이 섹션에서는 Neo4j에서 현재 쿼리에 맞춰 실행되는 절차에 대해 알아봅니다. 
```

- 전문 용어 
- 쿼리 리스트
- 잠금 리스트 
- 다중 쿼리 종료 
- 단일 쿼리 종료 
- 트랜잭션 리스트 

명시되지 않는 한, 이 섹션의 절차 내에 있는 모든 변수는 제공되어야 합니다. 

### 8.3.2.1. 전문 용어

**관리자**
관리자는 ```admin``` 역할을 맡은 [사용자](/monitoring/query-management.md)입니다. 유저와 그 역할에 관련된 내용은 [섹션 7.1.4,"네이티브 사용자와 역할 관리"](../../security/authentication-authorization/native-user-and-role-management.md)를 참조하십시오.

**현재 사용자**
[사용자](/monitoring/query-management/procedures.md)는 이 섹션에서 설명된 명령어를 호출하고 현재 로그인한 유저입니다. 

**유저**

+ 유저는 유저 이름과 자격으로 구성되고, 자격은 비밀번호와 같은 정보 단위로 구성되어 개인 신원을 확인할 수 있습니다. 
+ 유저는 인간, 어플리케이션 등을 나타냅니다.


### 8.3.2.2. 실행되는 모든 쿼리 나열

관리자는 현재 인스턴스 내에 실행되는 모든 쿼리를 확인할 수 있습니다. 대신에 현재 유저는 자신의 실행되는 모든 쿼리를 확인할 수 있습니다. 

**문법:**
```CALL dbms.listQueries()```

**반환 값:**

| 이름                       | 타입    | 설명                                                         |
| -------------------------- | ------- | ------------------------------------------------------------ |
| ```queryId```             | String  | 쿼리의 ID입니다.                                             |
| ```username```            | String  | 쿼리를 실행하는 유저의 이름입니다.                           |
| ```metaData```             | Map     | 트랜잭션과 관련된 모든 메타데이터 입니다.                    |
| ```query```                | String  | 쿼리 그 자체입니다.                                          |
| ```parameters```           | Map     | 쿼리에서 사용되는 모든 변수를 포함하는 맵(map) 입니다.       |
| ```planner```             | String  | 쿼리에서 사용되는 플래너 입니다.                             |
| ```runtime```             | String  | 쿼리에서 사용되는 런타임입니다.                              |
| ```indexes```             | List    | 쿼리에서 사용되는 인덱스입니다.                              |
| ```startTime```           | String  | 쿼리가 시작된 시간입니다.                                    |
| ```elapsedTime```         | String  | ```elapsedTimeMillis``` 을 대신 사용합니다. 쿼리가 시작된 이후 경과된 시간입니다. |
| ```connectionDetails``` | String  | ```requestScheme```,   ```clientAddress```, ```requestUri``` 을 대신 사용합니다. 쿼리와 관련된 세부 연결 정보입니다. |
| ````protocol````           | String  | 쿼리를 실행하는 연결에서 사용되는 프로토콜입니다.            |
| ```clientAddress```       | String  | 쿼리를 실행하는 연결에서 사용되는 클라이언트 주소입니다.     |
| ````requestUri````         | String  | 쿼리를 실행하는 클라이언트 연결에서 사용되는 URI 요청입니다. |
| ```status```              | String  | 실행중인 쿼리의 상태입니다.                                  |
| ```resourceInformation``` | Map     | 실행중인 쿼리의 상태입니다.                                  |
| ```activeLockCount```     | Integer | 쿼리를 실행하는 트랙잭션이 보유한 활성화된 잠금 개수입니다.  |
| ```elapsedTimeMillis```   | Integer | 쿼리가 시작된 후 경과된 초단위 시간입니다.                   |
| ```cpuTimeMillis```       | Integer | 쿼리를 실행하는데 경과된 CPU 초단위 시간입니다.              |
| ```waitTimeMillis```      | Integer | 잠금하기까지 흐른 초단위의 대기 시간                         |
| ```idleTimeMillis```      | Integer | 초단위의 유휴시간                                            |
| ```allocatedBytes```      | Integer | 실행중인 쿼리에 할당 된 바이트 수는 쿼리 기간 동안 누적됩니다. 즉 메모리가 많거나 오래 실행되는 쿼리의 경우 값은 현재 메모리 할당보다 클 수 있습니다. |
| ```pageHits```            | Integer | 실행하는 동안 발생한 페이지 히트                             |
| ```pageFaults```          | Integer | 실행하는동안 발생한 페이지 오류                              |


**예 8.5. 현재 실행되는 쿼리 보기**

아래 예는 유저 'alwood'가 현재 특정 변수를 생성하는 ```dbms.listQueries()```, 즉 ```queryId```, ```username```,```query```,```elapsedTimeMillis```,```requestUri```,그리고 ```status```를 실행하고 있음을 보여줍니다. 

```
CALL dbms.listQueries() YIELD queryId, username, query, elapsedTimeMillis, requestUri, status
```

```
╒══════════╤═══════════╤══════════════════════════════╤════════════════════╤════════════════╤═══════════════════╕
│"queryId" │"username" │"query"                       │"elapsedTimeMillis" │"requestUri"    │"status"           │
╞══════════╪═══════════╪══════════════════════════════╪════════════════════╪════════════════╪═══════════════════╡
│"query-33"│"alwood"   │"CALL dbms.listQueries() YIELD│"1"                 │"127.0.0.1:7687"│{"state":"RUNNING"}│
│          │           │ queryId, username, query, ela│                    │                │                   │
│          │           │psedTime, requestUri, status" │                    │                │                   │
└──────────┴───────────┴──────────────────────────────┴────────────────────┴────────────────┴───────────────────┘
1 row
```


### 8.3.2.3. 쿼리에 대한 모든 활성 잠금 나열 

관리자는 트랙잭션 실행 쿼리의 ```queryId```을 사용해서 모든 활성 잠금을 확인할 수 있습니다. 

**문법**

```CALL dbms.listActiveLocks(queryId)```

**반환 값**

| 이름                | 타입    | 설명                          |
| ------------------- | ------- | ----------------------------- |
| ```mode```          | String  | 트랜잭션에 해당하는 잠금 모드 |
| ```resourceType``` | String  | 잠금 리소스의 종류            |
| ```resourceId```   | Integer | 잠금 리소스의 아이디          |


**예 8.6. 쿼리에 대한 활성 잠금 확인**

아래 예는 쿼리를 아이디 ```query-614```을 사용하여 트랜잭션을 실행하는 쿼리가 가진 활성 잠금을 보여줍니다.

```
CALL dbms.listActiveLocks( "query-614" )
```

```
╒════════╤══════════════╤════════════╕
│"mode"  │"resourceType"│"resourceId"│
╞════════╪══════════════╪════════════╡
│"SHARED"│"SCHEMA"      │"0"         │
└────────┴──────────────┴────────────┘
1 row
```

아래 예는 ```dbms.listQueries```절차에서 ```queryId```생성하여 현재 실행 되는 모든 쿼리에 대한 활성 잠금을 보여줍니다. 

```
CALL dbms.listQueries() YIELD queryId, query
CALL dbms.listActiveLocks( queryId ) YIELD resourceType, resourceId, mode
RETURN queryId, query, resourceType, resourceId, mode
```

```
╒═══════════╤══════════════════════════════╤══════════════╤════════════╤════════╕
│"queryId"  │"query"                       │"resourceType"│"resourceId"│"mode"  │
╞═══════════╪══════════════════════════════╪══════════════╪════════════╪════════╡
│"query-614"│"match (n), (m), (o), (p), (q)│"SCHEMA"      │"0"         │"SHARED"│
│           │ return count(*)"             │              │            │        │
├───────────┼──────────────────────────────┼──────────────┼────────────┼────────┤
│"query-684"│"CALL dbms.listQueries() YIELD│"SCHEMA"      │"0"         │"SHARED"│
│           │ .."                          │              │            │        │
└───────────┴──────────────────────────────┴──────────────┴────────────┴────────┘
2 rows
```


### 8.3.2.4. 여러 쿼리 종료

관리자는 주어진 모든 쿼리 ID중 하나를 사용하여 인스턴스의 모든 트랙잭션에서 실행되는 쿼리를 종료할 수 있습니다. 대안으로, 현재 유저는 주어진 쿼리 ID중 임의의 쿼리 ID를 갖는 유저의 모든 트랜잭션을 종료시킬 수 있습니다. 

**문법:**

```CALL dbms.killQueries(ids)```

**변수:**

| 이름      | 종류         | 설명                            |
| --------- | ------------ | ------------------------------- |
| ```ids``` | List<String> | 종료되는 모든 쿼리 ID 리스트 |


**반환 값:**

| 이름            | 종류   | 설명                                                |
| --------------- | ------ | --------------------------------------------------- |
| ```queryId```  | String | 종료된 쿼리의 ID입니다.                             |
| ```username``` | String | 쿼리(현재는 종료된)를 실행하는 사용자의 이름입니다. |


**예 8.7. 여러 쿼리 종료**

아래의 예는 유저가 'query-378' 와 'query-765' 아이디로 종료하고, 각각 'joesmith'와 'annebrown'로 로그인한 사용자를 나타냅니다. 

```
CALL dbms.killQueries(['query-378','query-765'])
```

```
+---------------------------+
| queryId     | username    |
+---------------------------+
| "query-378" | "joesmith"  |
| "query-765" | "annebrown" |
+---------------------------+
2 rows
```


### 8.3.2.5. 단일 쿼리 종료

관리자는 ID가 주어진 실행중인 모든 트랜잭션을 인스턴스에서 종료할 수 있습니다. 대안으로 현재 유저는 자신의 주어진 ID로 쿼리를 실행하여 트랜잭션을 종료할 수 있습니다. 

**문법:**

```CALL dbms.killQuery(id)```

**변수:**

| 이름     | 타입   | 설명                        |
| -------- | ------ | --------------------------- |
| ```id``` | String | 쿼리를 종료시키는 ID입니다. |


**반환 값:**

| 이름            | 종류   | 설명                                                |
| --------------- | ------ | --------------------------------------------------- |
| ```queryId```  | String | 종료된 쿼리의 ID입니다.                             |
| ```username``` | String | 쿼리(현재는 종료된)를 실행하던 사용자의 이름입니다. |
| ```message```  | String | 쿼리가  검색 성공 여부를 나타내는 메시지입니다.     |


**예 8.8. 단일 쿼리 종료**

아래는 사용자 'joesmith' 가 그의 쿼리를 아이디 'query-502'로 종료한 예 입니다.  

```
CALL dbms.killQuery('query-502')
```

```
+-----------------------------------------+
| queryId     | username    | message     |
+-----------------------------------------+
| "query-502" | "joesmith"  | Query found |
+-----------------------------------------+
1 row
```

아래는 존재하지 않는 아이디로 쿼리를 kill할 때 결과를 보여주는 예 입니다. 

```
CALL dbms.killQuery('query-502')
```

```
+---------------------------------------------------------+
| queryId     | username    | message                     |
+---------------------------------------------------------+
| "query-502" | "n/a"       | No Query found with this id |
+---------------------------------------------------------+
1 row
```

### 8.3.2.6. 실행되는 모든 트랜잭션 리스트 

[관리자](/monitoring/query-management/procedures.md)는 인스턴스를 포함하여 현재 실행되는 모든 트랜잭션을 확인할 수 있습니다. 그 대신에, [현재 사용자](/monitoring/query-management/procedures.md)는 그들의 현재-실행되는 모든 트랜잭션을 확인할 수 있습니다. 

**구문:**

```
CALL dbms.listTransactions()
```

**반환 값:**

| 이름                   | 종류    | 설명                                                         |
| ---------------------- | ------- | ------------------------------------------------------------ |
| `transactionId`        | String  | 트랜잭션의 ID 입니다.                                        |
| `username`             | String  | 현재 트랜잭션을 실행하는 사용자의 이름입니다.                |
| `metaData`             | Map     | 트랜잭션에 할당된 모든 메타데이터 입니다.                    |
| `startTime`            | String  | 트랜잭션이 시작된 시간입니다.                                |
| `protocol`             | String  | 트랜잭션을 발행하는 커넥션에 사용되는 프로토콜입니다.        |
| `clientAddress`        | String  | 트랜잭션을 발행하는 커넥션에 사용되는 클라이언트 주소입니다. |
| `requestUri`           | String  | 클라이언트에의해 트랜잭션을 발행하는 커넥션에 사용되는 URI 요청입니다. |
| `currentQueryId`       | String  | 트랜잭션에서 실행되는 현재 쿼리의 ID입니다.                  |
| `currentQuery`         | String  | 트랜잭션에서 실행되는 현재 쿼리입니다.                       |
| `activeLockCount`      | Integer | 트랜잭션에서 보유하는 활성화된 잠금 개수입니다.              |
| `status`               | String  | 실행되는 트랜잭션의 상태입니다.                              |
| `resourceInformation`  | Map     | 트랜잭션이 차단 될 때까지 기다리는 트랜잭션 정보입니다.      |
| `elapsedTimeMillis`    | Integer | 이것은 트랜잭션이 시작된 후 경과 된 시간 (밀리 초)입니다.    |
| `cpuTimeMillis`        | Integer | 트랜잭션을 실행하는 데 실제로 소비 된 CPU 시간 (밀리 초)입니다. |
| `waitTimeMillis`       | Integer | 잠금 설정에 소비된 대기 시간(밀리 초) 입니다.                |
| `idleTimeMillis`       | Integer | 유휴 시간(밀리 초)                                           |
| `allocatedBytes`       | Integer | 실행중인 트랜잭션에 할당 된 바이트. 이 번호는 거래 기간 동안 누적됩니다. |
| `allocatedDirectBytes` | Integer | 실행 중인 트랜잭션에서 사용되는 바이트 입니다.               |
| `pageHits`             | Integer | 실행 중에 발생한 페이지 히트입니다.                          |
| `pageFaults`           | Integer | 실행 중에 발생한 페이지 오류 입니다.                         |

**예 8.9. 현재 실행되는 트랜잭션 확인**

아래 예는 사용자 `alwood`가 현재 ```dbms.listTransactions()```를 사용하는 것을 보여줍니다. 프로시저를 호출하면 실행되는 트랜잭션과 관련된 특정 정보를 보여줍니다. 이는 ```transactionId```, ```username```, ```currentQuery```, ```elapsedTimeMillis```, ```requestUri```, 및 ```status``` 와 같습니다. 

```
CALL dbms.listTransactions() YIELD transactionId, username, currentQuery, elapsedTimeMillis, requestUri, status
```

```
╒════════════════╤═══════════╤══════════════════════════════════════╤════════════════════╤════════════════╤═══════════════════╕
│"transactionId" │"username" │"currentQuery"                        │"elapsedTimeMillis" │"requestUri"    │"status"           │
╞════════════════╪═══════════╪══════════════════════════════════════╪════════════════════╪════════════════╪═══════════════════╡
│"transaction-22"│"alwood"   │"CALL dbms.listTransactions() YIELD   │"1"                 │"127.0.0.1:7687"│"Running"          │
│                │           │ transactionId, username, currentQuery│                    │                │                   │
│                │           │elapsedTime, requestUri, status"      │                    │                │                   │
└────────────────┴───────────┴──────────────────────────────────────┴────────────────────┴────────────────┴───────────────────┘
1 row
```
