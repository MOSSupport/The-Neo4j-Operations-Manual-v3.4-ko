
## 4.3. 새로운 클러스터 생성

```
이 섹션에서는 새로운 인과관계 클러스터를 배포하는 방법에 대해서 설명합니다. 
```

이 섹션에서는 세가지 코어 인스턴스로 구성되는 새로운 인과관계 클러스터를 설정하는 방법에 대해 다룹니다. 이 것은 내 결함성 코어 클러스터에서 필요한 최소 서버 개수입니다. 읽기 복사본과 같이 코어 서버를 추가하는 방법에 대해 배웁니다. 기본 패턴에서 이전에 배운 것을 모든 사이즈의 클러스터를 생성을 확장합니다. 

다루는 내용은 다음과 같습니다:

- 코어 전용 클러스터 설정 
- 존재하는 클러스터에 코어 서버 추가
- 존재하는 클러스터에 읽기 복사본 추가
- 클러스터에서 코어 제거
- 팔로워 전용의 클러스터 리더쉽 인스턴스 성향 
- 클러스터 멤버의 초기 검색 

클러스터링 구조 및 개념 요약은 [인과관계 클러스터 소개](./introduction.md)를 참조하십시오. 현재 Neo4j 인스턴스에서 데이터를 임포트하는 방법은 설명하지 않습니다. 기존 Neo4j 데이터베이스를 사용하여 새로운 인과 관계 클러스터를 구축하는 방법은 [인과관계 클러스터 시드](./seed-cluster.md)을 참조하십시오. 

