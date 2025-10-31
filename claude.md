# Go Server Template

## 프로젝트
재사용 가능한 Go 1.25 서버 템플릿. 사이드 프로젝트 시작용.

## 기술 스택
- Go 1.25, MySQL (GORM), MongoDB, Redis (go-redis v9)
- 라이브러리: viper, uber/zap, validator, testify
- ❌ 금지: Gin/Echo/Fiber 등 웹 프레임워크

## 구조
```
cmd/api/           # 진입점
internal/          # handler, service, repository, model
pkg/              # database, logger, httpclient
```

**레이어**: Handler → Service → Repository
**의존성**: 내부→외부만, 역방향은 인터페이스

## 코딩 규칙
- 파일: `snake_case`, 패키지: 소문자 단수형
- 에러: 항상 래핑 `fmt.Errorf("...: %w", err)`
- 컨텍스트: 첫 번째 파라미터, 타임아웃 3초(일반)/10초(heavy)
- 구조체 태그 순서: `json → db → validate`
- 인터페이스: 사용하는 쪽에서 정의

## 명령어
```bash
make run          # 로컬 실행
make test         # 전체 테스트
make docker-up    # 로컬 DB 실행
```

## 커맨드 체이닝 (자동 선택)

### 패턴 자동 매칭
- 새 기능 (추가/만들어/구현): /plan → /implement → /test → /verify
- TDD (TDD로/테스트 먼저): /plan → /tdd → /verify
- 수정 (수정/고쳐/변경): /implement → /test
- 리팩토링 (리팩토링/개선): /plan → /implement → /verify
- 검증만 (검증/확인/체크): /verify

### 실행 규칙
- /plan 후 사용자 확인 대기
- 테스트 통과 시 자동 진행
- 5줄 이하 수정은 커맨드 생략 가능
- 명시적 지시 시 그대로 따름

상세: .claude/guides/workflows.md 참조

## 미들웨어 순서 (고정)
RequestID → Logger → Recovery → CORS → Auth → Handler

## 특이사항
- MongoDB 초기 연결 타임아웃 가능 (재시도 있음)
- GORM AutoMigrate 프로덕션 금지
- Graceful shutdown 30초 타임아웃

## 테스트
- 테이블 드리븐 패턴, testify/mock 사용
- 커버리지: 전체 80%, service 90%

---
**더 자세한 내용**:
- 워크플로우: `.claude/guides/workflows.md`
- 아키텍처: `.claude/guides/architecture.md`
- 코딩 스타일: `.claude/guides/coding-standards.md`