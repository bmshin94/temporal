# Temporal 전수조사 & 활용 전략 정리 (한국어)

> 이 문서는 `bmshin94/temporal` 저장소를 전수조사하고, 활용 방안과 수익화 전략까지
> 논의한 내용을 정리한 노트입니다.
> 작성일: 2026-09-29

## 🔗 관련 깃허브 주소

| 구분 | 주소 |
|---|---|
| 이 저장소 (포크) | https://github.com/bmshin94/temporal |
| 원본 프로젝트 | https://github.com/temporalio/temporal |
| Temporal 조직 | https://github.com/temporalio |
| Temporal CLI | https://github.com/temporalio/cli |
| Go 샘플 | https://github.com/temporalio/samples-go |
| Java 샘플 | https://github.com/temporalio/samples-java |
| PHP SDK | https://github.com/temporalio/sdk-php |
| TypeScript SDK | https://github.com/temporalio/sdk-typescript |
| Python SDK | https://github.com/temporalio/sdk-python |
| SDK Core (아키텍처 문서) | https://github.com/temporalio/sdk-core |
| 기능 제안 저장소 | https://github.com/temporalio/proposals |
| 공식 문서 | https://docs.temporal.io/ |
| 커뮤니티 포럼 | https://community.temporal.io |

---

## 1. 이 저장소의 정체

| 항목 | 내용 |
|---|---|
| 진짜 이름 | **Temporal** (temporalio/temporal) |
| 우리 저장소 | `bmshin94/temporal` (원본의 포크, 얕은 클론 51커밋) |
| 스타 / 포크 | ⭐ 23,359 / 🍴 1,943 (조사 시점) |
| 오픈 이슈 | 1,027개 |
| 언어 / 규모 | Go, 2,977개 파일 / 약 105만 줄 / 테스트 파일 969개 / proto 75개 |
| 라이선스 | MIT (상업적 사용·재배포 자유) |
| 정체 | **Durable Execution Platform** — 절대 중단되지 않는 워크플로 실행 서버 |
| 계보 | Uber의 `Cadence`를 포크 → 창시자들이 Temporal Technologies 창업 |

