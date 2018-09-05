
## 4. 클러스터링 `Enterprise Edition`

```
이 챕터에서는 Neo4j 인과관계 클러스터링 특징에 대해 다룹니다. 
```
 
### 설치 및 환경 설정

이 파트는 이론적 배경 및 구조에 대한 설명으로 시작합니다. 그 후, 환경설정 세부 사항 및 인과관계 클러스터를 설정과 작동 방법에 대해 다룹니다. 

- [소개](/clustering/introduction.md) — 인과관계 클러스터 구조 개요.
- [생활 주기](/clustering/lifecycle.md) — 클러스터 주기에 대한 설명.
- [새로운 클러스터 생성](/clustering/setup-new-cluster.md) — 새로운 인과관계 클러스터 생성 방법.
- [클러스터 전송](/clustering/seed-cluster.md) — 이미 있는 데이터베이스에서 인과관계 클러스터 생성 										방법 
- [백업 계획](/clustering/backup-planning.md) — 인과관계 클러스터 백업 설계 고려사항. 
- [인프라-클러스터 암호화](/clustering/intra-cluster-encryption.md) — 클러스터 통신 안전화 방법. 

Neo4j 인과관계 클러스터를 업그레이드 하는 방법은 [인과관계 클러스터 업그레이드](/upgrade/causal-cluster.md)를 참조하십시오.  

Neo4j 인과관계 클러스터를 모니터링할 수 있는 기능에 관련된 요약은 [챕터 8, 모니터링](/monitoring.md) (특히, [인과관계 클러스터 모니터](/monitoring/causal-cluster.md))을 참조하면 됩니다. 

### 멀티-데이터 센터 

이 부분은 다중 데이터 센터 작업에 쓰이는 고급 배포 및 구성 옵션입니다. 


- [멀티-데이터 센터](/clustering/multi-data-center.md) — 멀티-데이터 센터 부분 개요. 
  - [멀티-데이터 센터 작동 허가 — 멀티-데이터 센터 작동 허가와 관련된 정보.

- [멀티-데이터 센터 설계](/clustering/multi-data-center/design.md) — 멀티-데이터 센터 배포 패턴.

- [멀티-데이터 센터 작동](/clustering/multi-data-center/configuration.md) — 멀티-데이터 센터 배포 설정 옵션.

- [멀티-데이터 센터 로드 발렌싱](/clustering/multi-data-center/load-balancing.md) — 
클라이언트 응용 프로그램이 다중 데이터 센터 토폴로지를 인식하도록하는 구성 옵션.

- [데이터 센터 재해 복구](/clustering/multi-data-center/disaster-recovery.md) — 데이터 센터 손실 후 클러스터가 작동하도록 회복시키는 방법. 

### 멀티-클러스터링 설계 및 환경설정 

이 부분은 멀티-클러스터링 배포 생성 및 환경설정 관련 내용으로 구성됩니다. 하나의 검색 서비스에서 연결되고 보여지는 독립 그래프 저장소를 가진 여러개의 개별 클러스터를 설치하는 방법을 보여줍니다. 

- [멀티-클러스터링](/clustering/multi-clustering.md) — 멀티-클러스터링 섹션 개요.
- [소개](/clustering/multi-clustering/introduction.md) — 멀티-클러스터링 기능 소개.
- [멀티-클러스터 환경설정](/clustering/multi-clustering/config.md) — 멀티-클러스터 환경설정 지침. 

Neo4j 멀티-클러스터를 모니터링할 때 이용하는 기능 요약 내용은 [섹션 8.4.3, 멀티-클러스터 모니터링](/monitoring/causal-cluster/multi-cluster.md)를 참조하십시오. 

### 참조 설정

- [참조 설정](/clustering/settings.md) — 인과관계 클러스터 설정에서 가장 중요한 것 요약.

단일 컴퓨터에서 로컬로 테스트 클러스터 설정 방법은 [섹션 B.1, 로컬 인과관계 클러스터 설정](/tutorial/local-causal-cluster.md)을 참조하면 됩니다. 