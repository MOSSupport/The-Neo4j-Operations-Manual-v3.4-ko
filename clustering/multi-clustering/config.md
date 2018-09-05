
### 4.8.2. 다중-클러스터 환경설정 

```
이 섹션에서는 단일 다중-클러스터를 구성하기 위해서 여러가지 인과관계 클러스터를 구성하는 방법에 대해 다룹니다. 
```

이 섹션에서 우리는 아래 그림과 같이 다중-클러스터를 구성할 것 입니다. 각 클러스터는 세 가지 코어 서버와 두 가지 읽기 복제본을 포함합니다:

**그림 4.19. 다중-클러스터를 세 가지 클러스터로 구성**

![multicluster example](https://neo4j.com/docs/operations-manual/3.4/images/multicluster-example.svg)


1. 일반 인과 관계 클러스터를 구성 할 때와 같이 각 클러스터를 구성하십시오. 자세한 내용은 [섹션 4.3, 새로운 클러스터 생성](././setup-new-cluster.md)을 참조하십시오. 

   예를 들어, 각각 코어 멤버를 포함하는 세 가지 클러스터의 다중-클러스터를 구성할 때 그 클러스터에서 ```causal_clustering.minimum_core_cluster_size_at_formation```의 값을```3```으로 설정해야 합니다. ( ```9```와 반대로, 모든 핵심 멤버의 합계가 될 것입니다). 
2. 각 클러스터를 시작하고 클러스터가 예상대로 작동하는지 테스트하십시오.
3. 모든 클러스터의 모든 멤버를 종료하십시오. 
4. 각 데이터베이스/클러스터에 이름을 할당 하십시오:

 각 클러스터의 멤버를 [causal_clustering.database](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_causal_clustering.database) 파일을 참조하여 설정하십시오. 

**예 4.19. 클러스터에 이름 할당**

아래 내용을 *neo4j.conf*에 추가해서 데이터베이스에 ```foo``` 이름을 붙입니다. 

```
causal_clustering.database=foo
```

클러스터의 의도된 회원 설정에 ```causal_clustering.database```이 없으면 ```default```가 있는 별도 클러스터가 형성됩니다. 

**제한 사항:** 

기본값이 아닌 이름으로 인스턴스를 처음 실행하면 그 이름은 유지됩니다. 해당 인스턴스의 다음 것을 시작할 때, 이름 변경이 감지되면 시작에 실패합니다. 인스턴스에 설정된 데이터베이스 이름은 ```neo4j-admin unbind```을 사용하여 인스턴스를 언바인드 할 때만 변경이 가능합니다. 

5. 검색 멤버를 구성하십시오.

다중-클러스터를 설정할 때 클러스터 모든 구성성분에 전 멤버를 위치시키려면 검색 서비스를 설정해야합니다. 이것은 위의 그림에 표시된 다중 클러스터 전체 검색 서비스에 참여할 수 있게 합니다. 


이것은 각 클러스터의 ```causal_clustering_initial_discovery_members``` 설정 값을 결합하여 실행합니다. 더 자세한 내용은 [섹션 4.2.1, 검색 프로토콜](./lifecycle.md)을 참조하십시오. 

**예 4.20. 다중-클러스터를 위한 초기 검색 설정**

다음과 같이 초기 검색 멤버 설정에 3가지 클러스터가 있다고 가정합시다. 

```
# `foo` members
causal_clustering.initial_discovery_members=neo20:5000,neo21:5000,neo22:5000
```

```
# `bar` members
causal_clustering.initial_discovery_members=neo23:5000,neo24:5000,neo25:5000
```

```
# `baz` members
causal_clustering.initial_discovery_members=neo26:5000,neo27:5000,neo28:5000
```

이 세가지 클러스터로 다중-클러스터를 생성하려면, 모든 클러스터의 모든 멤버 설정 값을 업데이트 하십시오.

```
# `foo`,`bar and `baz` members
causal_clustering.initial_discovery_members=neo20:5000,neo21:5000,neo22:5000,neo23:5000,neo24:5000,neo25:5000,neo26:5000,neo27:5000,neo28:5000
```

6. 다중-클러스터 시작 및 테스트:
   
a. 다중-클러스터의 모든 멤버를 시작하십시오. 

b. ```dbms.cluster.overview ()``` 프로시저를 사용하여 다중-클러스터가 예상대로 작동하는지 테스트하십시오.

**예 4.21. 다중-클러스터 테스트**

다중-클러스터가 제대로 작동하는지 확인 및 참여 멤버 정보와 이들이 다루는 데이터베이스에 대해 알려면, ```dbms.cluster.overview()``` 프로시저를 사용하면 됩니다.

```
CALL dbms.cluster.overview()
```

프로시저는 아래와 같은 결과를 나타낼 것 입니다:

| id                                   | 주소                                                       | 역할         | 그룹 | 데이터베이스 |
| ------------------------------------ | ---------------------------------------------------------- | ------------ | ---- | ------------ |
| 08eb9305-53b9-4394-9237-0f0d63bb05d5 | [bolt://neo20:7687, http://neo20:7474, https://neo20:7473] | LEADER       | []   | foo          |
| cb0c729d-233c-452f-8f06-f2553e08f149 | [bolt://neo21:7687, http://neo21:7474, https://neo21:7473] | FOLLOWER     | []   | foo          |
| ded9eed2-dd3a-4574-bc08-6a569f91ec5c | [bolt://neo22:7687, http://neo22:7474, https://neo22:7473] | FOLLOWER     | []   | foo          |
| 3df6ec18-6333-4fb8-8b34-db7ecdfd967f | [bolt://neo23:7687, http://neo23:7474, https://neo23:7473] | LEADER       | []   | bar          |
| 395f708f-ae0c-4e24-a48d-9e90d4d63590 | [bolt://neo24:7687, http://neo24:7474, https://neo24:7473] | FOLLOWER     | []   | bar          |
| fef25f21-6a99-4694-ae1c-10d91465acd2 | [bolt://neo25:7687, http://neo25:7474, https://neo25:7473] | FOLLOWER     | []   | bar          |
| dcc57b70-d4a4-486b-ae57-e635bdcb1563 | [bolt://neo26:7687, http://neo26:7474, https://neo26:7473] | LEADER       | []   | baz          |
| a664ede1-1f87-4828-aee8-1ffd2bcce09c | [bolt://neo27:7687, http://neo27:7474, https://neo27:7473] | FOLLOWER     | []   | baz          |
| 17addd42-a9f9-427e-9b32-e59d650e35af | [bolt://neo28:7687, http://neo28:7474, https://neo28:7473] | FOLLOWER     | []   | baz          |
| a049dea4-d4de-421c-898c-e638fa98440b | [bolt://neo30:7687, http://neo30:7474, https://neo30:7473] | READ_REPLICA | []   | foo          |
| 894ced90-85cd-47c2-9a75-f86b22ef74bf | [bolt://neo31:7687, http://neo31:7474, https://neo31:7473] | READ_REPLICA | []   | foo          |
| 151e46c9-14b8-49bf-bf25-cf09cf3ad582 | [bolt://neo32:7687, http://neo32:7474, https://neo32:7473] | READ_REPLICA | []   | bar          |
| e1d153aa-c3e6-40fd-acf7-d73c6c79baf4 | [bolt://neo33:7687, http://neo33:7474, https://neo33:7473] | READ_REPLICA | []   | bar          |
| 6e0b1801-9dba-4925-bc45-ceee91c10338 | [bolt://neo34:7687, http://neo34:7474, https://neo34:7473] | READ_REPLICA | []   | baz          |
| 87577513-db46-4ca3-a75f-3d8919f9b454 | [bolt://neo35:7687, http://neo35:7474, https://neo35:7473] | READ_REPLICA | []   | baz          |


```dbms.cluster.overview()``` 프로시저는 어떤 클러스터 멤버가 쿼리를 실행할 때 사용되고, 그 멤버가 속한 클러스터 구성에 관계없이 전체 멀티 클러스터 개요를 제공합니다. 각 클러스터 멤버가 다루는 데이터베이스 이름은 *database* 컬럼에서 제공됩니다. 