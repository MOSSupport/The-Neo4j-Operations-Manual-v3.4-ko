
## 8.1. 메트릭스(Metrics)  

```
이 섹션에서는 Neo4j 메트릭스 출력 기능을 이용해서 다양한 메트릭스를 로그하고 표시하는 방법에 대해 다룹니다.
```

이 섹션에서는 다음에 대해서 다룹니다.:

- [메트릭스 로깅 활성화](./metrics.md)
- [사용가능한 메트릭스](./metrics/reference.md)
	- 범용 메트릭스
	- 인과 관계 클러스터 메트릭스

### 8.1.1. 메트릭스 로깅 활성화

Neo4j는 두 가지 방법으로 리포트를 보고할 수 있습니다. 

- 메트릭스를 [Graphite]("https://graphiteapp.org/")나 Graphite 프로토콜 기반 모니터링 툴에 전송합니다. 
- 메트릭스를 CSV 파일로 보냅니다.
- [프로 메테우스](https://prometheus.io/) 엔드 포인트로 폴링 메트릭을 게시하십시오.

기본적으로 메트릭은 CSV 파일에 메트릭을 보고하도록 활성화되고 구성됩니다.

Neo4j는 다음 데이터베이스 파트 일부로 보내질 수 있습니다. 

```
# Setting for enabling all supported metrics.
metrics.enabled=true

# Setting for enabling all Neo4j specific metrics.
metrics.neo4j.enabled=true

# Setting for exposing metrics about transactions; number of transactions started, committed, etc.
metrics.neo4j.tx.enabled=true

# Setting for exposing metrics about the Neo4j page cache; page faults, evictions, flushes and exceptions, etc.
metrics.neo4j.pagecache.enabled=true

# Setting for exposing metrics about approximately entities are in the database; nodes, relationships, properties, etc.
metrics.neo4j.counts.enabled=true

# Setting for exposing metrics about the network usage of the HA cluster component.
metrics.neo4j.network.enabled=true
```

#### 8.1.1.1. Graphite

Graphite와 통합하기 위해서 아래 내용을 neo4j.conf 설정에 추가합니다.

```
# Enable the Graphite integration. Default is 'false'.
metrics.graphite.enabled=true
# The IP and port of the Graphite server on the format <hostname or IP address>:<port number>.
# The default port number for Graphite is 2003.
metrics.graphite.server=localhost:2003
# How often to send data. Default is 3 seconds.
metrics.graphite.interval=3s
# Prefix for Neo4j metrics on Graphite server.
metrics.prefix=Neo4j_1
```

Neo4j를 시작하고 웹 브라우저를 통해 Graphite에 연결하여 Neo4j 메트릭스를 모니터링 합니다.   


Graphite 서버를 호스트 이름 또는 DNS 항목으로 구성하면 JVM이 호스트 이름을 IP 주소로 확인하고 보안상의 이유로 결과는 무한으로 캐시됩니다. 이 작업은 JVM보안 속성 내 ```networkaddress.cache.ttl``` 가 통제합니다. 더 많은 정보는 [https://docs.oracle.com/javase/8/docs/technotes/guides/net/properties.html]("https://docs.oracle.com/javase/8/docs/technotes/guides/net/properties.html")에서 확인할 수 있습니다.  

#### 8.1.1.2. CSV 파일

메트릭스를 local.CSV 파일에 추가하려면 아래 설정을 neo4j.conf 파일에 추가하십시오.:

```
# Enable the CSV exporter. Default is 'false'.
metrics.csv.enabled=true
# Directory path for output files.
# Default is a "metrics" directory under NEO4J_HOME.
#dbms.directories.metrics='/local/file/system/path'
# How often to store data. Default is 3 seconds.
metrics.csv.interval=3s
```

#### 8.1.1.3. Prometheus

 Prometheus 엔드포인트를 활성화하려면 아래 설정을 neo4j.conf 파일에 추가하십시오.

```
# Enable the Prometheus endpoint. Default is 'false'.
metrics.prometheus.enabled=true
# The IP and port the endpoint will bind to in the format <hostname or IP address>:<port number>.
# The default is localhost:2004.
metrics.prometheus.endpoint=localhost:2004
```

Neo4j가 완전히 시작되면 설정 주소에서 Prometheus 엔드포인트를 사용할 수 있습니다.
