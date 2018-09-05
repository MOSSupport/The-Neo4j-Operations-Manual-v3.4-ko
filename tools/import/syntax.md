
### 10.6.1. 구문 

CSV 파일에서 데이터를 임포트하는 구문은 다음과 같습니다:

```
neo4j-admin import [--mode=csv] [--database=<name>]
                          [--additional-config=<config-file-path>]
                          [--report-file=<filename>]
                          [--nodes[:Label1:Label2]=<"file1,file2,...">]
                          [--relationships[:RELATIONSHIP_TYPE]=<"file1,file2,...">]
                          [--id-type=<STRING|INTEGER|ACTUAL>]
                          [--input-encoding=<character-set>]
                          [--ignore-extra-columns[=<true|false>]]
                          [--ignore-duplicate-nodes[=<true|false>]]
                          [--ignore-missing-nodes[=<true|false>]]
                          [--multiline-fields[=<true|false>]]
                          [--delimiter=<delimiter-character>]
                          [--array-delimiter=<array-delimiter-character>]
                          [--quote=<quotation-character>]
                          [--max-memory=<max-memory-that-importer-can-use>]
                          [--f=<File containing all arguments to this import>]
                          [--high-io=<true/false>]
```

**예제 10.5. CSV 파일에서 데이터 임포트**
 
movies_header.csv, movies.csv, actors_header.csv, actors.csv, roles_header.csv 및 roles.csv의 6 가지 파일로 구성되도록 데이터를 각 섹션별로 [10.6.2 절, CSV 파일 헤더 형식](/tools/import/file-header-format.md)에 형식을 지정했다고 가정합니다. 아래 명령은 데이터 세트 세 가지를 가져옵니다.

```
neo4j_home$ bin/neo4j-admin import --nodes "import/movies_header.csv,import/movies.csv" \
--nodes "import/actors_header.csv import/actors.csv" \
--relationships "import/roles_header.csv,import/roles.csv"
```

3.0-이전 데이터베이스로 임포트하는 명령어는 다음과 같습니다:

```
neo4j-admin import --mode=database [--database=<name>]
                          [--additional-config=<config-file-path>]
                          [--from=<source-directory>]
```

3.0-이전 버전 데이터베이스 업그레이드에 ```neo4j-admin import``` 이용 방법은 [업그레이드](../../upgrade/deployment-upgrading.md)을 참고하십시오.  
