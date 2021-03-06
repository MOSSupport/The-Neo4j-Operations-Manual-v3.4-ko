
## 4.6. 클러스터 내 암호화 

```
이 챕터에서는 서버 인스턴스 사이 클러스터 통신을 보호하는 방법에 대해서 설명합니다. 
```

### 4.6.1. 소개 

클러스터 통신에서 보안 솔루션은 기본 SSL/TLS 기술(공동으로 SSL이라고도 함)을 기반으로 합니다. 
사실상 암호화는 보안의 일부이며 인증 및 무결성과는 다른 토대가 됩니다. 보안 솔루션은 인증 요구 사항과 함께 배포되는 주요 인프라를 기반으로 합니다. 

플랫폼 내 SSL 지원은 [섹션 7.3, SSL 프레임워크](/security/ssl-framework.md)에서 확인할 수 있습니다. 이 섹션에서는 클러스터 보안과 관련된 내용을 다룹니다. 

SSL에서 엔드포인트는 [공용 키 인프라 (*PKI*)](/security/ssl-framework.md)에서 관리하는 인증서를 사용해서 증명할 수 있습니다. 
 
안전 키 관리 인프라 배포는 이 메뉴얼 범위를 넘어가는 것을 기억해야하고, 숙련된 보안 전문가가 처리해야 합니다. 아래 배치는 예시는 참고 목적으로 만 사용됩니다.

### 4.6.2. 배포 예시 

아래 단계는 배포 예시를 생성하며, 각 단계는 아래와 같이 확장됩니다. 

- [암호 객체](/security/ssl-framework.md) 생성 및 설치  
- SSL 정책 생성 
- SSL 정책으로 인과관계 클러스터링 환경설정 
- 클러스터의 보안 작동 입증 

#### 4.6.2.1. 암호 객체 생성 및 설치 

암호 객체 생성은 이 메뉴얼의 범위를 벗어납니다. 이것은 일반적으로 조직 내에 [증명 권한 (*CA*)](/security/ssl-framework.md)와 PKI를 필요로하며, 이곳에서 사용할 수 있어야 합니다. PKI와 관련된 이 메뉴얼의 정보는 주로 설명이 목적입니다. 

증명서 및 보안 키를 얻으면 이들은 각 서버에 설치할 수 있습니다. 각 서버는 그 자체 인증서, CA의 서명 및 일치하는 보안 키를 가집니다. CA의 증명서는 ```trusted```(신뢰할 수 있는) 디렉토리에 설치되기 떄문에 CA가 서명한 모든 증명서도 신뢰할 수 있습니다. 이제 서버가 다른 서버와 신뢰를 형성할 수 있다는 것을 의미합니다. 

이 예에서는 상호 인증 설정을 배포하기 때문에 채널의 양쪽 끝을 인증해야합니다. 상호 인증을 활성화 하려면 SSL 정책은 ```REQUIRE```(기본 설정 값)로 설정된 ```client_auth```를 포함해야 합니다. 서버는 기본적으로 자신을 인증해야하므로 해당하는 서버 설정이 없습니다.

Neo4j는 자체 서명 인증서를 생성할 수 있지만 이를 사용하는 배포는 일반적으로 특별한 경우로 간주됩니다. 일부 상황에서는 의미가 있으며, 인스턴스간 자체 생성 인증서를 공유하여 상호 인증을 설정할 수도 있습니다. 

특정 서버에 대한 인증서가 손상되면 ```revoked``` 디렉토리 내에 인증서 해지 목록 [*CRL*](/security/ssl-framework.md)을 설치하여 인증서를 폐기할 수 있습니다. 또한, 새로운 CA를 이용해서 재배포할 수도 있습니다. 만일의 사태에 대비하여, 클러스터 전체에 별도의 중간 CA가 있어야하며, 필요한 경우 클러스터 전체를 대체 할 수 있습니다. 이 방법은 해지를 처리하고 증식을 보장하는 것보다 훨씬 쉬울 것입니다.

**예 4.5. 암호 객체 생성 및 설치**

이 예제에서 우리는 SSL 정책 ```cluster``` 이름을 정하도록 가정하며, 개인 키와 인증서 파일의 이름은 각각 *private.key*와 *public.crt* 입니다. 다른 이름을 사용할 경우, 키 및 인증 이름/위치 정책 설정을 무시할 수 있습니다. 이 서버의 기본 설정을 사용할 때 우리는 적당한 디렉토리 구조를 생성하고 증명서를 설치할 수 있습니다. 

```
$neo4j-home> mkdir certificates/cluster
$neo4j-home> mkdir certificates/cluster/trusted
$neo4j-home> mkdir certificates/cluster/revoked

$neo4j-home> cp $some-dir/private.key certificates/cluster
$neo4j-home> cp $some-dir/public.crt certificates/cluster
```

#### 4.6.2.2. SSL 정책 생성 

