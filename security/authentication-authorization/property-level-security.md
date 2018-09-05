
### 7.1.7. 속성-레벨 보안 

```
이 섹션에서는 Neo4j에서 보안 속성-레벨을 설정하는 방법에 대해서 다룹니다. 
```

역할 기반 데이터베이스 전체 속성 블랙리스트를 사용하여 사용자가 읽을 수 있는 속성을 제한할 수 있습니다.
아래 표에는 이 기능을 통제하는 매개변수 구성이 나열됩니다.

| 변수 명                                  | 기본 값 | 설명                                                         |
| ---------------------------------------- | ------- | ------------------------------------------------------------ |
| `dbms.security.property_level.enabled`   | `false` | 보안 속성 레벨을 활성화 합니다.                              |
| `dbms.security.property_level.blacklist` |         | 역할에 대한 속성 수준 접근 권한 매핑입니다.    지도는 키 - 값 쌍의 세미콜론으로 구분된 목록으로 형식을 지정합니다. 키는 역할 명이고 값은 쉼표로 구분 된 블랙리스트 속성입니다.  주어진 사용자의 블랙리스트에 등록 된 속성은 사용자가 속한 모든 역할 집합입니다 |

**고려사항**
 
속성 블랙리스트는 사용자가 속성을 읽을 수 없도록합니다. 블랙리스트에 등록된 속성을 쿼리하는 사용자는 노드/관계에 속성이 없는 것과 같은 결과를 가집니다. 

속성을 블랙리스트에 올리면 그 속성을 쓰는 것이 아닌 읽을 때만 영향을 줍니다. 따라서 ```reader``` 역할이 할당된 사용자만 블랙리스트 역할에 추가할 것을 권장합니다. 

**제한사항**

블랙리스트 내에 있는 것과 일치하는 이름의 모든 속성만 영향을 받습니다. 이것은 노드 및 관계 연관 여부, 노드 라벨 및 관계 유형과도 관계없이 적용됩니다. 
 
#### 7.1.7.1. 블랙리스트 속성 실행 

이 속성을 활성화하려면, 다음 단계를 따르십시오. 

1. ```dbms.security.property_level.enabled```에서 보안 속성-레벨을 활성화합니다. 

2. ```dbms.security.property_level.blacklist```을 설정하여 특정 역할 속성을 읽지 못하도록 제한합니다.
 
3. 제한된 속성 접근을 가진 한 개 또는 여러 개의 사용자 지정 역할을 만듭니다.  

4. 적합한 역할에 사용자를 추가합니다. 

**예시 7.23. 블랙리스트 속성 실행하기**

먼저, 보안 속성-레벨을 활성화하고 블랙리스트를 생성합니다:
```
dbms.security.property_level.enabled=true
dbms.security.property_level.blacklist=\
  roleX=propertyA;\
  roleY=propertyB,propertyC
```

그 후, 고객 역할을 생성하여 사용자를 할당합니다. 
```
CALL dbms.security.createRole('roleX')
CALL dbms.security.createRole('roleY')
CALL dbms.security.addRoleToUser('roleX', 'user-1')
CALL dbms.security.addRoleToUser('roleY', 'user-2')
CALL dbms.security.addRoleToUser('roleX', 'user-3')
CALL dbms.security.addRoleToUser('roleY', 'user-3')
```

이렇게하면 다음 결과를 가집니다. 

- 사용자 ```user-1```는 속성 ```porpertyA```를 읽을 수 없습니다. 

- 사용자 ```user-2```는 속성 ```propertyB```와 ```propertyC```를 읽을 수 없습니다. 

- 사용자 ```user-3```는 속성 ```porpertyA```, ```propertyB``` 및 ```propertyC```를 읽을 수 없습니다.