우리 저장소에는 원본 코드 외에 직접 추가한 커밋이 있음:
`docs: created CLAUDE.md persona guide` (PR #1 머지 완료)

---

## 2. 핵심 개념 (쉽게)

- **게임의 자동 세이브**: 한 걸음마다 DB에 기록 → 서버가 죽어도 그 지점부터 재개
- **절대 안 잊는 비서**: "결제 → 재고 → 문자 → 3일 뒤 후기 메일"을 중복 없이 끝까지 수행
- 원리: **이벤트 소싱(Event Sourcing)** + **리플레이(Replay)**
- 오빠 코드는 서버 안이 아니라 **별도 Worker 프로세스**에서 실행되고, gRPC 폴링으로 통신

---

## 3. 아키텍처 — 4개 서비스

| 서비스 | 역할 | 비유 |
|---|---|---|
| `service/frontend` | gRPC 게이트웨이, 인증/레이트리밋 | 주문받는 카운터 |
| `service/history` | 워크플로 상태·이벤트 히스토리를 샤드 단위 관리 (핵심) | 주문서 철 |
| `service/matching` | Task Queue 관리, 워커에게 일감 분배 | 주문 전달 창구 |
| `service/worker` | 아카이빙·스케줄·복제 등 내부 시스템 잡 | 설거지/청소 담당 |

---

## 4. 폴더 전수조사 결과

### 핵심 코드
- `api/` (34개 서브폴더) + `proto/` (75개 .proto) — gRPC API 정의 및 생성 코드
- `common/` (81개 서브폴더) — 공통 라이브러리
  - `persistence/` DB 추상화, `dynamicconfig/` 무중단 설정 변경,
    `membership/` 클러스터 멤버십, `metrics/`, `nexus/`, `namespace/`
- `chasm/` — Chasm(Coordinated Heterogeneous Application State Machines), 차세대 범용 상태머신 엔진
- `client/` — 서비스 간 내부 통신 클라이언트
- `cmd/server/main.go` — 서버 엔트리포인트

### 운영 / 배포
- `schema/` — **Cassandra / MySQL / PostgreSQL / SQLite / Elasticsearch** 스키마
- `config/` — 개발용 설정 20여 개 (sqlite, postgres-es, cluster-a/b/c(XDC 복제), jwt 등)
- `docker/`, `.goreleaser.yml` — 이미지/바이너리 릴리스 자동화

### 개발자 도구
- `tools/tdbg` — 워크플로 히스토리·DLQ 디버깅 CLI
- `tools/flakereport`, `optimize-test-sharding`, `testrunner` — 깨지는 테스트 리포팅 및 테스트 샤딩 최적화
- `cmd/tools/*` — protoc 플러그인, 동적설정/RPC 래퍼 코드 생성기 등

### 문서 (보물)
- `docs/architecture/` — history/matching 서비스, workflow-lifecycle, nexus, chasm,
  retry, speculative-workflow-task 등 설계 문서 15개 + SVG/D2 다이어그램
- `docs/development/` — 빌드·테스트·TLS·트레이싱 가이드
- `CONTRIBUTING.md` — 빌드/테스트 방법 (CLA 서명 필요)

### AI 에이전트 관련 설정 (트렌드 포인트)
| 파일 | 정체 |
|---|---|
| `CLAUDE.md` | 직접 추가한 페르소나 가이드 |
| `AGENTS.md` | Temporal 팀이 넣은 AI 에이전트용 개발 규칙서 (lint/테스트/주석 규칙까지) |
| `.claude/skills/review/SKILL.md` | Claude Code 코드리뷰 스킬 |
| `.cursor/rules/review.mdc` | Cursor IDE 룰 |
| `.github/copilot-instructions.md` | Copilot 지침 |
| `.github/workflows/claude-review-teams.yml` | CI에서 AI가 PR 자동 리뷰 |

> 참고: 저장소 전체에서 "MCP / Model Context Protocol" 문자열은 **검색 결과 없음**.

---

## 5. 언제 쓰나

- 💳 결제·정산 (Saga / 보상 트랜잭션)
- 📦 주문·물류 파이프라인 (며칠~몇 달 걸리는 프로세스)
- 🚀 인프라 프로비저닝 / 배포 오케스트레이션
- ⏰ 분산 크론·스케줄러 (정확히 한 번 실행)
- 🙋 휴먼 인 더 루프 (승인 대기)
- 🤖 **AI 에이전트 오케스트레이션** (최근 급부상)

---

## 6. Q&A 정리

### Q1. 설치 및 사용법
```bash
# (A) 그냥 써보기 — 소스 빌드 불필요
brew install temporal
temporal server start-dev        # 서버 :7233 / Web UI http://localhost:8233
temporal workflow list

# (B) 서버 소스 직접 빌드 (이 저장소)
make                     # 최초 1회: 의존성 설치 + 전체 빌드
make bins                # 빌드만
make start-dependencies  # Cassandra/MySQL/ES/UI를 docker compose로
make unit-test           # 유닛 테스트
make lint-code-fast      # 커밋 전 빠른 린트
```
- 필요: Go(go.mod 기준 1.27+), Docker, (proto 수정 시) protobuf compiler
- 앱 개발은 SDK(Go/Java/TS/Python/.NET/PHP/Ruby)로 Worker + Client 작성

### Q2. 플러그인? 스킬? MCP?
**전부 아님 — 독립 실행되는 백엔드 인프라 서버**(Postgres/Kafka 급).
저장소 안의 `.claude/skills`, `.cursor/rules`, `copilot-instructions.md`는
"이 서버를 개발할 때 쓰는 AI 보조 설정"이지 Temporal 자체가 아님.

### Q3. API 토큰 필요?
| 상황 | 인증 |
|---|---|
| `temporal server start-dev` | 필요 없음 |
| 셀프호스팅 | 기본 없음. 옵션으로 JWT(`config/jwt/`, `development-jwt.yaml`) / mTLS(`docs/development/tls/tls.md`) |
| Temporal Cloud | API Key 또는 mTLS 인증서 필요 |
| Temporal 자체가 외부 토큰 요구 | ❌ 없음 (셀프컨테인드) |

※ LLM API 키는 Activity 코드가 쓰는 것이며 Temporal 서버와 무관.

### Q4. 왜 유명한가
1. Durable Execution 카테고리의 대표 주자 (Cadence 계보의 정통성)
2. 분산 환경 상태관리·재시도·exactly-once라는 난제를 평범한 코드로 해결
3. Netflix, Snap, Stripe, Datadog, Coinbase 등 대형 레퍼런스
4. MIT + 완전 셀프호스팅 → 벤더 락인 없음
5. 7개 언어 공식 SDK
6. AI 에이전트 붐과 유즈케이스가 정확히 일치
7. Web UI 기반의 뛰어난 디버깅 경험
8. 105만 줄 규모에 테스트·CI 품질이 매우 높음

### Q5. 로컬 에이전트 구축에 도움?
**매우 도움 됨.**

| 에이전트의 문제 | Temporal의 해결 |
|---|---|
| LLM 429/타임아웃/5xx | Activity 자동 재시도 + 지수 백오프 |
| 중간에 프로세스 죽음 | 이벤트 리플레이로 죽은 지점부터 재개 |
| 사용자 승인 대기 | Signal/Update로 며칠도 대기 가능 |
| 툴 호출·비용 추적 | 히스토리 자동 기록 + Web UI 시각화 |
| 중복 실행 | Workflow ID 기반 중복 방지 |
| 병렬 에이전트 | Child Workflow / 병렬 Activity |
| 주기 실행 | Schedule 내장 |

권장 구조: `Workflow`(두뇌, 결정론적) → `Activity`(LLM·툴·IO 등 부작용 전부) + `Signal`(사람 개입) + `Query`(진행상황 조회)

주의: 간단한 챗봇엔 오버킬. Workflow 코드에 `random()`, `now()`, 직접 HTTP 호출 금지(리플레이 깨짐).
판단 기준 — **30분 이상 / 실패 시 손실 큼 / 사람 승인 있음 / 다중 에이전트 협업** 중 하나면 도입 가치 있음.

### Q6. React나 PHP로 만들 수 있나?
- ❌ **서버 자체 재구현**: 비현실적 (Go 105만 줄, 샤딩·복제·DB 5종). 게다가 MIT라 공짜로 쓰면 됨
- ✅ **Temporal을 쓰는 앱**: 완전 가능
  - PHP: **공식 SDK 있음** (`temporalio/sdk-php`, RoadRunner 기반) → Laravel/Symfony와 조합
  - TypeScript/Node: 공식 SDK, Next.js 백엔드에서 워커 실행
  - React: 브라우저에서 gRPC 직접 호출 불가 → `React → REST → 백엔드 → Temporal Client`
  - Python: AI/LLM 조합 최강

```
[React 프론트] 대시보드 / 승인 UI / 실시간 진행률
      ↓ REST
[PHP(Laravel) or Node 백엔드] Temporal Client (start / signal / query)
      ↓ gRPC
[Temporal 서버] ← 그냥 실행만, 직접 만들지 않음
      ↑ 폴링
[Worker 프로세스] 실제 비즈니스 로직 + LLM 호출
```

---

## 7. 수익화 아이디어

> 대전제: 엔진으로 Temporal Cloud와 경쟁 ❌ → **Temporal 위에 얹는 얇은 레이어**로 승부 ⭕
> MIT라 상업적 재판매·SaaS화 가능. 단, 제품명에 "Temporal" 상표 사용은 지양 ("Powered by Temporal" 표기 권장).

### 티어 1 — React/PHP 스택에 가장 적합
1. **AI 에이전트 워크플로 빌더 SaaS** — React Flow 캔버스 + Temporal 실행 엔진.
   경쟁사 대비 차별점은 "실행 중 서버가 죽어도 유실 없음". 월정액 + 실행량 과금.
   난이도 ★★★★☆ / 시장성 ★★★★★
2. **버티컬(업종 특화) 자동화 솔루션 — 최우선 추천 🏆**
   병원 예약·노쇼·보험청구 / 이커머스 정산·역물류 / HR 온보딩 / 건설·제조 공정.
   기간이 길고 승인 단계가 많은 업종일수록 Temporal 가치 큼. 구독 + 구축비.
   난이도 ★★★☆☆ / 시장성 ★★★★★
3. **Laravel/Symfony ↔ Temporal 통합 패키지 (오픈코어)**
   무료 OSS + 유료 Pro(관리 패널, 멀티테넌시, 모니터링). PHP 생태계 선점 가능.
   난이도 ★★☆☆☆ / 시장성 ★★★☆☆ — **첫 프로젝트로 적합**

### 티어 2 — 도구·서비스형
4. **옵저버빌리티 / 코스트 대시보드** — 성공률·p95·**LLM 토큰 비용 집계**·SLA 알림. 난이도 ★★★☆☆
5. **매니지드 호스팅 (틈새)** — 국내 리전·한국어 지원·원화 결제·규제 대응. 난이도 ★★★★★
6. **컨설팅 / 마이그레이션 / 교육** — 현금화 가장 빠름. Celery·Cron 지옥 → Temporal 전환 컨설팅,
   한국어 강의·템플릿 판매 (한국어 콘텐츠 희소). 난이도 ★☆☆☆☆ — **즉시 시작 가능**

### 티어 3 — 제품에 녹이기
7. **장기실행 AI 에이전트 서비스** — 며칠짜리 딥리서치, RFP 대응 파이프라인,
   코드 마이그레이션 에이전트. "안 죽는다"가 마케팅 포인트. 난이도 ★★★★☆ / 시장성 ★★★★★
8. **워크플로 템플릿 마켓플레이스** — Saga 결제·구독 갱신·온보딩 템플릿 판매. 난이도 ★★☆☆☆

### 추천 로드맵
```
1~2개월차 : 아이디어 6(컨설팅·교육)으로 현금흐름 + 아이디어 3(OSS 패키지)로 인지도
3~6개월차 : 아이디어 2(버티컬 SaaS) MVP — React + PHP/Node + 셀프호스팅 Temporal
6개월~    : 아이디어 1/4로 확장 또는 아이디어 7로 피벗
```

**공식:** Temporal(공짜 엔진) + React/PHP(UI·도메인) + 특정 업종의 아픈 문제 = 수익
파는 것은 "Temporal"이 아니라 **"절대 끊기지 않는 자동화"라는 가치**.
