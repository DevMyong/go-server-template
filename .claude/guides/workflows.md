# Command 체이닝 워크플로우 상세 가이드

이 문서는 CLAUDE.md에 정의된 커맨드 체이닝의 상세 버전입니다.

---

## 📌 워크플로우 패턴 상세

### 1️⃣ 새 기능 추가

**체인**: `/plan` → `/implement` → `/test` → `/verify`

**트리거 키워드**:
- 한글: 추가, 만들어, 구현, 생성, 새로운
- 영어: create, add, implement, build, new

**적용 상황**:
- 새 API 엔드포인트
- 새 서비스/레포지토리 레이어
- 새 미들웨어
- 새 기능 모듈

**단계별 상세**:

#### Step 1: /plan

- 요구사항 분석
- 파일 구조 설계
- 인터페이스 정의
- 데이터 흐름도
- 예상 소요 시간
- **출력**: 구현 계획서 (코드 없음)
- **대기**: 사용자 승인 ("OK", "좋아", "ㄱㄱ")

#### Step 2: /implement

- 계획 기반 단계별 구현
- 한 파일씩 작업
- 각 파일 완료 후 "✓ 완료" 표시
- **대기**: 각 주요 단계마다 확인 (선택적)

#### Step 3: /test

- 테스트 파일 생성
- 테이블 드리븐 테스트
- Success/Failure/Edge 케이스
- `go test -v` 실행
- **조건**: 모든 테스트 통과 시 다음 단계

#### Step 4: /verify

- 코드 품질 체크
- 아키텍처 준수 확인
- 프로덕션 준비도 평가
- 개선사항 제안
- **출력**: ✓/⚠️/❌ 체크리스트

**예시 대화**:
```
사용자: "payment API 추가해줘"
Claude: "payment API를 다음 순서로 만들게요:
/plan → /implement → /test → /verify
[/plan 실행 중...]

구현 계획
1. 파일 구조
- internal/handler/payment_handler.go
- internal/service/payment_service.go
- internal/repository/payment_repository.go
- internal/model/payment.go

2. API 엔드포인트
- POST /api/v1/payments
- GET  /api/v1/payments/:id

3. 구현 순서
- model 정의
- repository 인터페이스 & 구현
- service 비즈니스 로직
- handler 라우팅

계획이 괜찮으면 구현 시작할게요!"
```

---
### 2️⃣ TDD 방식 개발

**체인**: `/plan` → `/tdd` → `/verify`

**트리거 키워드**:
- TDD로, 테스트 먼저, 테스트 주도로, test-first, test-driven

**적용 상황**:
- 복잡한 비즈니스 로직
- 버그가 치명적인 기능 (결제, 인증)
- 레거시 코드 리팩토링
- 명확한 입출력 사양

**Red-Green-Refactor 사이클**:

#### Red (실패)
```go
// user_service_test.go
func TestUserService_CreateUser(t *testing.T) {
// Given: 준비
// When: 실행
// Then: 검증 (실패 예상)
}
```
→ go test 실행 → 당연히 실패 (구현 안 됨)

#### Green (통과)
```go
// user_service.go
func (s *UserService) CreateUser(...) error {
// 최소한의 구현 (일단 통과만)
return nil
}
```
-> `go test` 실행 → 통과

#### Refactor (개선)
```go
// user_service.go
func (s *UserService) CreateUser(...) error {
    // 중복 제거, 코드 개선
    // 테스트는 계속 통과해야 함
}
```
→ go test 실행 → 여전히 통과

**주의사항**:
- 각 사이클마다 반드시 테스트 실행
- Green에서 "최소 구현"만 할 것
- Refactor 시 기능 변경 금지

---
### 3️⃣ 빠른 수정/작은 변경
**체인**: /implement → /test

**트리거 키워드**:
- 수정, 고쳐, 변경, 바꿔, fix, change, update, modify

**적용 상황**:
- 단순 버그 수정
- 설정값 변경
- 로그 메시지 수정
- 타입 변경
- 간단한 로직 수정

**생략 가능한 경우 (5줄 이하)**:
- 오타 수정
- import 추가
- 주석 수정
- 상수값 변경

**예시**:
```
사용자: "DB 타임아웃을 5초로 수정해줘"

Claude: "간단한 수정이라 바로 진행할게요.

[/implement]
pkg/database/mysql.go
- Timeout: 3 * time.Second
+ Timeout: 5 * time.Second

[/test]
기존 테스트 실행... 통과 ✓"
```

---
### 4️⃣ 리팩토링
**체인**: /plan → /implement → /verify

**트리거 키워드**:
- 리팩토링, 개선, 정리, 클린업, refactor, clean up, improve

**적용 상황**:
- 중복 코드 제거
- 함수 분리
- 인터페이스 추출
- 네이밍 개선
- 구조 재정렬