로컬 기계에서 인과관계 클러스터를 설정할 때, [섹션 B.1, 인과관계 로컬 클러스터 설정](https://neo4j.com/docs/operations-manual/3.4/tutorial/local-causal-cluster/)을 참조하십시오. 

### 4.3.1. 코어 전용 클러스터 설정 

이 섹션에서는 이미 Neo4j 엔터프라이즈 버전을 [Neo4j 다운로드 사이트](https://neo4j.com/download/other-releases/#releases)에서 다운 받고 서버에 설치하여 클러스터를 구성했다고 가정합니다. 

새로운 인과관계 클러스터를 배치할 떄 기본 클러스터를 구성시 다음 구성을 설정해야 합니다. 
 
아래 설정은 헤더 "네트워크 커넥터 환경설정" [neo4j.conf](/configuration/file-locations.md)에서 확인할 수 있습니다. 

```dbms.connectors.default_listen_address```

이 컴퓨터가 수신 메시지를 받을 때 사용하는 주소 또는 네트워크 인터페이스입니다. 주석 처리를 제거하면이 값이 ```0.0.0.0```으로 설정되어 Neo4j가 모든 네트워크 인터페이스에 결합됩니다. ```dbms.connectors.default_listen_address=0.0.0.0``` 줄 주석처리를 제거합니다. 


```dbms.connectors.default_advertised_address```

다른 기계가 연결할 주소입니다. 일반적으로, 이것은 서버의 공용 IP을 사용하도록 설정해야 합니다. 예를 들어, IP 주소가 33.44.55.66라면, 이 설정은 ```dbms.connectors.default_advertised_address=33.44.55.66```가 됩니다. 


아래 설정은 인과관계 클러스터링 환경설정 헤더 밑인 [neo4j.conf](/configuration/file-locations.md)에 위치합니다. 

- ```dbms.mode```

단일 데이터베이스 인스턴스의 작동 모드 입니다. 인과관계 클러스터링에는 두 가지 모드가 사용가능 합니다: ```CORE``` 또는 ```READ_REPLICA```. 세 가지 인스턴스에서 우리는`CORE` 모드를 사용할 것입니다. ```#dbms.mode = CORE``` 줄 주석 처리를 제거하십시오.

```causal_clustering.minimum_core_cluster_size_at_formation```

클러스터 형성 시 코어 기계의 최소 개수 입니다. 클러스터는 이 설정으로 정의된 코어없이 형성되지 않으며, 일반적으로 전체 및 고정량으로 구성해야 합니다. 예를 들어서, ```causal_clustering.minimum_core_cluster_size_at_formation=3```는 시작을 위해서 클러스터는 세 가지 코어 멤버를 필요로 합니다. 


```causal_clustering.minimum_core_cluster_size_at_runtime```

일치 하는 그룹에 존재하는 코어 인스턴스의 최소 개수 입니다. 예를 들어서, ```causal_clustering.minimum_core_cluster_size_at_runtime=3```는 
공유 그룹이 손실 된 경우에도 일치하는 그룹 사이즈가 3이 되면, 클러스터가 인스턴스를 동적으로 투표 아웃하지 않도록 지정합니다.

```causal_clustering.initial_discovery_members```

초기 코어 클러스터 멤버의 네트워크 주소 설정은 이 코어 나 읽기 복제본 인스턴스를 부트스트랩할 수 있도록 합니다. 이 값의 해석 방법은 ```causal_clustering.discovery_type```으로  결정됩니다. 검색 종류가 그 기본 값인 ```LIST```로 되어있다면, 초기 검색 멤버는 /포트 쌍인 주소 리스트의 ,로 분류됩니다. 예 ```core1:5000,core2:5000,core3:5000```. 검색 서비스의 기본 포트는 ```:5000``` 입니다. 이 설정의 로컬 기계에 주소를 포함해야 합니다. 

```causal_clustering.discovery_type```

시작 시 클러스터의 다른 멤버의 주소를 결정하기 위해`config_causal_clustering.initial_discovery_members`에 제공된 값과 함께 사용하는 메커니즘입니다. 가장 단순한 경우 ```config_causal_clustering.discovery_type```은 ```LIST```로 설정되며 지정된 네트워크 주소는 작동중인 Neo4j Core 서버 인스턴스로 해석합니다. 이 값 (```LIST```)는 클러스터 내 코어 인스턴스의 호스트 명이나 IP 주소를 필요로 합니다. 다른 코어 인스턴스의 주소를 알지 못하면, [DNS로 초기 클러스터 멤버 검색](./setup-new-cluster.md)에 설명된 검색 메커니즘을 바꿔야 합니다. 

이 설정을 각 인스턴스의 환경 설정 파일에 적용하십시오. 개별 값은 같을 수도 있습니다. 

**예 4.1. 코어 전용 클러스터 시작**

```
server-1$ ./bin/neo4j start

```

```
server-2$ ./bin/neo4j start
```

```
server-3$ ./bin/neo4j start
```

서버를 시작할 때 따라 가려면 [neo4j.log](/configuration/file-locations.md).
에 있는 메시지를 따라갈 수 있습니다. 인스턴스를 클러스터에 결합하는 동안 서버는 이용하지 못할 수도 있습니다. 

이제 세 가지 서버에 접속하고 그 상태를 확인할 수 있습니다. 아래 웹 브라우저 위치를 열고 쿼리를 실행하십시오. ```CALL dbms.cluster.overview()```. 이것은 클러스터의 상태와 각 클러스터 멤버의 정보를 나타냅니다. 

- ```http://server-1:7474/```
- ```http://server-2:7474/```
- ```http://server-3:7474/```

이제 실행되는 세 가지 인스턴스에 대하여 Neo4j 인과관계가 있습니다. 

### 4.3.2. 존재하는 클러터에 코어 서버 추가 

인스턴스를 코어 클러스터에 추가하는 것은 새로운 데이터베이스 서버를 
 [섹션 4.3.1, 코어 전용 클러스터 설정](./setup-new-cluster.md)에 설명된 것과 같이 적절한 환경에 설정하는 것과 같습니다. 아래 지시를 따라 원하는 새로운 코어 서버 설정의 *neo4j.conf*를 변경해야 합니다. 

- ```dbms.mode=CORE``` 설정 
- ```causal_clustering.initial_discovery_members```가 존재하는 코어 서버에 일치하도록 변수 설정 

설정 후, 서버를 시작하면 새로운 서버는 존재하는 클러스터와 결합할 것 입니다. 많은 데이터와 결합된 인스턴스의 클러스터는 새로운 인스턴스가 클러스터에서 데이터를 받고 이용할 수 있을 때까지 오래 소요됩니다. 서버가 해당 피어의 그래프 데이터를 복사하면 사용 가능하게됩니다.

### 4.3.3. 존재하는 클러스터에 읽기 복제본 추가 

초기 읽기 복제본 환경설정은 neo4j.conf을 통해서 비슷하게 코어 서버로 제공됩니다. 읽기 복제본은 클러스터 정족수에 관여하지 않기 떄문에 이 설정은 더 짧습니다. 이들은 검색 프로토콜을 실행하기위해 결합되는 일부 코어 서버의 주소를 알아야합니다. (참조: [섹션 4.2.1, 프로토콜 검색](./clustering.lifecycle.md). 작업이 완료되면, 초기 검색 읽기 복제본은 코어 서버를 사용하여 적절한 캐치 업을 선택합니다. (참조: [섹션 4.2.5, 캐치업 프로토콜](./clustering/lifecycle.md)). 

*neo4j.conf* 파일 내에 인과관계 클러스터링 설정 섹션에서 아래 설정은 변경해야 합니다.   

- 작동 모드를 읽기 복제본으로 설정 : ```dbms.mode=READ_REPLICA```.
- ```causal_clustering.initial_discovery_members``` 변수가 코어 서버와 일치하도록 변수 설정 .


### 4.3.4. 클러스터에서 코어 제거 

코어 서버는 ```neo4j-admin unbind``` 명령어를 이용해서 독립 인스턴스로 다운그레이드될 수 있습니다. 

서버가 클러스터에서 풀리면, 저장소 파일은 Neo4j 독립 인스턴스와 동일해집니다. 이 관점에서 파일은 ```SINGLE``` 모드에서 재시작하여 독립 인스턴스를 실행할 수 있습니다. 

디스크에 기록된 핵심 서버 상태 인스턴스는 독립 서버 인스턴스와는 다릅니다. 인스턴스는 클러스터에서 연결 해지되면 클러스터와 재결합할 수 없다는 것을 알아야 합니다. 이것은 클러스터와 단일 인스턴스가 각각 다른 쓰기가 적용된 독립 데이터베이스이기 때문입니다. 기술적으로 클러스터에는 Raft 로그에 항목이 기록되고 독립 실행 형 인스턴스는 독립 인스턴스에 Raft 로그가 없으므로 트랜잭션 로그에 직접 기록됩니다.

독립 인스턴스를 클러스터에 다시 넣으려면, 로그와 저장소는 일치하지 않게 됩니다. 운영자는 데이터가 복제 될 것이라는 희망으로 독립형 데이터베이스를 클러스터에 병합하면 안됩니다. 그렇지 않으면 예측할 수 없는 클러스터 동작이 발생할 수 있습니다.

### 4.3.5. 팔로워 전용 인스턴스를 사용한 클러스터 리더십 성향

인과관계 클러스터 내 코어 서버는 일관성 및 안정성을 위해서 Raft 프로토콜을 사용합니다. Raft의 세부 구현 사항은 *Leader* 역할로 기본 로그에 순서를 정하고 리더의 상태를 복제하고 *Followers* 역할을하는 다른 인스턴스를 지정하는 것입니다. Neo4j 관점에서 이것은 데이터베이스가 현재 리더 역할을 수행하는 코어 인스턴스에 의해 정렬됨을 의미합니다. 


일부 상황에서, 운영자는 일부 인스턴스의 리더 역할을 수행을 적극적으로 막을 수 있습니다. 예를 들어서, 다중-데이터 센터 시나리오에서 성능이나 관리를 이유로 리더가 선호하는 위치에 있는지 확인할 수 있습니다. Neo4j 인과관계 클러스터에서 우리는 인스턴스를 *리더가 되지못하게* 설정할 수 있으며, 이는 항상 팔로워로 남는것을 의미합니다. 이것은 ```causal_clustering.refuse_to_be_leader```에서 설정할 수 있습니다. 클러스터 내 첫번 째 우선순위가 가용성을 극대화하는 것이기 때문에 일반적으로 이 옵션을 사용은 바람직하지 않습니다. 
가장 높은 가용성은 인스턴스가 실패로 인해 클러스터의 주도권을 잡는 데서 비롯됩니다.

피상적 유사성에도 불구하고 리더가 아닌 핵심 인스턴스는 읽기 복제본과 동일하지 않습니다. 읽기 복제본은 트랜잭션 처리에 참여하지 않으며 클러스터 토폴로지 관리에 관여 할 수 없습니다.

역으로 오직 팔로워 코어 인스턴스는 트랜잭션과 클러스터 회원 요청을 일관성과 안정성을 보장하기 위해서 Raft마다 처리할 것 입니다. 

### 4.3.6. 클러스터 멤버 초기 검색 

Neo4j 코어 서버가 시작할 때, 새로운 클러스터를 구성하거나 존재하는 클러스터와 결합하기 위해서  다른 코어 멤버와 연결해야 합니다. 다른 멤버와 연결하기 위해서 코어 서버는 그 주소를 알아야 합니다. 주소는 호스트명 또는 IP주소 및 검색 서비스가 수신하는 포트로 구성됩니다. 

클러스터 멤버가 다른 것을 발견하면, 이들은 나머지 클러스터가 아는 것을 공유합니다. 그렇기 때문에 각 인스턴스가 클러스터 내 다른 모든 인스턴스 주소를 가질 필요가 없습니다. 그러나 초기 발견 멤버가 둘 이상의 분리 된 세트를 형성하면 클러스터를 형성 할 수 없습니다.

다른 클러스터 구성원의 주소를 알고있는 경우 명시 적으로 나열 할 수 있습니다. 이 경우 기본 ```causal_clustering.discovery_type = LIST```을 사용하고 각 머신의 설정에서 주소를 하드 코딩 할 수 있습니다. 예시: ```causal_clustering.initial_discovery_members=10.0.0.1:5000,10.0.0.2:5000,10.0.0.3:5000```.


핵심 구성원 주소를 명시적으로 나열하는 것은 검색할 때 편리하지만 제한 사항이 있습니다.

- 첫째, 코어 멤버가 대체되고 새 멤버가 다른 주소를 가질 경우 기존 리스트는 최신이 아닙니다. 기존 회원과 동일한 주소로 신입 회원에게 설정하여 구식 명단을 피할 수는 있지만, 항상 실용적인 것은 아닙니다. 이는 예를 들어 컨테이너 결합을 사용하여 인과관계 클러스터를 배포 할 때 발생할 수 있습니다.

- 둘째, 같은 환경에서 클러스터를 설정할 때 주소는 알려지지 않습니다. 이는 컨테이너 결합을 사용하여 인과관계 클러스터를 배포 할 때 발생하는 경우와 같습니다. 

실용적이지 않거나 발견 할 클러스터 구성원의 주소를 리스트할 수 없는 때는 DNS를 사용하여 추가 검색 메커니즘을 제공합니다. 

### 4.3.7. DNS로 초기 클러스터 멤버 검색


기본 ```LIST``` 검색 유형 외에 코어 클러스터 구성원의 주소를 얻을 때 사용하는 두 가지 메커니즘이 있습니다. 이 방법 모두 DNS를 사용합니다. 

- ```config_causal_clustering.discovery_type=DNS``` 를 사용하면 초기 검색 멤버는 연결을 위해 IP 주소를 찾을 떄 DNS A 기록을 사용합니다. 

- ```config_causal_clustering.discovery_type=SRV```을 사용하면, 초기 검색 멤버가 DNS SRV 레코드에서 확인해서 IP주소 및 호스트 명과 연결할 서비스 포트를 찾을 수 있습니다. 

```discovery_type=DNS```을 사용할 때, ```causal_clustering.initial_discovery_members``` 는 검색 서비스 포트 및 단일 도메인 이름으로 설정해야 합니다. 예: ```causal_clustering.initial_discovery_members=cluster1.neo4j.example.com:5000```.

DNS 검색을 실행할 때, 도메인 이름은 각 코어 멤버별로 A 레코드를 반환해야 합니다. DNS에서 반환된 각 A레코드는 코어 멤버의 IP 주소를 포함해야 합니다. 설정된 코어 서버는 클러스터와 결합하거나 형성하도록 A 레코드에서 모든 IP 주소를 사용합니다. 

```discovery_type=DNS```가 사용된다면,  모든 코어의 검색 포트는 같아야 합니다. 가능하지 않으면, ```discovery_type=SRV``` 사용을 고려해보십시오. 

```discovery_type=SRV```을 사용할 때, ```causal_clustering.initial_discovery_members``` 는 단일 도메인으로 설정되어야하고 포트는 0으로 설정해야 합니다. 예: ```causal_clustering.initial_discovery_members=cluster1.neo4j.example.com:0```. 도메인 이름은 DNS 검색을 실행할 때 단일 SRV 레코드를 반환해야 합니다. DNS 에서 반환된 SRV 기록은 검색할 코어 서버의 IP주소 또는 호스트 명 및 검색 포트를 포함해야 합니다. 설정된 코어 서버는 클러스터와 결합하거나 형성하도록 SRV 레코드에서 모든 주소를 사용합니다. 

```DNS```와 ```SRV``` 환경 설정에서, DNS 레코드 검색은 인스턴스가 시작될 떄 수행됩니다. 
인스턴스가 클러스터와 결합하면 검색 서비스의 일부로 핵심 구성원간에 추가 구성원 변경 내용이 전달됩니다.

### 4.3.8. 저장소 복사 

인과관계 클러스터링은 견고하고 구성 가능한 저장소 복사 프로토콜을 사용합니다. 저장소 사본이 시작되면 먼저 지정된 인스턴스로 준비 요청을 전송합니다. 준비 요청이 완료되면 클라이언트는 파일 및 인덱스 클러스터 요청을 제공된 업스트림 멤버에게 보냅니다. 요청별 재시도 로직은 ```causal_clustering.store_copy_max_retry_time_per_request```를 통해 수정할 수 있습니다. 만약이 요청이 실패되어 최대 재시도 횟수를 초과하면 재시도를 멈추고 저장소 복사는 실패할 것입니다. 

```catchal_clustering.catch_up_client_inactivity_timeout```을 사용하여 모든 캐치 업 요청별 비활성화 제한 시간을 설정하십시오. 이 설정은 트랜잭션을 가져 오는 것을 포함하여 캐치 업 클라이언트의 모든 요청에 ​​사용됩니다.
 
저장소 복사를 시작하는 세가지 시나리오가 있습니다. 업스트림 선택 전략은 각 시나리오 별로 다릅니다. 

**백업**

업스트림 전략은 ```neo4j-admin backup```을 사용해서 고정 숫자로 설정합니다. 모든 요청은 특정 숫자로 보내집니다. 

**빈 저장소에 새 회원 넣기**

인스턴스에서 설정된 업스트립 전략을 사용할 것 입니다.

**뒤 떨어진 인스턴스**

인스턴스에서 설정된 업스트림 전략을 사용할 것 입니다.


업스트림 전략은 코어 및 읽기 복제본에 따라 다릅니다. 코어는 저장소의 가장 최신 정보를 얻기 위해서 항상 리더에게 준비 요청을 전송합니다. 코어에 대한 파일 및 인덱스 요청 전략은 그 밖의 다른 요청을 읽기 복제본과는 다른 모든 코어 멤버로 변경하는 것입니다. 읽기 복제본의 전략은 트랜잭션 폴링과 같습니다. 기본 값은 랜덤 코어 숫자에서 폴링하는 것 입니다. 

다중-데이터 센터 클러스터를 실행하는 동안 코어와 읽기 복제본 업스트림 전략은 모두 설정 됩니다. 
읽기 복제본은 또한 트랜잭션 폴링의 영향을 받습니다. 더 자세한 내용은 [멀티-데이터 센터 작동](./multi-data-center/configuration.md)을 참조하십시오.

