
## 3.8. 트랜잭션 로그

```
이 섹션에서는 Neo4j 트랙잭션 로그 유지 및 회전 정책과 이를 설정하는 법에 대해 살펴봅니다. 
```

트랜잭션 로고는 데이터 베이스의 모든 동작을 기록합니다. 이것은 데이터 베이스를 복구하는 "진리의 근원" 입니다. 트랜잭션 로그는 클러스터 작동과 마찬가지로 증분 백업을 제공할 때 사용합니다. 특정 구성에서는 비어있지 않은 최근 트랜잭션 로그만 유지됩니다. 

**로그 위치**

기본적으로 트랜잭션 로그는 소속된 저장소와 동일한 폴더에 보관됩니다. 트랜잭션 로그가 저장된 디렉토리는 ```dbms.directories.tx_log```에서 설정됩니다. 성능을 최대화하려면 트랜잭션 로그를 전용 장치에 저장되도록 구성하는 것이 좋습니다.

**로그 회전**
로그 회전은 변수 ```dbms.tx_log.rotation.size```을 사용하여 설정됩니다. 기본적으로, 로그는 그 사이즈가 250MB을 넘을 때 회전됩니다. 

**로그 보존**

```dbms.tx_log.rotation.retention_policy```변수를 사용해서 유지되는 트랜잭션 로그 양은 여러 방법으로 제어가 가능합니다. 이 변수는 두 가지 방법으로 설정 가능합니다.:

- ```dbms.tx_log.rotation.retention_policy=<true/false>```

먄약 변수가 ```참```으로 설정되어 있다면, 트랜잭션 로그는 변하지 않습니다. 이 옵션을 사용할 때 서버 하드 드라이브가 가득 차지 않도록 로그 파일을 주기적으로 보관하는 것을 권장합니다. 

만약 변수가 ```거짓```으로 설정되어 있다면, 비어 있지 않은 최신 로그 파일만 남습니다. 이 옵션은 증가하는 백업이 마지막 백업된 후 트랜잭션 로그에 의존하기 때문에, 상업용으로는 권장하지 않습니다. 

- ```dbms.tx_log.rotation.retention_policy=<amount> <type>```


**로그 잘라내기**

트랜잭션 로그 잘라내기는 오래되고 불필요한 트랜잭션 로크 파일을 자동적으로 제거하여 안전을 보장합니다. 트랜잭션 로그는 한 개 이상 파일이 보존 정책에 어긋날 때 잘라낼 수 있습니다. 파일이 제거되려면 다음 두 가지가 필요합니다. 

- 파일은 회전되어 있어야 합니다. 
- 적어도 한 개의 체크 포인트가 최신 로크 파일에 발생해야 합니다. 

더 많은 트랜잭션 로그파일이 발생하는 것은 체크포인트가 자주 일어나지 않거나 오래걸리기 때문일 수 있습니다. 이것은 일시적이며 예상된 로그와 관찰된 로그 파일 수의 간격은 다음 성공적인 체크포인트에서 줄어듭니다. ```dbms.checkpoint.interval.time```와 ```dbms.checkpoint.interval.tx```을 사용하면 체크포인트 간격을 조정할 수 있습니다. 가장 적은 양의 로그 데이터 트랙잭션을 쓰는 것이 목표라면 체크 포인트 절차 자체 속도를 높힐 수 있습니다. 설정 변수 ```dbms.checkpoint.iops.limit``` 는 체크포인트 프로세스에서 사용할 수 있는 IOs를 초 단위로 통제합니다. 변수를 ```-1```로 설정하면, IOPS를 무한으로 사용하기에 빠르게 검사할 수 있습니다. IOPS 한계를 통제하면 트랜잭션 프로세스 처리가 느려질 수 있습니다. 

**표 3.6. 로그 보존을 통제하는 종류**

 | 종류   | 설명                                                         | 예                             |
| ------ | ------------------------------------------------------------ | ------------------------------ |
| 파일   | 가장 최근에 저장된 논리 로그 파일의 수                       | "10 파일"                      |
| 사이즈 | 로그 파일을 사용하는 최대 디스크 크기                        | "300M 사이즈" 또는 "1G 사이즈" |
| txs    | 유지할 트랜잭션 개수                                         | "250k txs" 또는 "5M txs"       |
| 시     | 현 시간을부터 N시간에 내 커밋 된 트랜잭션이 포함된 로그 유지 | "10 시간"                      |
| 일     | 현재 시간으로부터 몇 일내에 커밋 된 트랜잭션이 포함된 로그 유지 | "50 일"                        |

**예 3.8. 로그 보존 정책 설정**

이 예에서 로그 보존을 설정하는 여러가지 방법을 살펴봅니다. 

+ 트랜잭션 로그를 무기한으로 유지
```dbms.tx_log.rotation.retention_policy=true```

+ 비어있지 않은 최신 로그만 유지
```dbms.tx_log.rotation.retention_policy=false```

+ 논리 로그의 30일 이내 트랜잭션을 유지
```dbms.tx_log.rotation.retention_policy=30 days```

+ 논리 로그에 500 000을 가진 최근 항목 유지  
```dbms.tx_log.rotation.retention_policy=500k txs```