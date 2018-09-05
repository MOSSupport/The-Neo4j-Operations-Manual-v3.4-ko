#### 7.1.4.1. 기본 역할 

```
이 절에서는 Neo4j의 역할에 대해서 설명합니다. 
```

Neo4j는 역할 기반 액세스 제어 프레임워크에 다섯 가지 기본 제공 역할을 제공합니다.

```reader```
- 데이터 그래프에 읽기 전용으로 연결 (모든 노드, 관계, 속성).

```editor```
- 데이터 그래프에 읽기/쓰기 접근이 가능합니다. 
- 쓰기 액세스는 기존 속성 키, 노드 레이블 및 그래프의 관계 유형을 만들고 변경하는 것으로 제한됩니다.
 
```publisher```
- 데이터 그래프에 읽기/쓰기 접근.

```architect```
- 데이터 그래프에 읽기/쓰기 접근. 
- 다른 스키마 구성과 함께 인덱스에 대한 접근 설정/삭제. 

```admin```
- 데이터 그래프에 읽기/쓰기 접근. 
- 향후 다른 스키마 구성과 함께 인덱스에 대한 접근 설정/삭제. 
- 쿼리 보기/종료.

각 역할별로 규정된 데이터 및 데이터베이스의 일련 작업에 대해 자세히 설명합니다.

**표 7.1. 기본 역할 개요**

| Action                                   | `reader` | `editor` | `publisher` | `architect` | `admin` | (no role) |
| ---------------------------------------- | -------- | -------- | ----------- | ----------- | ------- | --------- |
| Change own password                      | `X`      | `X`      | `X`         | `X`         | `X`     | `X`       |
| View own details                         | `X`      | `X`      | `X`         | `X`         | `X`     | `X`       |
| Read data                                | `X`      | `X`      | `X`         | `X`         | `X`     |           |
| View own queries                         | `X`      | `X`      | `X`         | `X`         | `X`     |           |
| Terminate own queries                    | `X`      | `X`      | `X`         | `X`         | `X`     |           |
| Write/update/delete data                 |          | `X`      | `X`         | `X`         | `X`     |           |
| Create new types of properties key       |          |          | `X`         | `X`         | `X`     |           |
| Create new types of nodes labels         |          |          | `X`         | `X`         | `X`     |           |
| Create new types of relationship types   |          |          | `X`         | `X`         | `X`     |           |
| Create/drop index/constraint             |          |          |             | `X`         | `X`     |           |
| Create/delete user                       |          |          |             |             | `X`     |           |
| Change another user’s password           |          |          |             |             | `X`     |           |
| Assign/remove role to/from user          |          |          |             |             | `X`     |           |
| Suspend/activate user                    |          |          |             |             | `X`     |           |
| View all users/roles                     |          |          |             |             | `X`     |           |
| View all roles for a user                |          |          |             |             | `X`     |           |
| View all users for a role                |          |          |             |             | `X`     |           |
| View all queries                         |          |          |             |             | `X`     |           |
| Terminate all queries                    |          |          |             |             | `X`     |           |
| Dynamically change configuration ([3.7절, "동적 설정" 참조](/configuration/dynamic-settings.html)) |          |          |             |             | `X`     |           |

할당 된 역할이 없는 [사용자](/security/authentication-authorization/terminology.html/#term-user)에게는 읽기 권한조차 데이터와 관련된 권한이나 기능이 없습니다. 사용자는 둘 이상의 할당된 역할을 가질 수 있으며 이들의 결합은 사용자가 수행할 수 있는 데이터 작업을 결정합니다.

관리자가 다른 사용자 역할을 중지시키거나 삭제하면 다음 규칙이 적용됩니다.

- 관리자는 다른 사용자(다른 관리자 포함)를 중지 또는 삭제할 수는 있지만 사용자 자신은 삭제할 수 없습니다.
- 사용자를 삭제하면 실행중인 모든 쿼리와 세션이 종료됩니다.
- 삭제된 사용자에서 현재 실행되는 커리는 롤백 됩니다. 
- 사용자가 더 이상 다시 로그인 할 수 없게됩니다.(일시 중지 된 경우, 관리자가 다시 활성화 할 때까지).
- 사용자를 삭제하기 전에 사용자가 할당한 역할을 제거할 필요가 없습니다.
