
### 7.1.5. LDAP와 통합  

```
이 섹션에서는 LDAP 시스템과 통합을위한 Neo4j 지원에 대해서 다룹니다. 
```

#### 7.1.5.1. LDAP 인증 공급자 구성

Neo4j는 Active Directory,OpenLDAP 또는 다른 LDAP와 호환가능한 인증 서비스를 허용하는 LDAP프로토콜을 지원합니다. 다음 예에서는 해당 서비스 관리 기능을 사용하여, 페더레이션된 사용자 관리가 LDAP 서비스로 연기되는 것을 확인할 수 있습니다. 즉, 네이티브 네오위즈 사용자 및 관리 역할을 완전히 해제하고 LDAP 그룹을 Neo4j [네이티브 역할](./native-user-and-role-management/native-roles.md) 및 사용자 역할에 매핑합니다.

모든 설정은 기본 구성 파일 ```neo4j.conf```에서 서버 시작시 정의해야 합니다. 먼저, LDAP를 인증 및 권한 제공자로 사용하기 위해 Neo4j를 설정하십시오. 

```
# Turn on security
dbms.security.auth_enabled=true

# Choose LDAP connector as security provider for both authentication and authorization
dbms.security.auth_provider=ldap
```

#### Active 디렉토리 구성

Active 디렉토리 구성 예제는 아래를 참조하십시오.

```
# Configure LDAP to point to the AD server
dbms.security.ldap.host=ldap://myactivedirectory.example.com

# Provide details on user structure within the LDAP system:
dbms.security.ldap.authentication.user_dn_template=cn={0},cn=Users,dc=example,dc=com
dbms.security.ldap.authorization.user_search_base=cn=Users,dc=example,dc=com
dbms.security.ldap.authorization.user_search_filter=(&(objectClass=*)(cn={0}))
dbms.security.ldap.authorization.group_membership_attributes=memberOf

# Configure the actual mapping between groups in the LDAP system and roles in Neo4j
dbms.security.ldap.authorization.group_to_role_mapping=\
  "cn=Neo4j Read Only,cn=Users,dc=neo4j,dc=com"      = reader       ;\
  "cn=Neo4j Read-Write,cn=Users,dc=neo4j,dc=com"     = publisher    ;\
  "cn=Neo4j Schema Manager,cn=Users,dc=neo4j,dc=com" = architect    ;\
  "cn=Neo4j Administrator,cn=Users,dc=neo4j,dc=com"  = admin        ;\
  "cn=Neo4j Procedures,cn=Users,dc=neo4j,dc=com"     = allowed_role

# In case defined users are not allowed to search for themselves,
# we can specify credentials for a user with read access to all users and groups:
# Note that this account only needs read-only access to the relevant parts of the LDAP directory
# and does not need to have access rights to Neo4j or any other systems.
#dbms.security.ldap.authorization.use_system_account=true
#dbms.security.ldap.authorization.system_username=cn=search-account,cn=Users,dc=example,dc=com
#dbms.security.ldap.authorization.system_password=secret
```

다음은```sAMAccountName```로 로그인할 수있는 ```Active Directory```의 구성입니다.

```
# Configure LDAP to point to the AD server
dbms.security.ldap.host=ldap://myactivedirectory.example.com

# Provide details on user structure within the LDAP system:
dbms.security.ldap.authorization.user_search_base=cn=Users,dc=example,dc=com
dbms.security.ldap.authorization.user_search_filter=(&(objectClass=*)(samaccountname={0}))
dbms.security.ldap.authorization.group_membership_attributes=memberOf

# Configure the actual mapping between groups in the LDAP system and roles in Neo4j
dbms.security.ldap.authorization.group_to_role_mapping=\
  "cn=Neo4j Read Only,cn=Users,dc=neo4j,dc=com"      = reader       ;\
  "cn=Neo4j Read-Write,cn=Users,dc=neo4j,dc=com"     = publisher    ;\
  "cn=Neo4j Schema Manager,cn=Users,dc=neo4j,dc=com" = architect    ;\
  "cn=Neo4j Administrator,cn=Users,dc=neo4j,dc=com"  = admin        ;\
  "cn=Neo4j Procedures,cn=Users,dc=neo4j,dc=com"     = allowed_role

# In case defined users are not allowed to search for themselves,
# we can specify credentials for a user with read access to all users and groups:
# Note that this account only needs read-only access to the relevant parts of the LDAP directory
# and does not need to have access rights to Neo4j or any other systems.
dbms.security.ldap.authorization.use_system_account=true
dbms.security.ldap.authorization.system_username=cn=search-account,cn=Users,dc=example,dc=com
dbms.security.ldap.authorization.system_password=secret

# Perform authentication with sAMAccountName instead of DN.
# Using this setting requires dbms.security.ldap.authorization.system_username and
# dbms.security.ldap.authorization.system_password to be used since there is no way to log in
# through ldap directly with the sAMAccountName.
# Instead the login name will be resolved to a DN that will be used to log in with.
dbms.security.ldap.authentication.use_samaccountname=true
```

