
### 10.6.2. CSV 파일 헤더 형식 

```
이 절에서는 Neo4j 임포트 도구를 사용할 때 CSV 파일 헤더 포맷에 대해 설명합니다.
```

이 섹션에서는 다음에 대해 다룹니다:

- 헤더 파일
- 속성
- 노드 파일 
- 관계 파일 
- ID 공백 사용  
- 컬럼 스킵  
- 압축 파일 

#### 10.6.2.1. 헤더 파일 

헤더 파일의 각 데이터 소스는 데이터 필드 해석 방법을 지정합니다. 헤더 파일 및 데이터 파일에 동일한 구분 기호를 사용해야 합니다.

헤더에는 ```<name> : <field_type>``` 형식의 각 필드 정보가 있습니다. ```<name>```은 특성 및 노드 ID에 사용됩니다. 다른 모든 경우 필드의 ```<name>``` 부분은 무시됩니다.

#### 10.6.2.2. 속성

속성의 경우, 필드의 ```<name>``` 부분은 속성 키이고 ```<field_type>``` 부분은 데이터 형식입니다(아래 참조). 노드 데이터 파일 및 관계 데이터 파일 모두 속성을 가집니다. 

**데이터 형식**

속성의 데이터 형식을 지정하기 위해서 ```int```, ```long```, ```float```, ```double```, ```boolean```, ```byte```, ```short```, ```char```, ```string``` 중 하나를 사용합니다. 데이터 형식이 없을 경우 기본 값은  ```string```입니다. 배열 형식은 ```[]```을 데이터 형식에 추가하여 정의합니다. 기본적으로 배열 값은 ```;```로 구분합니다. 다른 구분자는 ```--array-delimiter```를 사용합니다.

**```point``` 데이터 타입의 특별 고려사항**

