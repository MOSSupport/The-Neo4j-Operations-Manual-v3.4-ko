
## 7.1.1. 소개 

```
이 섹션에서는 Neo4j의 인증 및 권한 부여에 대한 개요를 제공합니다. 
```

Neo4j 보안은 인증 및 권한 부여에 의해 통제합니다. 인증은 인증된 사용자가 특정 작업 수행 여부를 확인하는 것과 관련된 사용자가 사용자임을 주장하는 프로세스입니다.

권한 부여는 역할-기반 접근 통제를 사용해서 통제합니다.(RBAC-role-based access control)
Neo4j 보안 모델의 핵심은 이전 정의된 그래프-글로벌 데이터 접근 역할을 중심으로 이루어집니다. 각 역할은 Neo4j 데이터 그래프 및 스키마에서 허용된 일련의 승인 작업을 포함합니다. 사용자는 이런 사용자 지정 역할 외에 하나 또는 그 이상의 역할에 할당될 수 있습니다.

Neo4j에는 사용자 인증 및 권한 부여를 수행하는 인증 공급자가 있습니다.

**네이티브 인증 공급자**
Neo4j는 유저 및 역할 정보를 디스크에 로컬로 저장하는 인증 공급자를 제공합니다. 이 옵션으로, [섹션 7.1.4, 기본 사용자 및 역할 관리](./authentication-authorization/native-user-role-management.md)에서 설명된 것과 같이 전체 사용자를 관리할 수 있습니다. 

**LDAP 인증 공급자**

인증 및 권한 부여를 통제하는 또 다른 방법은 내장 LDAP 커넥터를 통해 연결되는 Active Directory나 OpenLDAP와 같은 외부 보안 소프트웨어를 이용하는 것 입니다. Active Directory를 사용하는 LDAP 플러그인에 대한 설명은 [섹션 7.1.5, LDAP로 통합](/security/authentication-authorization/ldap-integration.md)에서 확인할 수 있습니다. 

**맞춤형 플러그인 인증 공급자**

네이티브 또는 LDAP에 만족하지 않는 특정 요구 사항을 가진 클라이언트를 위해서 Neo4j는 사용자 정의 통합 빌드 플러그인 옵션을 제공합니다. 이 옵션은 전문적인 Neo4j 서비스와 협의하여 고객 전달의 일부로 사용하는 것이 좋습니다. 이 플러그인은 [개발자 메뉴얼->플러그인](https://neo4j.com/docs/developer-manual/3.4/extending-neo4j/auth-plugins/)에 설명되어 있습니다. 

**Kerberos 인증 및 단일 사인온(sign-on)**

LDAP, 네이티브 및 사용자 지정 공급자 외에도 Neo4j는 인증 및 단일 로그온을 위한 Kerberos를 지원합니다. Kerberos 지원은 [Neo4j Kerberos Add-On](https://neo4j.com/docs/add-on/kerberos/1.0)을 통해서 제공됩니다. 