다음은 Active 디렉토리 특성 ```sAMAccountName```을 사용하여 다른 조직 단위 사용자를 인증할 수 있는 Active 디렉토리의 다른 구성입니다:

```
# Configure LDAP to point to the AD server
dbms.security.ldap.host=ldap://myactivedirectory.example.com

dbms.security.ldap.authentication.user_dn_template={0}@example.com
dbms.security.ldap.authorization.user_search_base=dc=example,dc=com
dbms.security.ldap.authorization.user_search_filter=(&(objectClass=user)(sAMAccountName={0}))

# Configure the actual mapping between groups in the LDAP system and roles in Neo4j
dbms.security.ldap.authorization.group_to_role_mapping=\
"cn=Neo4j Read Only,cn=Users,dc=example,dc=com" = reader ;\
"cn=Neo4j Read-Write,cn=Users,dc=example,dc=com" = publisher ;\
"cn=Neo4j Schema Manager,cn=Users,dc=example,dc=com" = architect ;\
"cn=Neo4j Administrator,cn=Users,dc=example,dc=com" = admin ;\
"cn=Neo4j Procedures,cn=Users,dc=example,dc=com" = allowed_role
```

```user_dn_template``` 내에 ```{0}@example.com``` 패턴을 명시화하면 루트 도메인에서 시작할 수 있는 권한을 활성화 합니다. 트리 내 위치에 관계없이 전체 트리를 검사하여 사용자를 찾습니다.

```dbms.security.ldap.authentication.use_samaccountname``` 설정은 이 예에서 설정되지 않습니다. 

#### openLDAP 설정

아래는 openLDAP 설정과 관련된 예 입니다. 

```
# Configure LDAP to point to the OpenLDAP server
dbms.security.ldap.host=myopenldap.example.com

# Provide details on user structure within the LDAP system:
dbms.security.ldap.authentication.user_dn_template=cn={0},ou=users,dc=example,dc=com
dbms.security.ldap.authorization.user_search_base=ou=users,dc=example,dc=com
dbms.security.ldap.authorization.user_search_filter=(&(objectClass=*)(uid={0}))
dbms.security.ldap.authorization.group_membership_attributes=gidnumber

# Configure the actual mapping between groups in the OpenLDAP system and roles in Neo4j
dbms.security.ldap.authorization.group_to_role_mapping=\
  101 = reader       ;\
  102 = publisher    ;\
  103 = architect    ;\
  104 = admin        ;\
  105 = allowed_role

# In case defined users are not allowed to search for themselves,
# we can specify credentials for a user with read access to all users and groups:
# Note that this account only needs read-only access to the relevant parts of the LDAP directory
# and does not need to have access rights to Neo4j or any other systems.
#dbms.security.ldap.authorization.use_system_account=true
#dbms.security.ldap.authorization.system_username=cn=search-account,ou=users,dc=example,dc=com
#dbms.security.ldap.authorization.system_password=search-account-password
```

LDAP 구성 옵션에 대한 개요는 [섹션 A.1, 구성 설정](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/)에서 확인할 수 있습니다. 

