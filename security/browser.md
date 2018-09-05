
## 7.4. 브라우저 자격 증명 처리 

```
이 섹션에서는 Neo4j 브라우저에서 처리되는 인증서를 제어하는 방법에 대해 다룹니다. 
```

Neo4j 브라우저에는 사용자가 반복적으로 Neo4j 자격 증명을 입력하지 않아도 되는 두 가지 메커니즘이 있습니다.

첫째, 브라우저가 웹 브라우저 탭에 열려있는 동안 현재 데이터베이스 세션이 활성화를 유지하도록 합니다. 이것은 시간초과가 되도록 합니다. 시간 초과는 [browser.credential_timeout](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_browser.credential_timeout)에서 설정할 수 있습니다. 시간초과 리셋은 브라우저에서 사용자 상호작용할 때마다 재설정 됩니다. 

둘째, 또한 브라우저는 Neo4j 사용자 자격 증명을 로컬에서 캐시할 수 있습니다. 증명서가 캐싱되면, 이들은 웹 브라우저 로컬 저장소에 암호화되지 않은 상태로 저장합니다. 만약 웹 브라우저 텝이 닫히고, 다시 열리면 세션은 캐시된 증명서를 이용해서 자동적으로 재 생성됩니다. 또한 로컬 저장소는 ```browser.credential_timeout```에 설정된 시간 초과의 영향을 받습니다. 게다가, 로컬 저장소 브라우저 내 자격 증명 캐싱을 모두 비활성화할 수 있습니다. 자격 증명 캐싱을 비활성화하려면, 서버 구성에서 [browser.retain_connection_credentials=false](https://neo4j.com/docs/operations-manual/3.4/reference/configuration-settings/#config_browser.retain_connection_credentials)을 설정하면 됩니다. 

사용자가 ```:server disconnect``` 명령어를 실행하면 기존 세션은 종료되고 로컬 저장소의 자격 증명 내용은 지워집니다.
