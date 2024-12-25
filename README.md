# kipris-ip-data-pipeline

KIPRIS(한국특허정보원)의 API를 활용하여 기업과 대학의 특허, 실용신안, 디자인, 상표 정보를 수집하고 데이터베이스에 저장하는 시스템입니다.(준소프트웨어_기업연계 프로젝트)

* 소스코드는 업로드하지 않습니다.

## 프로젝트 구조

```
.
├── api/                    # API 관련 모듈
│   ├── api_fetcher.py     # API 요청 및 응답 처리
│   └── api_query_generator.py # API 쿼리 생성
├── config/                 # 설정 파일
├── db/                     # 데이터베이스 관련 모듈
├── logs/                   # 로그 파일 저장
├── mock_server/           # 테스트용 목업 서버
├── preprocessors/         # 데이터 전처리 모듈
├── utils/                 # 유틸리티 함수
├── main.py               # 메인 실행 파일
└── kipris-db_schema.sql  # 데이터베이스 스키마
```

## 주요 기능

1. **데이터 수집**
   - 기업 및 대학의 출원인 번호 수집
   - 특허/실용신안 정보 수집
   - 디자인 정보 수집
   - 상표 정보 수집

2. **데이터 처리**
   - XML 응답 데이터를 JSON 형식으로 변환
   - 수집된 데이터 전처리 및 정제
   - MySQL 데이터베이스에 데이터 적재

3. **모니터링**
   - Prometheus 메트릭을 통한 API 요청 모니터링
   - 로깅 시스템을 통한 에러 추적
   - 슬랙을 통한 알림 기능

## 실행 방법

1. 환경 설정
   - `.env` 파일에 필요한 환경 변수 설정
   - MySQL 데이터베이스 스키마 생성 (`kipris-db_schema.sql`)

2. 메인 프로그램 실행
   ```bash
   python main.py
   ```

## 데이터베이스 구조

- `tb24_100_bizinfo`: 기업 정보 테이블
- `tb24_300_corp_ipr_reg`: 기업 지식재산권 정보 테이블
- `tb24_400_univ_ipr_reg`: 대학 지식재산권 정보 테이블

## 모니터링

- Prometheus 메트릭:
  - 요청 수 카운터 (`requests_total`)
  - 응답 시간 측정 (`response_seconds`)
  - 각 메트릭은 IPR 모드와 조직 유형별로 구분

## 에러 처리

- 로깅 시스템을 통한 에러 추적
- 슬랙 알림을 통한 실시간 모니터링
- 토큰 버킷 알고리즘을 통한 API 요청 제한

## 성능 최적화

- 비동기 I/O를 통한 동시 요청 처리
- 토큰 버킷 알고리즘을 통한 API 요청 제한
- 커넥션 풀을 통한 데이터베이스 연결 관리

## 모니터링 시스템 상세 (Prometheus & Grafana)

### 프로메테우스 메트릭 수집

1. **요청 메트릭**
   - `kipris_requests_total`: API 요청 총 횟수 카운터
   - `kipris_errors_total`: 에러 발생 횟수 카운터
   - 레이블: ipr_mode(특허/상표/디자인), org_type(기업/대학), status(성공/실패)
   - 에러 타입별 트래킹 (error_type 레이블)

2. **성능 메트릭**
   - `kipris_response_seconds`: API 응답 시간 측정 (Summary)
   - `kipris_worker_process_seconds`: 워커 처리 시간 측정 (Summary)
   - 응답 시간 버킷: 0.1s, 0.25s, 0.5s, 0.75s, 1.0s, 2.5s, 5.0s, 7.5s, 10.0s

3. **리소스 메트릭**
   - `kipris_active_requests`: 현재 활성 요청 수 (Gauge)
   - `kipris_queue_size`: 현재 요청 큐 크기 (Gauge)
   - `kipris_token_bucket_tokens`: 토큰 버킷의 현재 가용 토큰 수 (Gauge)
   - `kipris_active_workers`: 현재 활성 워커 수 (Gauge)

### 메트릭 서버 구성
- 각 IPR 모드별 독립적인 메트릭 포트 할당:
  - 특허/실용신안: 8000
  - 디자인: 8001
  - 상표: 8002
  - 출원인번호: 8003
  - 목업서버: 8004
  - 데이터베이스: 8005

## 네트워크 최적화

### 비동기 요청 처리
1. **aiohttp 기반 구현**
   - 비동기 HTTP 클라이언트로 동시 요청 처리
   - 최대 동시 연결 수: 5 (MAX_CONNECTIONS_LIMIT)
   - TCP 연결 재사용으로 오버헤드 감소

2. **토큰 버킷 알고리즘**
   - 초당 토큰 생성 수: 20 (TOKENS_PER_SECOND)
   - 최대 토큰 수: 10 (MAX_TOKENS)
   - 버스트 트래픽 방지

### 성능 튜닝
1. **워커 설정**
   - 워커 수: 5 (WORKER_COUNT)
   - 워커 간 간격: 0.11초 (INTERVAL)
   - 메모리 사용량 최적화

2. **로깅 시스템**
   - 일별 로그 로테이션 (매일 00:00)
   - 1주일 로그 보관
   - 조직 유형 및 IPR 모드별 로그 분리
   - 상세한 로그 포맷: {time} | {level} | {org_type} | {ipr_mode} | {message}
