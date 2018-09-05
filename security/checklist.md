
## 7.5. 보안 체크리스트 

```
이 섹션에서는 Neo4j 내 보안과 관련된 권장사항 요약 정보를 제공합니다. 
```

아래는 Neo4j 내에 특정 범위를 강조하는 간단한 체크리스트 입니다. 이 체크리스트는 애플리케이션의 적정 보안 수준을 보장하기 위해 주의가 필요합니다. 

1. 안전한 네트워크의 안전한 서버에서 Neo4j를 배포하십시오:

a. 서브넷과 방화벽을 사용하십시오:
b. 필요한 포트만 열어 두십시오. 관련 리스트 목록은 [섹션 3.2, “Ports”](../configuration/ports.md)을 참조하면 됩니다. 

특히, ````dbms.backup.address````설정에 지정된 포트에 대한 외부 접근이 없도록 합니다. 이 포트 보호를 실패하면 권한이 없는 사용자가 다른 컴퓨터에 데이터베이스 복사본을 만드는것과 같이 보안상 문제가 생길 수 있습니다.

2. 남은 데이터 보호 

a. 용량 암호화를 사용합니다. (예. Bitlocker)
b. 데이터베이스 덤프 접근을 관리합니다.(참조 [섹션 10.7, 덤프 및 데이터베이스 로딩](../tools/dump-load.md)) 및 백업 (참조[섹션 6.2, 백업 수행](../backup/perform-backup.md))
c. Neo4j 파일에 대한 올바른 파일 권한을 보장하여 데이터 파일 및 트랜잭션 로그 접근을 관리합니다. 허가 수준에 대한 내용은 [섹션 3.1.3, 허가](../configuration/file-locations.md)을 참조하십시오. 

3. 전송 중 데이터 보호:

a. Neo4j 데이터베이스의 원거리 접속은 암호화된 볼트 및 HTTPS에서만 오픈됩니다. 
b. 신뢰할 수 있는 인증 기관에서 발급한 SSL 인증서를 사용하십시오.

   1. Neo4j 설치가 암호화된 통신을 사용하도록 구성하려면, [섹션 7.3, SSL 프레임워크](./ssl-framework.md)를 참조하십시오. 
   2. 인과관계 클러스터링을 사용할 경우, 내부 클러스터 통신 사용 및 암호화를 설정하십시오. 
   자세한 내용은 [섹션 4.6, 내부-클러스터 암호화](../clustering/intra-cluster-encryption.md)을 참조하십시오. 
   3. 인과관계 클러스터링을 사용할 경우, 백업을 위해서 암호화를 설정/구성 하십시오. 이렇게하면 특정 SSL 정책 및 SSL 인증서가 있는 서버만 서버에 접근하고 백업을 수행합니다. 
   4. Bolt 또는 HTTPS 커넥터를 설정하려면 [섹션 3.6, Neo4j 커넥터 설정](../configuration/connectors.md)을 참조하십시오. 
   5. LDAP를 사용하는 경우, LDAP 시스템은 StartTLS 암호화를 사용하여 설정합니다; 참조 [암호화된 StartTLS를 통한 LDAP 사용](/security/authentication-authorization/ldap-integration.md).

4. 고객 확장 기능의 보안 강화 :

a. 배포할 사용자 코드(프로시저 및 관리되지 않는 확장)를 확인하고 실수로 제품/데이터 일부를 노출하지 않도록하십시오.
b. 설정 ```dbms.security.procedures.unrestricted```와 ```dbms.security.procedures.whitelist```를 조사하여 의도적으로 노출된 배포 정보를 포함하도록 하십시오. 

5. Neo4j 파일에 올바른 파일 승인 있는지 확인하십시오. 

6. *bin*, *lib* 및 *plugins* 디렉토리 액세스를 제한하여 허가되지 않은 확장 실행을 보호하십시오. Neo4j를 실행하는 운영 시스템 사용자만 파일에 대한 사용 권한을 가집니다. 사용 권한 레벨은 [섹션 3.1.3, 권한](../configuration/file-locations.md)을 참조하십시오. 

7. 만약 ```LOAD CSV```가 활성화되었다면, 권한이 없는 사용자는 데이터를 임포트할 수 없도록 하십시오. ```LOAD CSV```를 설정하는 방법은 [개발자 메뉴얼 → LOAD CSV](https://neo4j.com/docs/developer-manual/3.4/cypher/clauses/load-csv)에서 확인할 수 있습니다. 

8. Neo4j 인증을 끄지 마십시오. 자세한 내용은 [섹션 7.1.3,인증 및 권한 부여 활성화](/authentication-authorization/enable.md)를 참조하십시오. 

9. *neo4j.conf* 파일에서 중요도가 떨어지는 함수 (예 : ```dbms.shell.port``` 매개 변수로 제어되는 ```neo4j-shell```) 및 원격 JMX (```dbms.jvm.additional = -Dcom.sun.management.jmxremote.port = 3637``` 매개 변수 설정에서 제어)와 관련된 포트를 조사하십시오.)
 
10. Neo4j Browser 기본 자격 증명 처리가 보안 규정을 준수하는지 확인하려면, [섹션 7.4, 브라우저 자격증명 처리](./browser.md)를 참조하십시오. 필요한 경우 다음 지침을 따라 설정하십시오.
 
11. Neo4j의 최신 버전 패치를 사용하십시오. 