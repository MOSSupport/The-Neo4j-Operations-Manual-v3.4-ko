
## 7. 보안

```
이번 장은 Neo4j의 보안에 대해서 설명합니다. 
```

데이터를 보호하려면 먼저 업계 모범 사례를 참조하여 서버 및 네트워크 보안과 관련된 실제 데이터 보안을 확인하십시오. Neo4j 배치가 적절한 인증 및 권한 부여 규칙 설정하고 보안 지침을 준수하는지 확인하십시오.


이번 장은 아래와 같이 구성됩니다.:

- [인증 및 권한 부여](/security/authentication-authorization.md)

	- [소개](/security/authentication-authorization/introduction.md)
	- [용어](/security/authentication-authorization/terminology.md)
	- [인증 및 권한 부여 사용](/security/authentication-authorization/enable.md)
	- [기본 사용자 및 역할 관리](/security/authentication-authorization/native-user-and-role-management.md)
	  - [기본 역할](/security/authentication-authorization/native-user-and-role-management/native-roles.md)
	  - [맞춤 역할(Custom roles)](/security/authentication-authorization/native-user-and-role-management/custom-roles.md)
	  - [사용자와 역할 전파](/security/authentication-authorization/native-user-and-role-management/propagate-users-and-roles.md)
	  - [초기 사용자 및 역할 관리 절차](/security/authentication-authorization/native-user-and-role-management/procedures.md)
	- [LDAP와의 통합](/security/authentication-authorization/ldap-integration.md)
	- [하위 그래프 엑세스 제어](/security/authentication-authorization/subgraph-access-control.md)
	- [보안 속성-레벨](/security/authentication-authorization/property-level-security.md)
- [확장 기능 보안](/security/securing-extensions.md)
- [SSL framework](/security/ssl-framework.md)
- [Neo4j 브라우저 인증 정보 처리](/security/browser.md)
- [보안 점검목록](/security/checklist.md)

또한, 로그는 지속적인 분석 또는 특정 조사에 유용할 수 있습니다. [챕터 8, 모니터링](./monitoring.md)의 설명과 같이 [보안 이벤트 로그](/monitoring/logging/security-events-logging.md)와 [쿼리 로그](/monitoring/logging/query-logging.md) 생성 기능을 사용할 수 있습니다.
