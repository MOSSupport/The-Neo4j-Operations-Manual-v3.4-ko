
## 4.4. 클러스터 시드(seed)

```
이 섹션에서는 새로운 Neo4j 인과관계클러스터를 존재하는 데이터로 시드하는 방법에 대해 설명합니다.
```

[섹션 4.3, 새로운 클러스터 생성](./setup-new-cluster.md)에서는 빈 저장소로 클러스터를 생성하는 방법에 대해 다뤘습니다. 그러나 Neo4j를 다루거나 프로덕션 환경을 설정하는 것과는 관계없이 클러스터에 전송할 기존 데이터가 있을 수도 있습니다.

이 섹션에서는 클러스터 시드하는 방법을 아래와 같이 설명합니다:

- 온라인 백업에서 시드
- 오프라인 백업에서 시드
- 임포트 풀을 사용해서 시드

이 섹션에서는 기존 온라인 또는 오프라인 Neo4j 데이터베이스에서 시드되거나 다른 데이터 소스에서 가져온 데이터를 포함하는 인과관계 클러스터 생성 방법에 대해 설명합니다. 일반적으로 클러스터를 시드 단계는 데이터 형식과 관계 없이 동일한 패턴을 가집니다. 

1. Neo4j 코어-클러스터 전용 생성 
2. 클러스터 시드
3. 클러스터 시작

클러스터를 시드할 때 사용하는 데이터베이스는 클러스터 자체 Neo4j 버전과 동일해야 합니다. 

### 4.4.1. 온라인 백업에서 시드 

예를 들어, 현재 실행되는 Neo4j 인스턴스의 온라인 백업 결과로 Neo4j 데이터 베이스는 안전하게 백업되었다고 가정합니다. (온라인 백업과 관련된 상세한 내용은 [챕터 6, 백업](/backup.md)을 참조하십시오.) 이것은 독립된 Neo4j 인스턴스, Neo4j는 고가용 클러스 또는 다른 Neo4j 인과관계 클러스터의 실행중인 인스턴스가될 수 있습니다. 또한, 이곳에 설명된 프로세스는 기존 읽기 복제본의 새로운 인과관계 클러스터를 시드할 때 사용할 수 있습니다. 예를 들어 재해 발생 시 일부 서버가 작동 가능한 상태를 유지하는 엄청난 피해 복구에 유용할 수 있습니다. 

**새로운 Neo4j 코어 전용 클러스터 생성**

[섹션 4.3.1, 코어 전용 클러스터 환경설정](./setup-new-cluster.md)을 참조하여 Neo4j 코어 전용 클러스터를 생성합니다. 작동을 확인하기 위해서 클러스터를 시작 후에 클러스터를 중지하고 각 인스턴스에서 ```neo4j-admin unbind```을 실행시킵니다. 

**클러스터 시드**

모든 코어 인스턴스에 백업된 데이터베이스에서 시드 저장소를 저장하려면 ```neo4j-admin```의 ```restore``` 명령어를 사용합니다. 모든 인스턴스가 저장소와 시드되었으므로, 클러스터는 인스턴스가 한번 시작되면 바로 사용할 수 있습니다.

**예 4.2. 모든 코어 멤버 시드를 위해 저장소 백업**

이 예에서는 기본 데이터베이스 명이 ```graph.db```이고, *seed-dir* 하위 디렉토리에 유효한 백업 파일이 있다고 가정합니다. 

다른 설정이 있을 경우, 명령어 라인을 다음과 같이 바꾸십시오. 

```
neo4j-01$ ./bin/neo4j-admin restore --from=seed-dir --database=graph.db
```

```
neo4j-02$ ./bin/neo4j-admin restore --from=seed-dir --database=graph.db
```

```
neo4j-03$ ./bin/neo4j-admin restore --from=seed-dir --database=graph.db
```

**클러스터 시작**

이 경우, 코어 클러스터의 모든 인스턴스는 그래프 데이터를 포함하는 저장소가 있습니다. 이 사이에 코어 서버는 클러스터를 형성하는데 필요한 모든 것이 있습니다. 이제 모든 인스턴스를 시작할 준비가 되었습니다. 클러스터는 형성될 것이고, 데이터는 인스턴스 사이에 복제됩니다. 

