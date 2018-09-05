
## 7.3. SSL 프레임워크 

```
이 섹션에서는 Neo4j 채널의 보안 통신을 위한 SSL/TLS 통합에 대해 설명합니다. 
```


Neo4j는 기본 SSL/TLS 기술을 이용해서 통신 채널 보안을 지원합니다. 

이 섹션에서는 다음 내용을 다룹니다:

- 소개 
- 전제조건 
  - Java 설정  
  - 자격  
- SSL 정책 정의  
- SSL 정책 지원  
- 기존 SSL 시스템  
- 용어 

### 7.3.1. 소개 

SSL 지원은 일련의 매개 변수와 함께 SSL 인증서로 구성된 SSL 정책을 작성하여 활성화됩니다. 그 후, 정책은 보안을 설정하려는 통신 채널에 적용됩니다. 과정은 다음 섹션에 설명되어 있습니다. 

Neo4j가 SSL 정책 정의 없이 시작된다면, 이것은 기존 *SSL system*이 기본으로 사용됩니다. 
추가적으로, 인증서가 설치되어 있지 않다면 Neo4j 절차는 자동적으로 자체-서명된 증명서 및 기본 디렉토리 내 암호 키를 생성할 것 입니다. 자세한 내용은 [섹션 7.3.5, 기존 SSL 시스템](./ssl-framework.md)를 참조하십시오. 

### 7.3.2. 전제조건 

##### Java 구성 

Neo4j SSL 프레임 워크는 특히 Java 플랫폼에서 활성화할 강력한 암호를 사용합니다. 참조: <http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html>

##### 증명 

이 섹션의 지침에서는 필요한 인증서(./ssl-framework.md)를 이미 취득했다고 가정합니다.

모든 인증서는 ```PEM```형식으로 있어야하며, 하나의 파일로 결합할 수 있습니다. 비밀 키도 ```PEM``` 형식으로 있어야 합니다. 다중-호스트 및 와일드카드 증명서가 지원됩니다. 이런 인증서는 Neo4j가 다른 인터페이스에 바인드하는 다중 커넥터로 구성되었을 때 필요합니다. 

### 7.3.3. SSL 정책 정의 

SSL 정책은 다음 형식으로 매개 변수를 할당하여 환경 설정 합니다. 
```dbms.ssl.policy.<policy-name>.<setting-suffix>```

```policy-name```은 정의하려는 정책 이름을 뜻합니다. 

```setting-suffix```의 기본 유효 값은 다음과 같습니다:

| 접미사 설정          | 설명                                                         | 기본 값                    |
| -------------------- | ------------------------------------------------------------ | -------------------------- |
| `base_directory`     | 암호 객체가 기본적으로 검색되는 기본 디렉토리입니다.         | 의무적으로 설정해야합니다. |
| `private_key`        | 이 인스턴스를 인증/보호하는 개인 키 입니다.                  | *private.key*              |
| `public_certificate` | 인증 기관 (CA)이 서명한 개인 키와 일치하는 공인인증서입니다. | *public.crt*               |
| `trusted_dir`        | 신뢰할 수있는 인증서로 채워지는 디렉토리입니다. 구성된 경우 절대 경로 여야합니다.  최소한 이 노드의 공개된 인증서 (public.crt)를 포함해야합니다. | *신뢰됨*                   |
| `revoked_dir`        | 인증서 해지 목록 (CRL)이 채워진 디렉토리입니다. 구성된 경우 절대 경로여야 합니다. | *폐지됨*                   |


공인인증서를 복제해야 합니다. 하나의 복사본은 ```base_directory```에 저장되고 다른 복사본은 ```trusted_dir```에 저장됩니다. 

