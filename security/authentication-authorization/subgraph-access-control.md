
### 7.1.6. 하위그래프 접근 통제 

```
이 섹션에서는 Neo4j에서 서브그래프 접근 통제를 설정하는 방법에 대해서 다룹니다. 
```
 
그래프의 지정된 부분에 대한 사용자 접근 및 후속 조치를 통제할 수 있습니다. 예를 들어 사용자는 특정 레이블 및 특정 유형의 관계가있는 노드를 읽을 수는 있지만 작성할 수는 없습니다.

하위 그래프 접근을 통제하려면, 다음 단계를 따르면 됩니다. 

1. 제어하려는 그래프의 읽기 및/또기 쓰기를 수행하는 프로시저나 함수를 배치하십시오. 이것은 조직 내부에서 개발되거나 제 3자 라이브러리로 제공될 수 있습니다. 사용자 정의 절차나 함수를 생성할 때, [Neo4j 개발자 메뉴얼 → Neo4j 확장](https://neo4j.com/docs/developer-manual/3.4/extending-neo4j/)를 참조하십시오. 
2. 위에서 설명된 절차를 실행하려면 한개 또는 그 이상의 사용자 역할을 생성해야 합니다. 이러한 역할에 나중에 관련 권한을 할당 할 수 있습니다.
3. 사용자 역할을 가진 유저가 실행할 수 있도록 절차를 설정하십시오. 

아래 단계에서는 절차나 함수가 이미 개발 및 설치되었다고 가정합니다. 

#### 7.1.6.1. 사용자 역할 생성 

네이티브 사용자 관리나 LDAP가 있는 연합 사용자 관리를 통해 사용자 지정 역할을 만들고 관리합니다.

**네이티브 사용자 시나리오**
네이티브 사용자 시나리오에서 사용자 역할은 관련 사용자에게 생성/할당됩니다. 

**예 7.19. 네이티브 사용자 시나리오**

이 예에서는 사용자 ```accounting``` 역할을 생성하고, 이전 사용자 ```billsmith```에게 이것을 할당할 때 사용합니다. 

```
CALL dbms.security.createRole('accounting')
CALL dbms.security.addRoleToUser('accounting', 'billsmith')
```

**페더레이션(Federated) 사용자 시나리오 (LDAP)**

LDAP 시나리오에서, LDAP 사용자 그룹은 Neo4j의 사용자 지정 역할에 매핑됩니다. 

**예 7.20. 페더레이션(Federated) 사용자 시나리오 (LDAP)**

이 예에서 우리는 사용자 ```accounting``` 역할을 생성하기 위해서 Cypher을 사용합니다. 

```
CALL dbms.security.createRole('accounting')
```

그 후, ```accounting``` 역할을 그룹 ID ```101```를 LDAP 그룹에 매핑합니다. 

```
dbms.security.realms.ldap.authorization.group_to_role_mapping=101=accounting
```

#### 7.1.6.2. 절차 허가 관리 

일반적으로 사용할 때 [섹션 7.1.4.1, 기본 역할](./native-user-and-role-management/native-roles.md)에 설명된대로 Cypher문과 동일한 보안 규칙에 따라 프로시저 함수가 실행됩니다. 예를 들어서 ```publisher```, ```architect``` 및 ```admin```기본 역할이 주어진 사용자는 ```mode=WRITE```를 사용하여 절차를 수행할 수 있지만, ```reader```역할만 가진 사용자는 이를 수행할 수 없습니다. 

서브 그래프 접근 제어를 위해 특정 역할이 할당된 기본 역할로 액세스할 수 없는 절차를 실행합니다. 
프로시저 실행 중에만 사용 권한이 사용자에게 부여됩니다. 다음 두 매개 변수는 원하는 동작을 구성할 떄 사용됩니다.

**dbms.security.procedures.default_allowed**

설정 ```dbms.security.procedures.default_allowed```는 모든 절차를 수행하는 단일 역할 또는 ```dbms.security.procedures.roles```설정과 일치하지 않는 기능을 정의합니다. 

**예 7.21. 절차 및 기능을 실행하시키는 기본 역할 설정**

우리는 다음 구성을 가지고 있다고 가정합니다.

```
dbms.security.procedures.default_allowed=superAdmin
```

이는 다음 효과를 가집니다.  

- ```dbms.security.procedures.roles``` 설정을 구성 해제된 상태로두면 ```superAdmin``` 역할은 모든 사용자 정의 절차 및 기능을 실행할 수 있습니다.  
- ```dbms.security.procedures.roles``` 설정에 역할 및 기능이 정의되어 있으면 ```superAdmin``` 역할은 ```dbms.security.procedures.roles```가 구성하지 않은 모든 사용자 정의 프로시저 및 함수를 실행할 수 있습니다.

```
dbms.security.procedures.roles
```

```dbms.security.procedures.roles``` 설정은 절차를 세부적으로 제어합니다. 

**예 7.22. 특정 절차를 실행하는 역할 설정**

다음 설정을 했다고 가정합니다.

```
dbms.security.procedures.roles=apoc.convert.*:Converter;apoc.load.json.*:Converter,DataSource;apoc.trigger.add:TriggerHappy
```

이는 다음 효과를 가집니다:

- ```Converter```역할을 가진 모든 사용자는 ```apoc.convert``` 네임스페이스 모든 절차를 실행할 수 있습니다. 
- 역할 ```Converter```와 ```DataSource```을 가진 모든 사용자는 ```apoc.load.json``` 네임스페이스 모든 절차를 실행할 수 있습니다. 
- 역할 ```TriggerHappy```을 가진 모든 사용자는 특정 절차 ```apoc.trigger.add```을 실행할 수 있습니다. 

모드를 위협하는 데이터베이스를 조작하려면 프로시저 작동은 실패합니다. 예를 들어, 쓰기 작업을하도록 프로그래밍 된 경우 지정된 프로시저 ```READ``` 모드 작동은 실패합니다. 이는 사용자 또는 역할 구성에 관계없이 발생합니다.