포인트는 맵(maps)의 Cypher 구문을 사용하여 명시합니다. map은 [Cypher 포인트 함수](https://neo4j.com/docs/developer-manual/3.4/cypher/functions/spatial/) 입력에 동일한 키 값을 허용 합니다 . 헤더의 포인트 데이터 유형은 해당 열의 모든 값에 사용되는 기본값 맵으로 수정할 수 있습니다 (예 : ```point {crs : 'WGS-84'}```. 헤더를 이 방법으로 명시하면 데이터 파일 값 위치에 불완전한 지도를 가질 수도 있습니다. 선택적으로, 데이터 파일 내 값이 헤더에서 기본 값을 대체할 수 있습니다. 

**예 10.6. ```point``` 데이터 유형의 형식 속성**

이 예는 임포트 헤더 및 데이터 파일에서 ```point```데이터 유형을 사용하는 다양한 방법을 보여줍니다. 도시의 이름과 위치 좌표를 가져옵니다. 먼저, 헤더를 아래와 같이 정의합니다. 

```
:ID,name,location:point{crs:WGS-84}
```

그 후, 데이터 파일 내 도시를 정의합니다. 

- 처음 도시의 위치는 헤더에 정의된 조정 시스템을 사용할 때 ```latitude```와 ```longitude```을 이용해서 정의합니다. 

- 두 번째 도시는 ```x```와 ```y```를 대신 사용합니다. 이것은 일반적으로 좌표 참조 시스템 ```cartesian```를 사용하는 포인트로 연결됩니다. 헤더가 ```crs:WGS-84```를 정의하기 때문에, 조정 참조 시스템을 사용합니다. 

- 세 번째 도시는 헤더에 정의된 조정 참조 시스템을 대체하고, 명시적으로 ```WGS-84-3D```를 설정합니다. 

```
city01,"Malmö",{latitude:55.6121514, longitude:12.9950357}
city02,"London",{y:51.507222, x:-0.1275}
city03,"San Mateo",{latitude:37.554167, longitude:-122.313056, height: 100, crs:'WGS-84-3D'}
````

**임시 데이터 타입의 특별 고려사항**

임시 데이터 유형의 형식은 [개발자 메뉴얼 -> 임시 인스턴스 문법](https://neo4j.com/docs/developer-manual/3.4/cypher/syntax/temporal)와 [개발자 메뉴얼 -> 문법 지속기간](ㅍhttps://neo4j.com/docs/developer-manual/3.4/cypher/syntax/temporal)에 설명된 것과 같이 정의해야 합니다. *Time* 값의 표준 시간대를 명시할 수 있습니다. 예: ```time{timezone:+02:00}```, *DateTime* 값 : ```datetime{timezone:Europe/Stockholm}``` . 기본 표준시간대가 명시되어 있지 않다면 표준 시간대는 ```db.temporal.timezon``` 환경 설정에서 조정할 수 있습니다. 기본 표준시간대는 데이터 파일의 값 위치에서 명시적으로 대체할 수 있습니다. 

**예 10.7. 임시 데이터 유형의 속성 형식**

이 예에서는 날짜시간 데이터 타입을 임포트 헤더 및 데이터 파일에서 사용하는 방법에 대해 설명합니다. 

먼저, 헤더를 두 가지 DateTime 행으로 정의합니다. 처음 것은 시간대를 정의하지만, 두 번째는 정의하지 않습니다. 

```
:ID,date1:datetime{timezone:Europe/Stockholm},date2:datetime
```

그 후, 데이터 파일에서 날짜를 정의합니다. 

- 첫 행의 두 가지 값은 정확한 표준 시간대를 명시하지 않습니다. ```date1``` 값은 그 필드의 헤더에 명시된 ```Europe/Stockholm``` 시간대를 사용합니다. ```date2```는 데이터베이스에 설정된 기본 시간대를 사용합니다. 

- 두 번째 행에서는 ```date1```과 ```date2```는 시간대가 ```Europe/Berlin```가 되도록 설정합니다. 이것은 데이터베이스 기본 시간대로 설정된 것과 같이 ```date1```의 헤더 정의를 대체합니다. 

#### 10.6.2.3. 노드 파일

노드 데이터가 들어있는 파일에는 필수 필드, ```ID```와 선택적 필드, ```LABEL:```이 있습니다.

**ID**

각 노드는 임포트되는 동안 사용할 고유 ID가 있어야 합니다. ID는 노드간 관계 생성 시 올바른 노드를 찾는데 사용됩니다. ID는 임포트되는 모든 노드 및 레이블이 다른 노드와 구별되도록 고유한 값을 가져야 합니다. 고유한 ID는 필드 정의 ```<name>:ID```의 ```<name>```에 정의 된 이름의 등록 정보에 유지 됩니다. 속성의 이름이 정의되어 있지않으면, 고유 ID는 임포트할 수 있지만 나중에 참조할 수는 없습니다.   

**LABEL**

이 필드에서 하나 이상의 레이블을 읽습니다. 배열과 마찬가지로, 여러 레이블은  ```;```로 구분되거나, ```--array-delimiter```로 지정된 문자를 표시합니다. 

**예제 10.8. 노드 파일 정의하기**  

이 예제는 movies.csv 파일에서 3가지 영화를 정의합니다.  영화는 ```[movieId]```, ```year```, ```title``` 속성을 가집니다. 레이블 필드도 지정합니다.

```
movieId:ID,title,year:int,:LABEL
```

movies.csv.파일에서 3가지 영화를 정의합니다. 이들은 헤더 파일에 정의된 모든 속성을 포함합니다. 모든 영화는 영화 레이블이 주어집니다. 두 영화에는 ```Sequel```이라는 레이블이 붙습니다.

```
tt0133093,"The Matrix",1999,Movie
tt0234215,"The Matrix Reloaded",2003,Movie;Sequel
tt0242653,"The Matrix Revolutions",2003,Movie;Sequel
```

비슷하게, 3명의 배우는 actors.csv 및 actors_header.csv 파일에 정의합니다. 이들 모두 ```personId```와 ```name``` 속성과, ```Actor``` 레이블을 갖습니다.

```
personId:ID,name,:LABEL
```

```
keanu,"Keanu Reeves",Actor
laurence,"Laurence Fishburne",Actor
carrieanne,"Carrie-Anne Moss",Actor
```

#### 10.6.2.4. 관계 파일

관계 데이터 파일에는 3개의 필수 필드가 있습니다:   

**TYPE**
관계의 형식

**START_ID**
이 관계의 시작 노드의 ID

**END_ID**
이 관계의 끝 노드의 ID

```START_ID``` 및 ```END_ID```는 노드 데이터 소스에 정의된 고유한 노드 ID를 참조합니다. 이들 중 어느 것도 이름을 사용하지 않습니다. ```<name>:START_ID``` 또는 ```<name>:END_ID```가 정의되면 ```<name>``` 부분은 무시됩니다.

**예제 10.9. 관계 파일 정의하기**

이 예제에서는 이전 예제의 두 노드 파일이 다음 관계 파일과 함께 사용된다고 가정합니다. 

```roles_header.csv```, ```roles.csv``` 파일 내 배우와 영화 간 관계를 정의합니다. 각 행은 관계 유형 ```ACTED_IN``` 관계로 시작 노드와 끝 노드를 연결합니다. 위 노드 파일에서 고유한 식별자인 ```personId```와 ```movieId```를 사용에 유의합니다. 영화에서 배우가 연기한 캐릭터의 이름은 관계에서 ```role```속성으로 저장됩니다.

```
:START_ID,role,:END_ID,:TYPE
```

```
keanu,"Neo",tt0133093,ACTED_IN
keanu,"Neo",tt0234215,ACTED_IN
keanu,"Neo",tt0242653,ACTED_IN
laurence,"Morpheus",tt0133093,ACTED_IN
laurence,"Morpheus",tt0234215,ACTED_IN
laurence,"Morpheus",tt0242653,ACTED_IN
carrieanne,"Trinity",tt0133093,ACTED_IN
carrieanne,"Trinity",tt0234215,ACTED_IN
carrieanne,"Trinity",tt0242653,ACTED_IN
```


#### 10.6.2.5. ID 공백 사용

기본적으로 임포트 도구는 노드 ID(식별자)가 여러 노드 파일에서 고유하다고 가정합니다. 대부분 ID는 각 엔티티 파일들에서 고유합니다. (예: CSV 파일에 관계형 데이터베이스에서 추출한 데이터가 들어 있고, ID 필드가 해당 테이블의 기본 키 열(primary key column)에서 가져온 경우). 이 상황을 처리하기 위해 ID 공백을 정의합니다. ```ID``` 공백은 ```ID(<ID space identifier>)```를 사용하여 노드 파일의 ID 필드에 정의됩니다. 관계 파일에서 ID 공백의 ID를 참조하려면 ```START_ID(<ID space identifier>)``` 및 ```END_ID(<ID space identifier>)``` 구문을 사용합니다.

**예제 10.10. ID 공백 정의 및 사용** 

movies.csv 파일에서 ```Movie-ID``` ID 스페이스를 정의합니다.

```
movieId:ID(Movie-ID),title,year:int,:LABEL
```

```
1,"The Matrix",1999,Movie
2,"The Matrix Reloaded",2003,Movie;Sequel
3,"The Matrix Revolutions",2003,Movie;Sequel
```

actors_header.csv 파일 헤더에서 ```Actor-ID``` ID 공백 정의합니다.

```
personId:ID(Actor-ID),name,:LABEL
```

```
1,"Keanu Reeves",Actor
2,"Laurence Fishburne",Actor
3,"Carrie-Anne Moss",Actor
```

영화 배우들을 연결할 때, 이전에 정의한 ID 공백을 사용합니다.

```
:START_ID(Actor-ID),role,:END_ID(Movie-ID),:TYPE
```

```
1,"Neo",1,ACTED_IN
1,"Neo",2,ACTED_IN
1,"Neo",3,ACTED_IN
2,"Morpheus",1,ACTED_IN
2,"Morpheus",2,ACTED_IN
2,"Morpheus",3,ACTED_IN
3,"Trinity",1,ACTED_IN
3,"Trinity",2,ACTED_IN
3,"Trinity",3,ACTED_IN
```

### 10.6.2.6. 스킵하는 열(skipping column)

데이터에서 필드를 완전히 무시하고 싶다면 헤더 파일의 ```IGNORE``` 키워드를 사용하십시오. ```IGNORE``` 앞에 ```:```를 붙여야합니다.

**예제 10.11. 열 건너뛰기**

이 예에서 노드 파일의 세 번째 열에 있는 데이터는 건너 뜁니다. ```IGNORE``` 키워드 앞에는 접두어 ```:```가 붙습니다.

```
personId:ID,name,:IGNORE,:LABEL
```

```
keanu,"Keanu Reeves","male",Actor
laurence,"Laurence Fishburne","male",Actor
carrieanne,"Carrie-Anne Moss","female",Actor
```

임포트하는 열의 오른쪽 열들에 불필요한 데이터가 모두 있으면, 명령행 옵션 ```--ignore-extra-columns```를 대신 사용할 수 있습니다.


#### 10.6.2.7. 압축 파일 임포트

The import tool can handle files compressed with `zip` or `gzip`. Each compressed file must contain a single file.

임포트 도구는 ```zip``` 또는 ```gzip```로 압축된 파일을 처리합니다. 각 압축 파일에는 단일 파일이 있어야합니다.

**예시 10.12. 압축 파일을 사용해서 임포트 수행**

```
neo4j_home$ ls import
actors-header.csv  actors.csv.zip  movies-header.csv  movies.csv.gz  roles-header.csv  roles.csv.gz
neo4j_home$ bin/neo4j-admin import --nodes import/movies-header.csv,import/movies.csv.gz --nodes import/actors-header.csv,import/actors.csv.zip --relationships import/roles-header.csv,import/roles.csv.gz
```