유일한 필수 설정은 ```dbms.ssl.policy. <policy-name> .base_directory```에서 정의한 기본 디렉토리입니다. 기본 디렉토리를 정의해서 Neo4j가 ```<policy-name>``` 이름으로 정책을 정의하도록 합니다. 이 정책에 다른 설정이 정의되어 있지 않다면, Neo4j는 기본적으로 *trusted*, *revoked* 하위 디렉토리뿐만 아니라 기본디렉토리 내 개인 키와 인증서 파일을 찾을 것 입니다. 다른 경로가 선호되는 경우, 모든 기본 값은 대체할 수 있습니다. 

보안상 이유로, Neo4j는 이런 디렉토리를 자동으로 생성하지 않습니다. 따라서 SSL 정책 생성은 수동으로 시스템을 설정하기 위해 적절한 파일을 필요로 합니다. 인증서 파일, 개인 키 및 디렉토리는 필수적으로 존재해야함을 기억하십시오. Neo4j 사용자만 읽을 수 있도록 개인 키에 사용 권한이 바르게 설정되어 있는지 확인하십시오.

**예 7.26. 정책 정의**

이 예제에서는 정책 ```example_policy```에 대한 구성을 정의하고 작성합니다. 가장 간단한 구성이 가능하므로 *neo4j.conf*에서 이 기본 디렉토리를 정의합니다.

```
dbms.ssl.policy.example_policy.base_directory=certificates/example_policy
```

그 후, 필수 디렉토리를 생성합니다. 

```
$neo4j-home> mkdir certificates/example_policy
$neo4j-home> mkdir certificates/example_policy/trusted
$neo4j-home> mkdir certificates/example_policy/revoked
```

마지막으로, *private.key*와 *public.crt* 파일을 기본 디렉토리에 옮기고, *trusted* 디렉토리에 *public.crt* 사본을 옮깁니다. 

```
$neo4j-home> cp /path/to/certs/private.key certificates/example_policy
$neo4j-home> cp /path/to/certs/public.crt certificates/example_policy
$neo4j-home> cp /path/to/certs/public.crt certificates/example_policy/trusted
```

다음 리스트 결과를 얻습니다:

```
$neo4j-home> ls certificates/example_policy
-r-------- ... private.key
-rw-r--r-- ... public.crt
drwxr-xr-x ... revoked
drwxr-xr-x ... trusted
$neo4j-home> ls certificates/example_policy/trusted
-rw-r--r-- ... public.crt
```

또한, 정책에 다음 매개변수를 구성할 수 있습니다.

| 접미사 설정            | 설명                                                         | 기본 값                                 |
| ---------------------- | ------------------------------------------------------------ | --------------------------------------- |
| `client_auth`          | 클라이언트 인증 여부. 이것을 ```REQUIRE```로 설정하면 서버 상호 인증을 효과적으로 수행 할 수 있습니다. 이 설정에서 가능한 값은  `NONE`, `OPTIONAL`, 또는 `REQUIRE`입니다. | `REQUIRE`                               |
| `ciphers`              | 암호 협상 중에 허용되는 암호 목록입니다.                     | Java 플랫폼의 디폴트의 기본 encode 방식 |
| `tls_versions`         | 지원하는 TLS/SSL 프로토콜 버전 리스트입니다.                 | `TLSv1.2`                               |
| `allow_key_generation` | 이 매개변수는 기본값  ```false```로 유지하는 것이 좋습니다. ```true```로 설정하면 시작할 때,  *.key / .crt* 파일의 자동 생성이 활성화됩니다. 필요한 디렉토리 구조도 자동으로 생성됩니다. | `false`                                 |
| `trust_all`            | 이 매개변수는 기본값  ```false```로 유지하는 것이 좋습니다. ```true```로 설정하면 "다른 사람 신뢰"를 의미하며 인증을 비활성화합니다. | `false`                                 |

Neo4j와 Java 플랫폼을 결합하면, 강력한 프로토콜 및 암호 결합을 제공합니다. 

### 7.3.4. SSL 정책 적용하기 

