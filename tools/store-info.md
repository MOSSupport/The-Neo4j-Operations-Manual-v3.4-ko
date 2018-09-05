## 10.4. 저장소 정보 표시 

```
이 섹션에서는 Neo4j Admin의 store-info 명령어에 대해 다룹니다.
```

```neo4j-admin```의 ```store-info``` 명령을 사용하여 Neo4j 데이터베이스 버전을 표시합니다.   

```
neo4j-admin store-info --store=<path-to-store>
```

```store``` 매개변수는 Neo4j 데이터베이스나 백업 경로를 사용합니다. 출력 결과에 ```Store format superseded in:``` 메시지는 백업 복원 중에 저장소 형식 포맷 마이그레이션이 필요한 경우입니다.


Neo4j Admin ```store-info``` 명령어는 실행 중인 데이터베이스에서 사용할 수 없습니다. 데이터베이스가 실행 중일 때 스토어는 보호를 위해 잠겨져 있으며 ```store-info``` 명령어를 사용하면 오류 메시지를 띄우고 실행에 실패합니다.

**예제 10.2. 이전 백업 사용 예제**

예전 Neo4j 백업이 /backups/graph-db.2016-11-11/ 폴더에 저장되어 있고 이를 복원한다고 가정합니다. 백업 시점의 Neo4j는 분명하지 않습니다. Neo4j 버전을 알아 내려면 다음을 실행합니다:

```
$neo4j-home> bin/neo4j-admin store-info --store=/backups/graph-db.2016-11-11
Store format version:         vE.H.0
Introduced in version:        3.0.0
Store format superseded in:   3.0.6
```

출력 결과 데이터베이스가 엔터프라이즈 버전인 상한 형식을 사용하도록 구성되었음을 알 수 있습니다. 즉, 백업은 Neo4j Enterprise 버전에서만 사용할 수 있습니다.

출력 결과에서 이 포맷은 Neo4j 3.0.0에 소개되었으며 새로운 포맷은 Neo4j 3.0.0에 출시되었습니다. 즉, 백업을 복원하려면 반드시 포맷 마이그레이션이 수행해야 합니다. (자세한 내용은 [5장. 업그레이드](../upgrade.md)를 참조합니다.).

**예제 10.3. 새로운 백업 사용 예제**

앞의 예제와 같이, Neo4j의 또 다른 백업이 /backups/graph-db.2016-11-11/ 있다고 가정합니다. 이 경우 출력 결과는 다음과 같습니다:

```
$neo4j-home> bin/neo4j-admin store-info --store=/backups/graph-db.2016-11-11
Store format version:    v0.A.7
Introduced in version:   3.0.0
```

결과는 Neo4j가 모든 버전에서 지원하는 표준 저장소 형식으로 백업되었음을 나타냅니다. 이는 또한 Neo4j의 가장 최신 버전입니다. 즉, 백업을 복원하기 위해 형식 마이그레이션 할 필요가 없음을 의미합니다. 