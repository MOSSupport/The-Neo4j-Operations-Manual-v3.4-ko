
## 9.3. 가비지 컬렉터 튜닝 


```
이 절에서는 자바 가상 머신의 가비지 컬렉터가 Neo4j 성능에 미치는 영향을 알아봅니다.
```

힙은 올드/영 제너레이션(generation)으로 구분됩니다. 새로운 개체(objects)는 영 제너레이션에서 할당되고 후에 오래 사용(사용중인 경우)되면 올드 제너레이션으로 이동합니다. 한 제너레이션이 채워지면 가비지 켈렉터가 수집(collection)되며, 그동안 해당 프로세스의 다른 모든 스레드(thread)는 정지됩니다. 정지 시간은 실시간 개체 집합과 관련있지만 영 제너레이션 크기와는 무관하므로 가비지 수집이 빠릅니다. 올드 제너레이션의 정지 시간은 힙 크기와 비례합니다. 따라서 힙은 트랜잭션 및 쿼리 상태가 올드 제너레이션에 적용되지 않도록 크기를 조정해야합니다.


힙 크기는 neo4j.conf 파일의 ```dbms.memory.heap.max_size``` (MB 단위)에서 설정합니다. 힙의 초기 크기는 ```dbms.memory.heap.initial_size``` 설정 값, 또는 ```-Xms\$ \{mem\}``` 플래그로 지정되거나, 지정되지 않은 경우 JVM 자체에 경험에 의해 선택됩니다. JVM은 필요에 따라 자동으로 힙을 최대로 확장합니다. 힙을 늘리려면 가비지 수집 주기가 필요합니다. 최초 힙 크기와 최대 힙 크기를 같은 값으로 설정하는 것이 좋습니다. 이렇게 하면 가비지 컬렉터가 힙을 늘릴 때 발생하는 정지 현상을 피할 수 있습니다.    

힙의 올드와 영 제너레이션 간 크기 비율은 ```-XX:NewRatio=N``` 플래그로 제어됩니다. N 기본 값은 2와 8 사이 값입니다. 비율 2는 올드 제너레이션 크기를 영 제너레이션 크기로 나눈값이 2라는 의미입니다. 즉, 힙 메모리의 3분의 2는 올드 제너레이션이라는 의미입니다. 비율 3은 힙 메모리의 4분의 3은 올드 제너레이션을 의미하며, 비율 1은 두 제너레이션의 크기가 같다는 의미입니다.비율 1은 매우 공격적이지만 트랜잭션이 많은 데이터를 변경하는 경우 필요합니다. 큰 결과-셋의 정렬과 같이 많은 양의 데이터를 유지하는 Cypher 쿼리를 실행하는 경우, 영 제너레이션 값이 커야합니다.  

영 제너레이션의 크기가 너무 작으면 수명이 짧은 개체들이 너무 빨리 올드 제너레이션으로 옮겨집니다. 이를 조기 프로모션(premature promotion) 이라고 하며 올드 제너레이션 가비지 수집 주기 빈도를 늘려서 데이터베이스 속도를 저하시킵니다. 영 제너레이션의 너무 크면 가비지 컬렉터는 영 제너레이션에서 올드 제너레이션으로 촉진하는 모든 개체를 수용할 공간이 충분하지 않다고 결정할 수 있습니다. 이 경우에 영 제너레이션의 가비지 콜렉션 사이클이 올드 제너레이션의 가비지 콜렉션 사이클로 바뀌어 데이터베이스가 다시 느려집니다. 동시 스레드를 많이 실행하면 주어진 시간 내에 할당이 더 많이 발생할 수 있으며, 특히 영 제너레이션에 대한 압박이 증가합니다.  

JVM의 압축 OOP기능을 사용하면 객체 참조를 32비트만 사용하도록 압축할 수 있습니다. 이렇게하면, 메모리를 많이 절약하지만, 32GB 이상 힙에서는 사용할 수 없습니다. 따라서 32GB를 초과하는 힙 크기의 증가가 중요하지 않다면(64GB 또는 그 이상), 작거나 음수일 수 있습니다.

Neo4j는 자바 프로세스 수명주기 동안 올드 제너레이션에는 긴-수명을 가진 개체들이 많이 있습니다. 이를 효율적으로 처리하고 가비지 수집 정지를 없애려면 동시 가비지 컬렉터를 사용하는 것이 좋습니다.  
 
특정 가비지 수집 알고리즘의 튜닝은 JVM 버전 및 작업 부하에 따라 다릅니다. 며칠 혹은 몇 주 동안 실제 부하에서 가비지 수집 설정을 테스트합니다. 힙 분활과 같은 문제는 알아내기 어렵습니다.    

좋은 성능을 얻으려면 다음 사항을 고려해야합니다.

* JVM이 가비지 수집에 너무 많은 시간을 소비하지 않는지 확인합니다. 작업 목표는 충분한 힙 공간을 확보하여 헤비/피크 로드시 GC-트레싱이 발생하지 않는 것입니다. GC-트레싱이 실행되면 성능이 두 배 정도 떨어집니다. 또한 너무 큰 힙을 사용하면 성능이 저하되므로 힙 사이즈를 다양하게 적용합니다.   
* 동시(concurrent) 가비지 컬렉터를 사용합니다. -XX:+ UseG1GC 옵션이 대부분의 경우 잘 작동합니다.  
  - Neo4j JVM은 트랜잭션 상태와 쿼리 처리 및 가비지 컬렉터에 충분한 힙 메모리를 필요로 합니다. 힙 메모리 요구량은 작업 부하에 따라 다르므로, 1GB에서 최대 32GB으로 힙 메모리를 설정하는 것이 일반적입니다.

* JVM을 -server 옵션 및 충분한 크기 힙으로 실행합니다.
  - 전용 서버의 운영 체제는 대개 1 - 2GB의 메모리로 실행되지만, 실제 시스템의 물리 메모리가 많을수록 운영 체제는 더 많은 메모리가 필요합니다.   

다음 속성을 편집합니다:  

표 9.1. neo4j.conf JVM 튜닝 속성  

| 속성 명 | 의 미 |
|:---|:---|
| ```dbms.memory.heap.initial_size``` | 초기 힙 크기(MB 단위) |
| ```dbms.memory.heap.max_size``` | 최대 힙 크기(MB 단위) |
| ```dbms.jvm.additional``` | 추가적인 JVM 매개변수 |