다른 통신 채널은 아래 환경 설정을 이용하여 각각 독립적으로 보호됩니다. 

```bolt.ssl_policy```

Bolt 클라이언트 통신에 이용되는 정책입니다. 

```https.ssl_policy```

HTTP 클라이언트 통신에 이용되는 정책입니다.

```causal_clustering.ssl_policy```

내부-클러스터 통신에 이용되는 정책입니다.

```dbms.backup.ssl_policy```

백업 트래픽 암호화에 이용되는 정책입니다. 

**예 7.27. SSL 정책 적용하기**

아래 나열된 정책을 설정했다고 가정합니다:

- 클라이언트 트래픽 암호화에 이용되는 정책은 ```client_policy```입니다. 
- 내부-클러스터 암호화에 이용되는 정책은 ```client_policy```입니다.
- 백업 트래픽 암호화에 이용되는 정책은 ```backup_policy```입니다. 

다음 예는 위 정책을 따라 암호화를 구성합니다.

```
bolt.ssl_policy=client_policy
https.ssl_policy=client_policy
causal_clustering.ssl_policy=cluster_policy
dbms.backup.ssl_policy=backup_policy
```

### 7.3.5. SSL 레거시(legacy) 시스템 

SSL 지원은 *legacy SSL system*을 이용해서 Bolt와 HTTPS에서 이용할 수도 있습니다. 
레거시 시스템은 향후 어느 시점에서 사용이 거절될 수 있으므로, 기본 SSL 설정 사용을 권장합니다.

Neo4j로 SSL 레거시 시스템을 설정하기 위해서 개인 키와 증명서 파일 이름을 각각 *neo4j.key* 와 *neo4j.cert*로 설정해야 합니다. 키는 암호화되지 않아야합니다. 파일은 할당된 디렉토리에 옮기십시오. 기본값은 *certificates* 디렉토리이며 *neo4j-home* 디렉토리에 있습니다. 디렉토리는 *neo4j.conf*. 내에[dbms.directories.certificates](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/)을 이용해서 명시할 수도 있습니다. 

인증서를 설치하지 않고 시작하면, Neo4j 프로세스는 자체 서명 된 SSL 인증서와 개인 키를 기본 디렉토리에 자동으로 생성합니다. 자체 서명된 SSL 인증서의 자동-생성 기능은 Neo4j가 다수의 다른 IP주소로 바인드 된 [커넥터](/configuration/connectors.md)로 구성되면 사용할 수 없습니다. 다수의 IP 주소를 사용할 경우, 수동으로 인증서를 구성하고 다수-호스트 및 와일트 카드 인증서를 대신 사용하십시오. 

특정 레거시 정책 이름 하에 SSL 프레임워크에서 레거시 정책은 이용가능하지만, 프레임워크 전체를 사용할 수는 없습니다. 이것은 기본적으로 다음 SSL 정책 정의와 같습니다.

```
bolt.ssl_policy=legacy
https.ssl_policy=legacy

dbms.ssl.policy.legacy.base_directory=certificates
dbms.ssl.policy.legacy.private_key=neo4j.key
dbms.ssl.policy.legacy.public_certificate=neo4j.cert
dbms.ssl.policy.legacy.allow_key_generation=true
dbms.ssl.policy.legacy.client_auth=NONE
```

HTTPS와 Bolt 서버는 클라이언트 인증은 지원하지 않습니다.(즉. *mutual authentication*)그 결과로 ```client_auth```는 HTTPS와 Bolt서버를 새로운 ssl정책으로 마이그레이션 동안 ```client_auth = NONE```를 이용해서 명시적으로 종료해야 합니다. 클라이언트 인증이 비활성화되어 있다면, ```trusted_dir```, ```revoked_dir``` 또는 ```trust_all```에 할당된 값은 클라이언트 인증에서 사용하므로 무시됩니다. 

