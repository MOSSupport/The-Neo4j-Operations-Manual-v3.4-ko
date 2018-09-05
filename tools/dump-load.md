## 10.7. 데이터베이스 덤프 및 로드

```
이 절에서는 Neo4j Admin의 덤프 및 로드 명령어에 대해 다룹니다. 
```

Neo4j 데이터베이스는 아래 명령어를 사용하여 덤프 및 로드됩니다. 

```
neo4j-admin dump --database=<database> --to=<destination-path>
```

```
neo4j-admin load --from=<archive-path> --database=<database> [--force]
```


**제한 사항**

```dump``` 및 ```load``` 명령어를 사용하기 전에 데이터베이스를 종료해야 합니다. 

**사용 예시**

- 데이터베이스를 다른 환경으로 옮기기.

```dump``` 및 ```load``` 명령어 사용
```dump```와 ```load``` 명령어 사용은 다른 환경 간 데이터베이스를 전송할 때 권장되며 안전합니다. 파일을 내보내고 가져올 것과 가져올 수없는 파일을 이해합니다.
반면에 데이터베이스 파일 시스템 복사 및 붙여 넣기는 지원하지 않습니다.

- 오프라인 백업

 온라인 오프라인 백업의 차이점은 [섹션 6.1.1, 온라인 및 오프라인 백업](/backup/backup-introduction.md)에 자세히 설명되어 있습니다. 

### 예시 

**예제 10.13. neo4j-admin의 ```dump``` 명령어**

```graph.db```라는 데이터베이스를 ```/backups/graph.db/2016-10-02.dump```파일로 덤프합니다. 덤프 파일의 대상 경로는 /backups/graph.db이며, 명령어 실행전에 해당 디렉터리가 반드시 존재해야 합니다.

```
$neo4j-home> bin/neo4j-admin dump --database=graph.db --to=/backups/graph.db/2016-10-02.dump
$neo4j-home> ls /backups/graph.db
$neo4j-home> 2016-10-02.dump
```


**예제 10.14. neo4j-admin의 ```load``` 명령어**

```/backups/graph.db/2016-10-02.dump``` 파일에 포함된 백업 데이터베이스를 ```graph.db```로 로드합니다. 데이터베이스가 실행 중이므로, 먼저 데이터베이스의 실행을 멈춥니다. ```--force``` 옵션을 사용하면, 기존 데이터베이스에 덮어쓰기 됩니다.

```
$neo4j-home> bin/neo4j stop
Stopping Neo4j.. stopped
$neo4j-home> bin/neo4j-admin load --from=/backups/graph.db/2016-10-02.dump --database=graph.db --force
```

인과 클러스터를 시드하기 위해 ```load``` 명령을 사용하려면, 먼저 각 클러스터 인스턴스에서 ```neo4j-admin unbind```를 수행해야 합니다. 이 절차는 [4.4.2 절. "오프라인 백업의 시드"](/clustering/seed-cluster.md)을 참조하십시오. 