SSL 정책은 설치된 암호 객체를 활용해서 추가적으로 변수를 구성할 수 있습니다. 우리는 환경설정에서 아래 변수를 사용합니다. 

**표 4.1. 설정 예시**

| 접미사 설정        | 값                                          | 설명                                                         |
| ------------------ | ------------------------------------------- | ------------------------------------------------------------ |
| ```client_auth```  | ```REQUIRE```                               | 이것을 ```REQUIRE```에 설정하면 서버의 상호 인증을 효율적으로 활성화시킵니다. |
| ```ciphers```      | ```TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384``` | 하나의 특정한 강력한 암호를 사용하고 어떤 암호가 선택되는지에 대한 의심을 제거 할 수 있습니다. 위의 암호는 일반적으로 바람직한 PFS(Perfect Forward Secrecy)를 제공합니다. 또한 대칭 암호화에 AES(Advanced Encryption Standard)를 사용하는 하드웨어 가속을 지원하므로 성능에 거의 영향을 주지 않습니다. |
| ```tls_versions``` | ```TLSv1.2```                               | 전체 클러스터를 제어하기 때문에 하위 호환성 문제없이 최신 TLS 표준을 적용 할 수 있습니다. 알려진 보안 취약성은 없으며 키 교환 등에 가장 최신 알고리즘을 사용합니다. |

아래 예에서 우리는 클러스터에서 사용할 SSL 정책 및 설정을 생성합니다. 

**예 4.6. SSL 정책 생성**

이 예에서 우리는 각 예시 별로 디렉토리 구조가 생성되었고, 인증 파일이 설치되었다고 가정합니다. 이제 ```cluster```라고 불리느 SSL 클러스터 정책을 생성하려합니다. 아래 변수를 정의해서 이 작업을 수행합니다. 

```
dbms.ssl.policy.cluster.base_directory=certificates/cluster
```

정책의 이름이 ```cluster```이므로 ```dbms.ssl.policy.cluster.*``` 그룹 아래에서 이 설정을 구성 할 수 있습니다. *neo4j.conf* 파일에 아래 내용을 추가합니다:

```
dbms.ssl.policy.cluster.tls_versions=TLSv1.2
dbms.ssl.policy.cluster.ciphers=TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
dbms.ssl.policy.cluster.client_auth=REQUIRE
```
 
정책은 모든 서버에 동일하게 설정하는 것을 기억하십시오. 동일한 증명서와 보안 키를 공유하지 않기 때문에 설치된 실제 암호 객체는 일반적으로 다를 것 입니다. 그러나 신뢰할 수있는 CA 인증서는 공유됩니다.

#### 4.6.2.3. SSL 정책으로 인과관계 클러스터 환경 설정 

일반적으로, 인과관계 클러스터에서 SSL 정책은 효력이 없기에 통신은 복호화 됩니다. ```causal_clustering.ssl_policy```를 이전에 생성한 SSL 정책 이름으로 변경하여 권한을 사용할 수 있습니다. 

**예 4.7. SSL 정책으로 인과관계 클러스터링 환경설정**

이 예에서는 이전 두 예제의 작업을 수행했다고 가정합니다. 이제 우리는`cluster`라는 이름의 SSL 정책을 사용하도록 클러스터를 설정합니다.

```
causal_clustering.ssl_policy=cluster
```

인스턴스 사이에서 통신한 모든 사용자 데이터는 보호됩니다. 올바르게 설정되지 않은 인스턴스는 다른 인스턴스와 통신 할 수 없습니다.

#### 4.6.2.4. 클러스터의 보안 작동 활성화   

모든 것이 의도한대로 안전한지 확인하려면 오픈 소스 평가 도구인 ```nmap``` 또는 ```OpenSSL```과 같은 외부 도구를 이용해 유효성을 검사하는 것이 좋습니다.

**예 4.8. 클러스터의 보안 작동 활성화**

이 예에서 우리는 클러스터의 보안 작동을 활성화하기 위해 ```nmap```도구를 사용합니다. 수행 할 간단한 테스트는 아래 명령어를 사용하는 암호 열거입니다.

```
nmap --script ssl-enum-ciphers -p <port> <hostname>
```

호스트 명과 포트는 설정에 따라 조정해야 합니다. 이는 TLS가 실제로 활성화되어 있고 의도된 암호 스위트만 사용가능함을 증명할 수 있습니다. 모든 서버 및 모든 해당 포트를 테스트해야합니다.


테스트 목적으로 신뢰할 수없는 인증서가 있는 별도 Neo4j 테스트 인스턴스를 활용할 수도 있습니다. 이 테스트의 예상 결과는 테스트 서버가 사용자 데이터 복제에 참여할 수 없다는 것 입니다. 디버그 로그는 일반적으로 SSL 또는 인증-관련 예외처리를 출력해서 이슈를 나타냅니다. 