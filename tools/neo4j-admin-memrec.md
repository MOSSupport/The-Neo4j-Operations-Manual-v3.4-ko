## 10.5. 메모리 권장사항

```
이 절에서는 Neo4j Admin의 memrec 명령어에 대해서 다룹니다.
```

```neo4j-admin```의 ```memrec``` 명령어를 사용하여, Neo4j 메모리 매개 변수를 구성하는 방법에 대한 권장 사항을 얻습니다. 

```
neo4j-admin memrec [--memory=<memory dedicated to Neo4j>] [--database=<name>]
```

권장 사항은 neo4j.conf에 바로 복사, 붙여넣기 가능한 포맷으로 제공됩니다.

**옵션**

| 옵션       | 기본 값            | 설명                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| --memory   | 기계의 메모리 용량 | Neo4j에 할당될 메모리의 크기. 유효한 단위: k, K, m, M, g, G. |
| --database | graph.db           | 데이터 베이스 이름. 이 옵션은 Lucene 인덱스와 데이터 베이스의 데이터 용량 및 원시 인덱스에 대한 숫자를 생성합니다. 더욱 상세한 메모리 분석에 대한 입력 값으로 사용될 수 있습니다. |


**고려 사항**

```neo4j-admin memrec``` 명령어는 제공된 메모리를 기반으로 Neo4j 메모리 설정의 유효한 시작점을 계산합니다. 사용 케이스마다 권장 값을 조정해야할 수도 있습니다. 사용 케이스의 특정 조건은 이러한 값의 조정을 보증합니다. Neo4j의 메모리 설정에 대한 설명은 [섹션9.1 메모리 구성](../performance/memory-configuration.md)을 참조하십시오.

**예제 10.4. ```neo4j-admin```의 ```memrec``` 명령어 사용**

다음 예제는 ```neo4j-admin memrec```이 16G 메모리에서 메모리 추천을 제공하는 방법을 보여줍니다:  

```
$neo4j-home> bin/neo4j-admin memrec --memory=16g

...
...
...
# Based on the above, the following memory settings are recommended:
dbms.memory.heap.initial_size=5g
dbms.memory.heap.max_size=5g
dbms.memory.pagecache.size=7g
```
 