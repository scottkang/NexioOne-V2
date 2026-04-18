# API Gateway Custom Extension Guide (Envoy Lua & Wasm)

## 1. 개요
본 문서는 Envoy 기반의 `api-gw` 기능을 커스텀하게 확장하기 위한 Lua 스크립팅 및 WebAssembly(Wasm) 필터 개발 가이드를 정의한다.

## 2. Lua 필터를 이용한 빠른 확장

단순한 헤더 조작, 응답 메시지 커스터마이징, 또는 조건부 라우팅이 필요한 경우 Lua 필터를 우선 사용한다.

### 2.1 Transaction ID 주입 예시
```lua
function envoy_on_request(request_handle)
  local tx_id = request_handle:headers():get("X-Nexio-Transaction-Id")
  if not tx_id then
    tx_id = "tx-" .. os.time() .. "-" .. math.random(1000, 9999)
    request_handle:headers():add("X-Nexio-Transaction-Id", tx_id)
  end
end
```

### 2.2 공통 응답 포맷 래핑
```lua
function envoy_on_response(response_handle)
  local status = response_handle:headers():get(":status")
  if status == "404" then
    response_handle:body():setBytes('{"status":404,"error":"Not Found","code":"SYS-1001","message":"resource not found","path":"/example"}')
    response_handle:headers():replace("content-type", "application/json")
  end
end
```

## 3. WebAssembly (Wasm) 기반 고도화

복잡한 계산, 외부 라이브러리 연동, 또는 고성능 데이터 변환이 필요한 경우 Wasm 필터를 개발한다.

### 3.1 개발 환경
- **Language**: C++, Rust, 또는 Go (TinyGo).
- **SDK**: `proxy-wasm-cpp-sdk`, `proxy-wasm-rust-sdk`.

### 3.2 배포 프로세스
1. 소스 코드 작성 및 `.wasm` 파일 컴파일.
2. `my-console-backend`의 xDS 서버를 통해 Wasm 바이너리 경로(또는 데이터) 전송.
3. Envoy가 `typed_config`를 통해 Wasm 필터 로드.

## 4. 필터 성능 최적화 가이드
- **I/O 최소화**: 외부 서비스 호출(External Authz 등)은 캐싱을 적극 활용한다.
- **메모리 제한**: Lua 스크립트 내에서 대량의 데이터를 로드하는 행위를 지양한다.
- **예외 처리**: 필터 내 에러가 전체 트래픽 차단으로 이어지지 않도록 `pcall`(Lua) 또는 적절한 에러 처리를 수행한다.
