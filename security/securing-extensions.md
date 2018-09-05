
## 7.2. 보안 확장기능 

```
이 섹션에서는 Neo4j에서 사용자 정의 내용 보안을 보장하는 방법에 대해 다룹니다. 
```

Neo4j는 Cypher에서 사용하는 사용자 코드를 이용해서 확장할 수 있습니다. 관련 내용은 [개발자 메뉴얼 → 절차](https://neo4j.com/docs/developer-manual/3.4/extending-neo4j/procedures/)에서 확인할 수 있습니다. 

이 섹션에서는 이런 추가적인 보안을 보장하는 방법에 대해서 다룹니다. 


### 7.2.1. 샌드박싱(Sandboxing)
 
Neo4j는 프로시저가 불안전한 APIs를 실수로 사용하지 않도록 샌드박싱 기능을 제공합니다. 예를 들어, 사용자 코드를 작성할 때 공개적으로 지원하지 않는 Neo4j APIs에 접속할 수 있고, 내부 APIs는 알림 없이 변경될 수 있습니다. 게다가, 이것을 사용하면 안전하지 않을 수도 있습니다. 샌드박싱 기능은 안전한 작업 및 보안 검사를 포함하고 공개적으로 지원되는 APIs로 확장을 제한합니다. 

구성 설정 ```dbms.security.procedures.unrestricted```는 쉼표로 구분된 프로시저 목록 및 / 또는 사용자 정의 함수 목록을 정의하여 데이터베이스 억세스 허용 및 샌드박스 기능을 우회할 수 있도록 합니다. 목록에는 완전한 프로시저 이름과 부분 이름에 와일드 카드 ```*```가 포함될 수 있습니다.

```dbms.security.procedures.unrestricted```에서 허용되지 않는 API 지원을 확장 로드하려면 보안 로그에 경고 메시지를 띄울 것 입니다. 경고는 확장 프로그램이 로드하려는 구성 요소에 대한 접근 권한이 없다는 것을 나타냅니다. 또한, 가짜 프로시저는 그 이름과 함께 로드됩니다. 가짜 프로시저를 호출하면, 더 많은 사용 권한이 필요하기 때문에 프로시저 로드에 실패했다는 에러 메시지를 출력합니다.  

**예 7.24. 샌드박스**

이 예에서는 프로시저 ```my.extensions.example```가 ```my.procedures``` 라이브러리의 프로시저 일부는 지원되지 APIs를 활용한다고 가정합니다. 

```
# Example sandboxing
dbms.security.procedures.unrestricted=my.extensions.example,my.procedures.*
```

### 7.2.2. 화이트 리스트(White listing)

화이트 리스트는 더 큰 라이브러리에서 몇 가지 확장만 로드할 때 사용할 수 있습니다. 

```dbms.security.procedures.whitelist``` 구성 설정은 라이브러리에서 활용가능한 특정 프로시저 이름을 정할 때 사용됩니다. 이것은 프로시저에서 로드할 콤마로 구분된 리스트를 정의합니다. 
리스트는 완전한 프로시저 이름과 부분 이름에 와일드 카드 ```*```가 포함됩니다.

**예 7.25. 화이트 리스트**

이 예에서는 ```apoc.coll``` 아래 모든 메소드뿐만 아니라 ```apoc.load.json``` 메소드 사용을 허용 합니다. 또한, ```apoc``` 라이브러리에서 이러한 기준에 맞는 것 외에 확장은 추가로 사용하지 않습니다. 

```
# Example white listing
dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.*
```
 
```dbms.security.procedures.whitelist```에 대해 알아둬야 할 몇 가지 사항이 있습니다 :

- 이 설정을 사용하면 나열된 것 이외 확장은 로드되지 않습니다. 특히 빈 문자열로 설정된 경우 확장은 로드되지 않습니다.
- 기본 설정 값은 ```*``` 입니다. 즉, 명시적으로 값을 지정하지 않거나 값이 없는 경우 [플러그인] (/ configuration / file-locations.md) 디렉토리의 모든 라이브러리가 로드됩니다.
- 이 매개변수가 지정한 확장이 내부 API에 액세스하도록 프로그래밍되었다면 [섹션 7.2.1, 샌드 박스](./securing-extensions.md)과 같이 명시적으로 허용해야 합니다. 

 