```tls_versions```와 ```ciphers``` 설정은 HTTPS와 Bolt 서버에서 이용가능합니다. ```legacy``` 정책은 자바 플랫폼이 지원하는 TLS 버전과 암호 세트로 기본 설정됩니다.

### 7.3.6. 용어  

아래 용어는 Neo4j에서 SSL가 지원하는 것과 관련있습니다. 

**인증 기관 (CA)**

전자 문서(디지털 개체의 신원을 확인)를 발행하는 신뢰가능한 객체입니다. 일반적으로 이 용어는 세계적으로 인정하는 CA를 나타내지만, 신뢰가능한 조직 내부 CA도 포함합니다. 전자 문서는 디지털 인증서 입니다. 이들은 보안 통신에서 필수적인 요소이며, 공용 키 인프라에서 중요한 역할을 합니다. 

**인증서 해지 목록(CRL)**

인증서가 손상된 경우 해당 인증서를 해지 할 수 있습니다. 이것은 폐지된 인증서 설명 리스트(한개 또는 여러 파일에 위치)를 활용하여 수행됩니다. CRL는 항상 일치하는 인증서를 발행하는 CA에서 발행됩니다. 

**암호**

암호화 또는 암호 해독을 수행하기 위한 알고리즘입니다. 일반적으로 Neo4j 통신을 구현할 때, Java 플랫폼 일부로 포함된 암호를 암시적으로 사용합니다. SSL 프레임워크 구성에서 허용된 암호를 명시적으로 선언할 수도 있습니다.

**통신 채널**

Neo4j 데이터베이스 통신 방법입니다. 이용 가능한 채널은 다음과 같습니다:

- Bolt 클라이언트 트래픽 
- HTTPS 클라이언트 트래픽 
- 내부-클러스터 통신 
- 백업 트래픽

**암호 객체**

이슈를 나타내는 용어 개인 키, 인증서 및 CRL입니다. 

**매개변수 설정**

*neo4j.conf* 내 특정 ssl 정책을 위해 정의된 매개 변수 입니다. 

**인증서**

SSL 인증서는 신뢰할 수 있는 인증기관(CA)에서 발행됩니다. 공개 키는 특정 수신자를 위한 메시지를 암호화하기 위해 누구든지 획득하고 사용할 수 있습니다. 인증서는 파일 내에 *<file name>.crt*이름으로 저장됩니다. 이것은 공개 키라고도합니다.

**SSL**

SSL은 *Secure Sockets Layer*의 두문자어며 TLS의 전문자입니다. 일반적으로 SSL / TLS는 SSL이라고합니다. 그러나 현대적이고 안전한 버전은 TLS이며, Neo4j 기본 값이기도합니다.

**SSL 정책**

Neo4j내 SSL 정책은 디지털 인증서와 *neo4j.conf*에 정의된 매개변수 설정으로 구성됩니다. 

**개인 키**

개인 키는 특정 수신자만 암호화된 메시지를 해독할 수 있도록합니다. 일반적으로 개인 키는 *<file name>.key*이름의 파일에 저장됩니다. 암호화 통신의 무결성을 보장하기위해 개인 키를 보호하는것이 중요합니다. 

**공용 키 인프라(PKI)**

디지털 인증서를 생성, 관리, 배포, 사용, 저장 및 해지하고 공개 키 암호화를 관리할 때 필요한 역할, 정책 및 절차입니다.

**공용 키**

공개 키는 특정 수신자를 위한 메시지를 암호화하기 위해 누구든지 사용할 수 있습니다. 이것은 인증서라고도 합니다. 

**TLS 버전**

TLS 프로토콜의 버전 입니다. 

**TLS 프로토콜**

컴퓨터 네트워크를 통해 통신 보안을 제공하는 암호화 프로토콜입니다. TLS (Transport Layer Security) 프로토콜과 이전 버전 SSL(Secure Sockets Layer) 프로토콜은 모두 "SSL"이라고 합니다.
 
 