**원칙**:
- 기능 변경 없음 (동작은 동일)
- 기존 테스트가 모두 통과해야 함
- /test 단계 생략 가능

**체크사항**:
```
Before Refactor:
✓ 모든 테스트 통과 확인

During Refactor:
✓ 기능 동일성 유지
✓ 단계별 검증

After Refactor:
✓ 모든 테스트 여전히 통과
✓ 코드 복잡도 감소 확인
```

---
### 5️⃣ 탐색/분석
**체인**: 커맨드 없음 (직접 분석)
**트리거 키워드**:
- 어떻게, 설명, 분석, 파악, 이해, explain, how, what, analyze

**적용 상황**:
- 코드베이스 파악
- 버그 원인 추적
- 구조 이해
- 의존성 파악

**실행 순서**:
1. 관련 파일 식별
2. 파일 읽기
3. 구조 분석
4. 시각화 (다이어그램)
5. 설명 작성

**출력 형식**:
```markdown
## 분석: User Service 구조

### 파일 구조
- handler: HTTP 요청 처리
- service: 비즈니스 로직
- repository: DB 접근

### 데이터 흐름
HTTP Request 
→ Handler (검증)
→ Service (로직)
→ Repository (DB)
→ 응답 역순

### 주요 함수
- CreateUser: 사용자 생성
- GetUser: 조회
...
```

---
### 6️⃣ 검증만
**체인**: /verify 단독
**트리거 키워드**:
- 검증, 확인, 체크, 리뷰, verify, check, review, audit

**적용 상황**:
- 코드 리뷰
- 프로덕션 배포 전
- 보안 감사
- 성능 체크

**검증 항목**:
```
코드 품질:
□ 에러 핸들링
□ 메모리 누수
□ Goroutine 안전성
□ 컨텍스트 타임아웃

아키텍처:
□ 의존성 방향
□ 레이어 분리
□ 인터페이스 사용

프로덕션:
□ 로깅
□ 환경변수
□ Graceful shutdown
```
**후속 조치**:
⚠️ 발견 시: "수정할까요?" 제안
❌ 발견 시: 즉시 수정 권고

--- 
### 7️⃣ 대규모 기능/모듈
**체인**: /plan → 반복 (각 컴포넌트별 ①~④)
**트리거 키워드**:
- 전체, 처음부터, 새 서비스, 모듈, from scratch, entire

**적용 상황**:
- 완전히 새로운 도메인
- 마이크로서비스 추가
- 대규모 기능 세트

**실행 전략**:
```markdown
1. /plan (전체 구조)
   ├─ 도메인 모델 정의
   ├─ 컴포넌트 분리
   └─ 구현 순서 결정

2. 컴포넌트별 구현 (순차)
   ├─ Model: /plan → /implement → /test
   ├─ Repository: /plan → /tdd → /verify
   ├─ Service: /plan → /tdd → /verify
   └─ Handler: /plan → /implement → /test

3. 통합 테스트
   └─ /test (e2e)

4. 최종 검증
   └─ /verify (전체)
```

**예시**:
```markdown
사용자: "주문 관리 모듈을 처음부터 만들어줘"

Claude: "주문 관리 모듈 전체를 만들겠습니다.

[/plan - 전체 구조]
## 컴포넌트
1. Order Model
2. Order Repository
3. Order Service
4. Order Handler
5. Payment Integration

## 구현 순서
Model → Repository → Service → Handler

각 컴포넌트마다 적절한 워크플로우를 적용할게요.

---

[1/5] Order Model 작업 시작
/plan → /implement → /test

[2/5] Order Repository 작업 시작
/plan → /tdd → /verify

... (반복)"
```

---
## 🔄 체이닝 실행 규칙
### 자동 진행 조건

### 중단 조건
- 사용자: "잠깐", "멈춰", "stop", "wait"
- 테스트 실패 시
- /verify에서 ❌ 이슈 발견 시
- 빌드 실패 시

### 에러 복구
```markdown
테스트 실패
→ 에러 로그 분석
→ 원인 파악
→ /implement로 수정
→ /test 재실행
→ 통과 시 계속 진행
```

---
## 💡 명시적 체이닝
사용자가 직접 순서를 지정하면 무조건 그대로 따름:
```database 패키지를 /verify → /plan → /implement 순서로 해줘"```
→ 비록 비효율적이어도 지시대로 실행

---

## 🎯 워크플로우 선택 로직
```markdown
사용자 요청
    ↓
[1] 키워드 분석
    ↓
[2] 복잡도 판단
    - 5줄 이하: 커맨드 생략
    - 단순 수정: ③
    - 중간 복잡도: ①
    - 높은 복잡도: ② (TDD)
    - 대규모: ⑦
    ↓
[3] 워크플로우 선택
    ↓
[4] 사용자에게 알림
    "이 워크플로우로 진행할게요: ..."
    ↓
[5] 실행
```
