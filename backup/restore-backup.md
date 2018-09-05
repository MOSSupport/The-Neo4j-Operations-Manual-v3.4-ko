
## 6.3. 백업 저장

```
이 섹션에서는 Neo4j 데이터베이스에서 백업을 저장하는 방법에 대해서 설명합니다. 
```

### 6.3.1. 저장 명령어 

Neo4j 데이터베이스는 ```neo4j-admin```의 ```restore``` 명령어를 사용해서 저장할 수 있습니다. 

**구문**

```neo4j-admin restore --from=<backup-directory> [--database=<name>] [--force[=<true|false>]]```

**옵션**

| 옵션       | 기본 값  | 설명                                |
| ---------- | -------- | ----------------------------------- |
| --from     |          | 복원 할 백업 경로                   |
| --database | graph.db | 데이터베이스 명                     |
| --force    | false    | 기존 데이터베이스를 교체해야할 경우 |


### 6.3.2. 단일 데이터베이스 저장 

**예 6.3. 단일 데이터베이스 저장**

*/mnt/backup/graph.db-backup*에 위치한 데이터베이스 `graph.db`를 저장합니다. 복원할 데이터베이스는 종료해야 합니다. 

```
neo4j-home> bin/neo4j stop
neo4j-home> bin/neo4j-admin restore --from=/mnt/backup/graph.db-backup --database=graph.db --force
neo4j-home> bin/neo4j start
```

### 6.3.3. Causal 클러스터 저장 

Casual 클러스터 백업을 복원하려면, 아래 단계를 따르십시오:

1. 클러스터 내 모든 데이터베이스 인스턴스를 종료하십시오. 

2. 각 핵심 서버에 ```neo4j-admin unbind``` 명령어를 실행하십시오. 

3. [단일 데이터베이스 저장](/backup/restore-backup.md)의 지침을 따라서 각 인스턴스에 백업을 복원합니다.

4. 새로운 하드웨어에 복원하려면, *neo4j.conf*의 *Causal Clustering*를 검토하십시오. 

   특히, `causal_clustering.initial_discovery_members`, `causal_clustering.minimum_core_cluster_size_at_formation`, `causal_clustering.minimum_core_cluster_size_at_runtime` 
   설정을 확인하고 새 설정이 올바르게 반영되었는지 확인하십시오.

5. 데이터베이스 인스턴스를 시작하십시오. 

### 6.3.4. HA 클러스터 복원 

HA 클러스터 환경에서 백업을 복원하려면, 아래 단계를 따르십시오. 

1. 클러스터 내 모든 데이터베이스를 종료합니다. 

2. [단일 데이터베이스 복원](/backup/restore-backup.md). 지침을 참조해서 각 인스턴스의 모든 인스턴스를 복원합니다.  

3. 새로운 하드웨어에 복원하려면, *neo4j.conf*의 *Causal Clustering*를 검토하십시오. 

특히, ```ha.initial_hosts``` 설정을 확인하여 나열된 서버가 복원되는 서버를 반영하는지 확인하십시오.

4. 마스터로 시작하는 데이터베이스 인스턴스를 시작하십시오.
