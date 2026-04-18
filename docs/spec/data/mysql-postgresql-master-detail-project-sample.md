# MySQL To PostgreSQL Master-Detail Project Sample

`my-console`의 `Project Import`로 바로 불러올 수 있는 샘플 프로젝트다.

샘플 파일:
- `docs/spec/samples/mysql-postgresql-master-detail-project-import.json`
- `docs/spec/samples/mysql-postgresql-master-detail-connections-import.json`

포함 내용:
- `SourceOrderMasterDetail` Data Definition
- `TargetOrderMasterDetail` Data Definition
- `MySQL(source) -> Mapping -> PostgreSQL(target)` Flow 1개

## Import

1. `Projects` 페이지에서 `Import JSON`을 연다.
2. 샘플 JSON 전체를 붙여 넣거나 파일을 선택한다.
3. import 후 프로젝트 상세의 `Flows`, `Data Definitions`를 확인한다.

## Sample Intent

Flow는 아래 의도를 표현한다.

1. `SQL Executor`
   - MySQL의 `source_order`, `source_order_item`를 읽어 `MASTER_DETAIL` 결과를 만든다.
2. `Mapping`
   - upstream/downstream Data Definition을 기준으로 입력/출력 tree를 보여준다.
3. `SQL Batch Executor`
   - PostgreSQL의 `target_order`, `target_order_item`에 master-detail 구조를 쓴다.

## Connection Expectations

샘플 Flow는 아래 connection 이름을 전제로 적어 두었다.

- `MYSQL_SOURCE_ORDERS`
- `POSTGRES_TARGET_ORDERS`

실행까지 검증하려면 해당 connection을 별도로 만들어야 한다. 샘플 import JSON은 `mysql-postgresql-master-detail-connections-sample.md`에 정리했다.

## Runtime Note

현재 runtime은 `MAPPING.outputMode=ROW_LIST`와 detail item field mapping을 지원하므로, 이 샘플은 `SQL_EXECUTOR -> MAPPING -> SQL_BATCH_EXECUTOR` master-detail 흐름의 기준 형태로 사용할 수 있다.

샘플의 `Mapping` 노드는 아래 두 수준을 함께 보여준다.
- master 필드 매핑
- `ITEMS[].LINE_NO -> details[].line_number` 같은 detail item 필드 rename 매핑
