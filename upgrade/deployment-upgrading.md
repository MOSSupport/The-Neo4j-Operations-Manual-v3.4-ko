
## 5.2. 단일 인스턴스 업그레이드

```
이 섹션에서는 단일 Neo4j 인스턴스를 업그레이드하는 방법에 대해  설명합니다.
```

Neo4j Casual 클러스터를 업그레이드하려면, [섹션 5.3, Neo4j Causal 클러스터 업그레이드](../upgrade/causal-cluster.md)를 참조하십시오. 


### 5.2.1. 2.x에서 업그레이드

데비안을 사용하는 경우 [섹션 2.2.1.2"업그레이드"](/installation/linux/debian.md)를 참조하십시오.

**이전 업그레이드 단계**

[섹션 5.1, 업그레이드 계획](../planning.md)를 참조하여 모든 단계를 수행해야 합니다.

**셧다운 및 백업**

1. 데이터 베이스가 실행중이면, 완전히 종료하십시오. 
2. [오프라인 백업](/tools/dump-load.md)을 사용할 경우 Neo4j를 백업해야 합니다.  
   모든 백업이 성공적으로 된 후에 [온라인 백업 툴](/backup.md)을 사용해야 합니다. 
3. [neo4j.conf](/configuration/file-locations.md)를 백업합니다. 

**업그레이드**

1. Neo4j 3.4.6를 설치합니다. 
2. 이전 설치 파일의 환경 설정을 확인하여 모든 사용자 정의 설정을 3.4.6 설치로 이동합니다. 

- Neo4j 버전 2.x와 3.4.6 사이 설정이 많이 변경되었으므로 ```2.x-config-migrator```(디렉토리에서 사용 가능한 툴)를 사용하여 구성 파일을 이동시키는 것이 좋습니다.

예를 들어, ```2.x-config-migrator```는 다음 명령어를 통해 사용합니다:
```
java -jar 2.x-config-migrator.jar path/to/neo4j2.3 path/to/neo4j3.4.6
```

- 출력된 모든 경고를 기록하고 편집 파일을 수동으로 검토하십시오.

3. 이전 버전에서 데이터를 임포트할 때 다음을 사용합니다. 
   
```
neo4j-admin import --mode=database --database=<database-name> --from=<source-directory>
```

4. 데이터베이스 이름이 *graph.db*가 아닐 경우 neo4j.conf 내 ```dbms.active_database```를 데이터베이스 이름으로 설정하십시오.

5. *neo4j.conf*의 3.4.6 버전을 설치할 때,  ```dbms.allow_upgrade=true```로 설정하십시오. 이 설정 없이 Neo4j는 시작되지 않습니다 .

6. Neo4j 3.4.6를 시작하십시오. 데이터베이스 업그레이드는 시작 중에 수행됩니다. 

7. 업그레이드 및 진행 상황 관련 내용은 [*debug.log*](/configuration/file-locations.md)에 기록됩니다.

**업그레이드 이후 단계**

1. 업그레이드가 완료되면, ```dbms.allow_upgrade``` 를 ```false```로 설정하거나 제거해야 합니다. 
2. 데이터베이스를 재시작 합니다. 
3. 업그레이드가 완료 직후 모두 백업하는 것이 좋습니다. 

### 5.2.2. 3.x에서 업그레이드

**업그레이드 이전 단계**

[섹션 5.1, “업그레이드 계획”](./planning.md)를 참조하여 그곳에 적힌 모든 단계를 수행합니다. 

**종료 및 백업**

1. 데이터베이스가 실행 중이면 정상적으로 종료하십시오.
2. [오프라인 백업](/tools/dump-load.md)을 사용할 경우 Neo4j를 백업해야 합니다.  
   모든 백업을 성공한 후, [온라인 백업 툴](/backup.md)을 사용합니다.  
3. [neo4j.conf](/configuration/file-locations.md)를 백업합니다. 
4. [기본 사용자 및 역할 관리](/security/authentication-authorization/ native-user-role-management.md)를 사용하는 경우 *dbms* 디렉토리 (기본 구성 : data/dbms)를 백업하십시오.

**업그레이드**

1. 아래 함수 중에서 하나를 사용하여 Neo4j 3.4.6를 설치 하십시오. 
 
a. tarball 또는  zipfile을 설치에 사용하는 경우:

1) Neo4j 3.4.6를 Untar하거나 unzip 합니다. 
2) [섹션 5.1, “업그레이드 계획”](./planning.md)에 쓰여진 *변경 설정 적용*과 같이  사용자 설정을 3.4.6 설치로 이동합니다.    
3) Neo4j 3.4.6 설치 설정 파일을  `dbms.allow_upgrade=true`로 설정합니다. Neo4j 는 이 설정이 없다면 작동하지 않습니다. 
4) 예전 설치를 새로운 설치 [*data*](/configuration/file-locations.md) 디렉토리로 복사합니다. ```dbms.directories.data```가 *NEO4J_HOME* 외부 디렉토리를 가리키고있는 경우에는 적용되지 않습니다.

b. Debian 또는 RPM을 사용하는 경우:

1) *neo4j.conf*를  ```dbms.allow_upgrade=true``` 로 설정합니다. 
2) Neo4j 3.4.6를 설치합니다. 
3) 메시지가 나타나면 이전 버전 *neo4j.conf* 파일과 Neo4j 3.4.6의 차이점을 검토하십시오. 위 에서 설정 한대로 ```dbms.allow_upgrade = true```를 유지하십시오. Neo4j는 이 구성없이 시작되지 않습니다.

2. Neo4j 3.4.6.를 시작하십시오. 데이터베이스 업그레이드는 시작 단계에서 진행됩니다. 
3. 업그레이드 및 진행 표시 관련 정보는 debug.log에 기록됩니다.

**업그레이드 이후 단계**

1. 업그레이드가 완료되면, ```dbms.allow_upgrade``` 를 `false`로 설정하거나 제거해야 합니다. 
2. 데이터베이스를 재시작 합니다. 
3. 업그레이드가 완료된 직후 모두 백업하는 것이 좋습니다. 

