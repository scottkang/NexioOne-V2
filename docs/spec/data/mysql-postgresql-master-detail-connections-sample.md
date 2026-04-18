# MySQL/PostgreSQL Master-Detail Connection Sample

`my-console`의 `Connections > Import JSON`으로 바로 가져올 수 있는 샘플 connection 묶음이다.

샘플 파일:
- `docs/spec/samples/mysql-postgresql-master-detail-connections-import.json`

포함 내용:
- `MYSQL_SOURCE_ORDERS`
- `POSTGRES_TARGET_ORDERS`

## Compose 준비

1. `connection-targets` profile로 compose를 올린다.
2. sample source/target schema를 적재한다.

```bash
./scripts/run-compose-stack.sh --with-connection-targets
./scripts/load-master-detail-sample-databases.sh
```

## Import

1. `Connections` 페이지에서 `Import JSON`을 연다.
2. 샘플 JSON 전체를 붙여 넣거나 파일을 선택한다.
3. import 후 `MYSQL_SOURCE_ORDERS`, `POSTGRES_TARGET_ORDERS` 두 connection이 보이는지 확인한다.

## Connection Details

- `MYSQL_SOURCE_ORDERS`
  - host/service: `mysql-source`
  - db: `nexio_source`
  - user/password: `nexio` / `nexio123`
- `POSTGRES_TARGET_ORDERS`
  - host/service: `postgres`
  - db: `nexio`
  - user/password: `nexio` / `nexio123`

샘플 JSON은 compose 네트워크 안에서 실행되는 `my-backend`/`my-console-backend`를 기준으로 service name(`mysql-source`, `postgres`)을 사용한다.
