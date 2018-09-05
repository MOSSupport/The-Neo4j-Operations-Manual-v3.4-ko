
## 3.7. 동적 설정

```
 이 섹션에서는 Neo4j이 작동하는 동안, Neo4j설정 변경하는 법과 변경가능한 설정에 대해 다룹니다.
```

Neo4j 기업용 배포판은 서비스 재시작 없이도 일부 설정을 변경할 수 있습니다. 설정은 ```dbms.setConfigValue()```절차를 사용해서 변경됩니다. ```dbms.setConfigValue()```을 작동하려면 [관리자](../security/authentication-authorization/terminology.md) 권한이 필요합니다. 

운영 시간동안 변경된 설정은 유지되지 않습니다. Neo4j을 재시작할 때, 변경 내용이 누락되지 않으려면 [Neo4j 설정 파일](file-locations.md)을 함께 업데이트 해야 됩니다.


### 3.7.1. 다이나믹 설정 절차

**문법:**

```CALL dbms.setConfigValue(setting, value)```

**반환 값:**

성공 값 반환 안함.

**예외 사항:**

```
알려지지 않았거나 옳바르지 않은 설정 이름
```

```
동적 설정이 아니고 작동 시 변경될 수 없는 것
```

```
올바르지 않은 설정 변수
```

**아래는 쿼리 로깅을 다이나믹하게 설정하는 예 입니다.**

**예 3.5. 환경 변수 설정**

```CALL dbms.setConfigValue('dbms.logs.query.enabled', 'true')```

**옳지 않은 변수가 있다면, 에러 메시지를 띄울 것 입니다.**

**예 3.6. 잘못된 구성값 설정**

```CALL dbms.setConfigValue('dbms.logs.query.enabled', 'yes')```

```
Failed to invoke procedure `dbms.setConfigValue`: Caused by: org.neo4j.graphdb.config.InvalidSettingException: Bad value 'yes' for setting 'dbms.logs.query.enabled': must be 'true' or 'false'
```

**기본 설정은 환경 변수 빈 스트링에 값 변수를 넣으면 됩니다.**

**예 3.7. 구성값 기본 설정**

```
CALL dbms.setConfigValue('dbms.logs.query.enabled', '')
```

### 3.7.2. 다이나믹 설정 참조

**표 3.5. 다이나믹 설정 참조**

| 이름                                                         | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| dbms.checkpoint.iops.limit | IOs의 백그라운드 체크포인트 수를 제한하면, 각 절차가 초마다 실행될 것 입니다. |
| dbms.logs.query.enabled | 설정된 임계값 dbms.logs.query.threshold 보다 오래 걸리는 쿼리를 로깅합니다. |
| dbms.logs.query.rotation.keep_number | 조희 로그의 최대 실행 파일 개수                     |
| dbms.logs.query.rotation.size | 쿼리 로그가 자동으로 회전하는 파일 바이트 사이즈                   |
| dbms.logs.query.threshold | 쿼리 실행이 임계값보다 오래 걸리면 제공된 쿼리 로깅을 사용할 수 있기 때문에 조회가 로깅됩니다. |
| dbms.transaction.timeout | 트랜잭션이 완료되는 최대 시간 간격.                          |

 
