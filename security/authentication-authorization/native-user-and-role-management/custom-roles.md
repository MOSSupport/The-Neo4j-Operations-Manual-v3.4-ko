#### 7.1.4.2. 사용자 역할 

```
이 절에서는 Neo4j의 사용자 역할에 대해서 설명합니다. 
```


사용자 지정 역할은 [관리자](/security/authentication-authorization/terminology.md)가 생성하고 삭제할 수 있습니다. 사용자 지정 역할은 특정 사용자의 개발된 프로시저 실행을 제어하기위해서 만들어집니다. 반면, 사용자 정의 역할은 [*neo4j.conf*](/configuration/file-locations.md)에서 명시적으로 허용된 절차를 실행하는 것 외에는 권한이 없습니다. 

하위 그래프 액세스 제어를 허용하기 위해 맞춤 역할을 사용하는 방법은 [7.1.6절, "하위 그래프 액세스 제어"](/security/authentication-authorization/subgraph-access-control.md)을 참조하면 됩니다. 