| 변수명                                                       | 기본 값                              | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------ | ------------------------------------------------------------ |
| [dbms.security.ldap.authentication.user_dn_template](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authentication.user_dn_template) | `uid={0},ou=users,dc=example,dc=com` | 사용자 이름을 로그인에 필요한 LDAP관련 정규화된 이름으로 변환합니다. |
| [dbms.security.ldap.authorization.user_search_base](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authorization.user_search_base) | `ou=users,dc=example,dc=com`         | 기본 객체 또는 컨텍스트를 설정하여 사용자 객체를 검색합니다. |
| [dbms.security.ldap.authorization.user_search_filter](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authorization.user_search_filter) | `(&(objectClass=*)(uid={0}))`        | 사용자 검색을 위해 LDAP 검색 필터를 설정합니다.              |
| [dbms.security.ldap.authorization.group_membership_attributes](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authorization.group_membership_attributes) | `[memberOf]`                         | 역할 매핑에 사용할 그룹이 포함된 사용자 개체 특성 이름을 나열합니다. |
| [dbms.security.ldap.authorization.group_to_role_mapping](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authorization.group_to_role_mapping) |                                      | 그룹에 정의된 역할 `admin`, `architect`, `publisher`and `reader `또는 기타 사용자 정의 역할 권한 맵핑을 나열합니다. |


#### 7.1.5.2. 'ldapsearch'를 사용하여 구성 확인

LDAP 명령어 줄 도구 ```ldapsearch```를 사용하여 설정이 옳바른지 여부와 서버가 응답하는지 확인합니다. LDAP 구성 설정 값을 포함하는 검색 명령을 발행하여 이를 수행합니다. 

이 예제 검색은 인증(```simple``` 메커니즘 활용)과 사용자 'john'의 인증을 모두 입증합니다. 보다 발전된 사용법과 SASL 인증 메커니즘을 사용 방법은 ``ldapsearch`` 문서를 참조하십시오. 

```
dbms.security.ldap.authorization.use_system_account=false(기본 값):\
``` 

```
#ldapsearch -v -H ldap://<dbms.security.ldap.host> -x -D <dbms.security.ldap.authentication.user_dn_template : replace {0}> -W -b <dbms.security.ldap.authorization.user_search_base> "<dbms.security.ldap.authorization.user_search_filter : replace {0}>" <dbms.security.ldap.authorization.group_membership_attributes>
ldapsearch -v -H ldap://myactivedirectory.example.com:389 -x -D cn=john,cn=Users,dc=example,dc=com -W -b cn=Users,dc=example,dc=com "(&(objectClass=*)(cn=john))" memberOf 
```

```
dbms.security.ldap.authorization.use_system_account=true:
```

```
#ldapsearch -v -H ldap://<dbms.security.ldap.host> -x -D <dbms.security.ldap.authorization.system_username> -w <dbms.security.ldap.authorization.system_password> -b <dbms.security.ldap.authorization.user_search_base> "<dbms.security.ldap.authorization.user_search_filter>" <dbms.security.ldap.authorization.group_membership_attributes>
ldapsearch -v -H ldap://myactivedirectory.example.com:389 -x -D cn=search-account,cn=Users,dc=example,dc=com -w secret -b cn=Users,dc=example,dc=com "(&(objectClass=*)(cn=john))" memberOf
```

그 후, 리턴된 구성원 값이 ```dbms.security.ldap.authorization.group_to_role_mapping``` 역할에 매핑된 그룹임을 확인하고 성공적인 응답을 받았는지 확인합니다.