**예 4.3. 각 코어 인스턴스 시작**

```
neo4j-01$ ./bin/neo4j start
```

```
neo4j-02$ ./bin/neo4j start
```

```
neo4j-03$ ./bin/neo4j start
```

### 4.4.2. 오프라인 백업에서 시드

Neo4j 커뮤니티에서 엔터프라이즈  버전을로 업그레이드하거나 엔터프라이즈 데이터베이스를 다른 사이트로 이식하는 것과 같이 오프라인 방식으로 데이터베이스를 시드하는 경우가 있습니다. 오프라인 백업을 처리할 때, ```neo4j-admin```의 명령어```dump``` 와 ```load```를 사용합니다. 이것과 관련된 자세한 사항은, [섹션 10.7, 데이터베이스 덤프 및 로드](/tools/dump-load.md)를 사용하면 됩니다. 

오프라인 백업에서 시드하는 총 프로세스는 백업이 모든 코어 멤버에 저장해는 것을 빼고는 온라인 백업 방식과 동일합니다. 

**새로운 Neo4j 코어전용 클러스터 생성**

[섹션 4.3.1, 코어전용 클러스터 환경설정](./setup-new-cluster.md)에서 설명된 것과 같이 새로운 코어전용 클러스터를 생성합니다. 작동여부를 확인하기 위해서 클러스터를 시작할 경우, 각 인스턴스에서 ```neo4j-admin unbind```을 멈추고 작동시켜야 합니다. 

**클러스터 시드**

```neo4j-admin```의 명령어 ```load```을 사용해서 덤프 파일을 새롭게 생성된 각 코어 멤버에 로딩하여 클러스터를 시드합니다.  

**예시 4.4. 각 클러스터 멤버에 데이터베이스 로드**

이 예에서 우리는 ```neo4j-admin```의 ```dump```명령어를 사용해서 Neo4j 데이터베이스 오프라인 백업 파일을 가지고 있다고 가정합니다. 기본 데이터베이스 명은 ```graph.db```이며 *seed-dir* 디렉토리 내에 *graph.dump*라고 불리는 덤프파일이 있습니다. 다른 설정이 있다면, 명령어 라인 요소를 바꾸면 됩니다. 

```
neo4j-01$ ./bin/neo4j-admin load --from=seed-dir/graph.dump --database=graph.db
```

```
neo4j-02$ ./bin/neo4j-admin load --from=seed-dir/graph.dump --database=graph.db
```

```
neo4j-03$ ./bin/neo4j-admin load --from=seed-dir/graph.dump --database=graph.db
```


**클러스터 시작하기**

이 경우에 코어 클러스터의 모든 인스턴스에는 그래프 데이터를 포함하는 저장소가 있습니다. 

### 4.4.3.임포트 도구를 사용한 시드 

임포트 된 데이터를 기반으로 클러스터를 생성할 때, 우리는 오프라인 백업에서 시드하는 것과 매우 비슷한 형태를 따릅니다. 

**새로운 코어 전용 클러스터 생성**

[섹션 4.3.1, 코어전용 클러스터 환경설정](./setup-new-cluster.md)에서 설명된 것과 같이 새로운 코어전용 클러스터를 생성합니다. 작동여부를 확인하기 위해서 클러스터를 시작할 경우, 각 인스턴스에서 ```neo4j-admin unbind```을 멈추고 작동시켜야 합니다. 

**클러스터 시드**

```neo4j-admin```의 명령어 ```load```을 사용해서 덤프 파일을 새롭게 생성된 각 코어 멤버에 로딩하여 클러스터를 시드합니다.  

**클러스터 시작하기**

이 경우에 코어 클러스터의 모든 인스턴스에는 그래프 데이터를 포함하는 저장소가 있습니다. 
데이터베이스를 옮기고 시드하기 위해서 내부 데이터 디렉토리를 복사-붙여넣기를 사용하는 것은 지원되지 않습니다. 새로운 클러스터를 위해 사용하려는 데이터 Neo4j 데이터베이스가 있는 경우 이 섹션에서 설명하는 프로세스 중 하나를 수행하면 됩니다. 