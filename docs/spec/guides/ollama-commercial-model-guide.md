# Ollama Commercial Model Guide

## Purpose
- `Ollama`에서 사용할 수 있는 모델 중 상업적 사용 후보를 빠르게 선별하기 위한 운영 메모다.
- 기준 시점은 `2026-03-15`이며, 실제 도입 전에는 각 모델의 최신 라이선스를 다시 확인해야 한다.
- 이 문서는 법률 자문을 대체하지 않는다.

## Key Rule
- `Ollama` 자체가 아니라 **개별 모델 라이선스**를 확인해야 한다.
- 같은 `Ollama` 라이브러리 안에 있어도 상업 사용 가능 여부는 모델마다 다르다.

## Recommended Starting Point

### 1. Phi-3.5-mini-instruct
- 상태: 상업 사용 시작점으로 가장 단순한 후보
- 이유:
  - Microsoft 모델 카드 기준 `commercial and research use`
  - `MIT license`
  - 경량 모델이라 내부 inference fallback 용도로 현실적
- 참고:
  - https://huggingface.co/microsoft/Phi-3.5-mini-instruct

### 2. Llama 3.1 / 3.3
- 상태: 상업 사용 가능 후보
- 이유:
  - Meta community license 기반으로 상업 사용이 가능
  - `Ollama` 라이브러리에서 바로 사용하기 쉽다
- 주의:
  - 매우 큰 서비스 사업자에 대한 별도 조건이 있다
  - 조직 규모와 서비스 형태에 따라 license 조건을 따로 검토해야 한다
- 참고:
  - https://ollama.com/library/llama3.1
  - https://ollama.com/library/llama3.3
  - https://ollama.com/library/llama3.1/blobs/0ba8f0e314b4

### 3. DeepSeek-R1
- 상태: 조건부 상업 사용 후보
- 이유:
  - `Ollama` 라이브러리 페이지 기준으로 commercial use 지원
  - 모델 가중치 라이선스가 `MIT`라고 명시돼 있다
- 주의:
  - distilled variant는 기반(base) 모델 라이선스 영향이 있을 수 있다
  - 실제 배포 모델이 순수 `DeepSeek-R1`인지, distill 버전인지 분리 확인해야 한다
- 참고:
  - https://ollama.com/library/deepseek-r1

## Use With Caution

### Qwen2.5-3B-Instruct
- 상태: 현재 기준으로 상업 무료 사용 후보로 보지 않는 편이 안전
- 이유:
  - 공개 license 문구에 `NON-COMMERCIAL PURPOSES ONLY`가 포함돼 있다
  - 상업 사용 시 별도 라이선스 요청 조건이 붙어 있다
- 운영 판단:
  - 성능이나 한국어 적합성이 좋아 보여도, 현재 저장소의 상업 사용 기본 후보로 두지 않는다
- 참고:
  - https://huggingface.co/Qwen/Qwen2.5-3B-Instruct/blob/main/LICENSE

## Practical Recommendation For NexioOne

### Default
- `Phi-3.5-mini-instruct`
- 이유:
  - 라이선스 해석이 가장 단순하다
  - 작은 모델이라 `schedule interpretation fallback` 용도에 맞다

### Alternate
- `Llama 3.1` 또는 `Llama 3.3`
- 조건:
  - Meta community license 조건을 사전 검토할 것

### Avoid As Default Commercial Candidate
- `Qwen2.5-3B-Instruct`
- 이유:
  - 현재 기준 비상업 제한 문구가 있어 리스크가 크다

## Suggested Runtime Policy
- 기본 parser는 규칙 기반으로 유지한다
- 규칙 기반 해석 실패 시에만 내부 LLM fallback을 사용한다
- fallback 모델은 `Phi-3.5-mini-instruct`를 1순위로 검토한다
- 모델 교체 시에는 다음을 같이 검토한다
  - 최신 license text
  - 한국어 schedule 해석 정확도
  - 메모리 사용량
  - 응답 시간

## Verification Checklist
- 모델 도입 전 다음을 다시 확인한다
- `Ollama` library page
- 모델의 공식 Hugging Face 또는 공식 배포 페이지
- license 전문 또는 commercial use 조항
- distill / derivative 여부

## Source Links
- Ollama library: https://www.ollama.com/library
- Ollama Docker docs: https://docs.ollama.com/docker
- Llama 3.1 library page: https://ollama.com/library/llama3.1
- Llama 3.3 library page: https://ollama.com/library/llama3.3
- Llama 3.1 license blob: https://ollama.com/library/llama3.1/blobs/0ba8f0e314b4
- Phi-3.5-mini-instruct model card: https://huggingface.co/microsoft/Phi-3.5-mini-instruct
- DeepSeek-R1 library page: https://ollama.com/library/deepseek-r1
- Qwen2.5-3B-Instruct license: https://huggingface.co/Qwen/Qwen2.5-3B-Instruct/blob/main/LICENSE
