# Standard Expression & Function Library (#fn)

## 1. 개요
NexioOne 표현식 엔진(SpEL 기반)에서 사용할 수 있는 표준 함수 세트인 `#fn` 헬퍼의 기능을 정의한다. 모든 함수는 런타임 엔진에 내장되어 있으며, Flow 설계 시 자동 완성 기능을 지원한다.

## 2. 주요 함수 목록

### 2.1 문자열 (String)
- `#fn.concat(str1, str2, ...)`: 가변 인자 문자열 결합.
- `#fn.substring(str, start, end)`: 안전한 부분 문자열 추출.
- `#fn.upper(str) / #fn.lower(str)`: 대소문자 변환.
- `#fn.trim(str)`: 앞뒤 공백 제거.
- `#fn.replace(str, target, replacement)`: 문자열 치환.
- `#fn.contains(str, search)`: 포함 여부 확인 (Boolean).

### 2.2 숫자 (Numeric)
- `#fn.abs(num)`: 절대값.
- `#fn.round(num, decimalPlaces)`: 지정된 소수점 자리수에서 반올림.
- `#fn.fmtNumber(num, pattern)`: Java DecimalFormat 기반 포맷팅 (예: `#,###`).
- `#fn.parseInt(str) / #fn.parseFloat(str)`: 문자열을 숫자로 변환 (실패 시 null).

### 2.3 날짜 및 시간 (Date/Time)
- `#fn.now()`: 현재 일시 (`LocalDateTime`).
- `#fn.fmtDate(date, pattern)`: 날짜 포맷팅 (예: `yyyy-MM-dd HH:mm:ss`).
- `#fn.dateAdd(date, amount, unit)`: `unit` ('Y', 'M', 'D', 'H', 'm', 's') 단위 연산.
- `#fn.diff(date1, date2, unit)`: 두 날짜 간의 차이 계산.

### 2.4 유틸리티 (Utility)
- `#fn.uuid()`: RFC 4122 호환 UUID 생성.
- `#fn.nvl(val, default)`: Null Safety 처리.
- `#fn.jsonPath(json, path)`: Jayway JsonPath를 이용한 데이터 추출.
- `#fn.isEmpty(val)`: String, Collection, Map의 빈 상태 체크.
- `#fn.size(val)`: Collection 또는 Map의 크기 반환.

## 3. 사용 예시
- **ROUTER**: `${#fn.size(rows) > 0}`
- **MAPPING**: `fullName: ${#fn.concat(lastName, firstName)}`
- **FUNCTION**: `expired: ${#fn.diff(#fn.now(), expiryDate, 'D') < 0}`