```
# extended LDIF
#
# LDAPv3
# base <cn=Users,dc=example,dc=com> with scope subtree
# filter: (cn=john)
# requesting: memberOf
#

# john, Users, example.com
dn: CN=john,CN=Users,DC=example,DC=com
memberOf: CN=Neo4j Read Only,CN=Users,DC=example,DC=com

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

#### 7.1.5.3. 인증 캐시 

*auth cache*는 성능을 지원하기 위해 Neo4j가 LDAP 서버로 인증 결과를 캐시하는 메커니즘입니다. 이것은 ```dbms.security.ldap.authentication.cache_enabled``` 및 ```dbms.security.auth_cache_ttl```로 구성됩니다. 

```
# Turn on authentication caching to ensure performance
dbms.security.ldap.authentication.cache_enabled=true
dbms.security.auth_cache_ttl=10m
```

| 변수명                                                       | 기본 값         | 설명                                                         |
| ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ |
| [dbms.security.ldap.authentication.cache_enabled](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.ldap.authentication.cache_enabled) | `true`          | LDAP 서버로 인증 결과를 캐시할지 여부를 결정합니다. 인증 캐싱 사용 여부는 회사 보안 지침 내에서 고려해야합니다.  REST API를 사용할 때 인증 캐싱을 사용하지 않으면 모든 요청에 대해 사용자 재 인증 및 재 인증이 발생하여 프로덕션 시스템의 성능에 심각한 영향을 줄 수 있으며 LDAP 서버에 과부하가 걸릴 수도 있습니다. |
| [dbms.security.auth_cache_ttl](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_dbms.security.auth_cache_ttl) | `10000 minutes` | 캐시된 인증 및 권한 부여 정보에 대한 TTL (Time To Live)입니다.  TTL을 0으로 설정하면 모든 인증 캐싱이 비활성화됩니다. 짧은 ttl은 성능에 영향을 줄 수 있는 빈번한 재 인증을  필요로합니다. 매우 긴 ttl은  LDAP 서버 사용자 설정에 대한 변경 사항이 적시에 Neo4j 인증에 반영되지 않을 수도 있음을 의미합니다. |

관리자는 인증 캐시를 삭제하여 페더레이션 인증 공급자 시스템에서 인증 및 권한 부여 정보를 다시 쿼리하도록할 수 있습니다.

**예 7.17 캐시 권한 제거**

이 명령어를 사용하려면 Neo4j 브라우저나 Neo4j 사이퍼 쉘을 사용하면 됩니다. 

```
CALL dbms.security.clearAuthCache()
```

#### 7.1.5.4. 사용가능한 암호화 방법 

```dbms.security.ldap.host``` 매개 변수를 지정하는 다음 방법은 모두 유효합니다. 이렇게하면 암호화없이 LDAP를 사용하여 구성됩니다. 프로토콜이나 포트를 명시화하지 않으면, 기본 포트 ```389```나 ```ldap```가 사용됩니다. 

```
dbms.security.ldap.host=myactivedirectory.example.com
dbms.security.ldap.host=myactivedirectory.example.com:389
dbms.security.ldap.host=ldap://myactivedirectory.example.com
dbms.security.ldap.host=ldap://myactivedirectory.example.com:389
```

**StartTLS를 통한 암호화와 LDAP 사용**

StartTLS를 통한 암호화로 Active 디렉토리를 구성하려면 다음 매개 변수를 설정하십시오.

```
dbms.security.ldap.use_starttls=true
dbms.security.ldap.host=ldap://myactivedirectory.example.com
```

**암호화된 LDAP와 함께 LDAP 사용**

Active 디렉토리를 암호화된 LDAPS와 구성하려면, ```dbms.security.ldap.host```를 다음과 같이 설정하십시오. 포트를 명시화하지 않으면 ```ldaps```는 기본 포트 ```636```를 이용해 사용됩니다. 

```
dbms.security.ldap.host=ldaps://myactivedirectory.example.com
dbms.security.ldap.host=ldaps://myactivedirectory.example.com:636
```

이 Active Directory 보안 방법은 더 이상 사용되지 않으므로 권장하지 않습니다. 대신 StartTLS를 통한 암호화와 Active 디렉토리를 사용하십시오.

#### 테스트 환경에서 자체-서명 인증서 사용 

프로덕션 환경에서는 LDAP 서버에 안전하게 액세스하기 위해 항상 인증 기관에서 발급한 SSL 인증서를 사용해야 합니다. 그러나 테스트 환경과 같이 LDAP 서버에서 자체 서명된 인증서를 사용하는 시나리오가 있습니다. 이 시나리오에서는 Neo4j에 로컬 인증서를 알려야합니다. 이 작업은 neo4j.conf에서 ```dbms.jvm.additional```을 사용하여 인증서의 세부 정보를 입력하여 수행됩니다.

**예 7.18. LDAP 서버에서 자체 서명 증명 명시**

이 예에서는 LDAP 서버에서 자체-서명된 증명서에 대해 세부 사항을 명시하는 방법에 대해서 다룹니다. 인증서 파일 ```MyCert.jks```에 대한 경로는 Neo4j 서버의 절대 경로입니다.

```
dbms.jvm.additional=-Djavax.net.ssl.keyStore=/path/to/MyCert.jks
dbms.jvm.additional=-Djavax.net.ssl.keyStorePassword=secret
dbms.jvm.additional=-Djavax.net.ssl.trustStore=/path/to/MyCert.jks
dbms.jvm.additional=-Djavax.net.ssl.trustStorePassword=secret
```