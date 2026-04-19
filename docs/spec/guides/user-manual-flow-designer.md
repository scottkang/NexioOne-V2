# User Guide: Flow Designer & Component Catalog

**Status**: [Draft/Design]
**Target Audience**: Flow Developers, Data Analysts, System Admins

## 1. 개요
본 문서는 사용자가 NexioOne `my-console` UI를 통해 비즈니스 로직(Flow)을 설계, 테스트, 배포하는 전체 과정을 가이드한다.

## 2. Flow 설계 라이프사이클

### Step 1: 프로젝트 및 Connection 준비
1. **프로젝트 생성**: [Projects] 메뉴에서 새로운 프로젝트를 생성한다.
2. **Connection 등록**: Flow에서 사용할 외부 연동 정보(DB, REST API, MQ 등)를 [Shared Connections]에 등록하고 연결 테스트(Ping)를 수행한다.
3. **Data Definition**: 입출력 데이터의 스키마(JSON)를 [Data Definitions]에 사전 정의한다.

### Step 2: Flow Designer 작성
1. 프로젝트 내에서 **신규 Flow**를 생성하고 Designer 캔버스를 연다.
2. 좌측 **Palette (Node Catalog)**에서 필요한 노드(컴포넌트)를 드래그 앤 드롭한다.
3. **노드 연결**: 노드 간의 포트를 드래그하여 실행 순서(Edge)를 정의한다. 조건 분기(Split/Join)가 필요한 경우 Expression을 설정한다.
4. **노드 설정**: 각 노드를 클릭하여 우측 Inspector 패널에서 상세 속성(Connection 선택, 쿼리 작성, 파라미터 매핑 등)을 입력한다.

### Step 3: 디버깅 및 테스트 (Dry-Run / Execute Stub)
1. 상단의 **[Test Run]** 버튼을 클릭한다.
2. 입력 파라미터(Input Payload)를 JSON 형태로 입력한다.
3. 실제 타겟 시스템에 영향을 주지 않는 **Dry-Run** 모드 또는 부분 실행 모드(**Execute Stub**)를 선택하여 실행한다.
4. 하단의 디버그 패널에서 각 단계별(Step) 입출력 데이터와 처리 시간, 에러 메시지를 확인하여 로직을 검증한다.

### Step 4: 배포 (Deployment)
1. 테스트가 완료된 Flow를 저장하고 **[Deploy]** 탭으로 이동한다.
2. 배포 대상 환경(Dev, Stage, Prod)을 선택하고 **[Create Deployment]**를 실행한다.
3. 배포 상태가 `SUCCESS`로 변경되면 해당 Flow는 외부 API 호출 또는 스케줄러에 의해 실행될 준비가 완료된다.

## 3. 주요 노드(컴포넌트) 활용 팁
- **SQL_EXECUTOR**: 단건 데이터 조회/수정에 사용. `connectionRef`를 매핑하고, 바인드 변수는 `${input.userId}` 형태로 작성한다.
- **REST_CLIENT**: 외부 API 호출 시 사용. Timeout 설정(Read/Connect)을 반드시 비즈니스 요구사항에 맞게 조정한다.
- **MAPPING**: 복잡한 데이터 구조 변환 시 사용. 시각적 매핑 UI를 활용하거나 표준 표현식 함수(`#fn`)를 조합하여 데이터를 가공한다.
