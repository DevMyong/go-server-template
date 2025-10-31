# Go Server Template

재사용 가능한 Go 서버 템플릿. 사이드 프로젝트를 빠르게 시작하기 위한 보일러플레이트입니다.

## 특징

- **레이어드 아키텍처**: Handler → Service → Repository
- **클린 코드**: 의존성 역전 원칙, 인터페이스 기반 설계
- **프레임워크 없음**: 표준 라이브러리 기반 (Gin/Echo/Fiber 미사용)
- **다중 데이터베이스**: MySQL, MongoDB, Redis 지원
- **테스트 우선**: 테이블 드리븐 패턴, 80%+ 커버리지 목표

## 기술 스택

- **언어**: Go 1.25
- **데이터베이스**: MySQL (GORM), MongoDB, Redis (go-redis v9)
- **라이브러리**: viper, uber/zap, validator, testify

## 프로젝트 구조

```
cmd/api/           # 애플리케이션 진입점
internal/
  ├── handler/     # HTTP 핸들러
  ├── service/     # 비즈니스 로직
  ├── repository/  # 데이터 액세스
  └── model/       # 도메인 모델
pkg/
  ├── database/    # DB 연결 (MySQL, MongoDB, Redis)
  ├── logger/      # 로깅 (zap)
  └── httpclient/  # HTTP 클라이언트
```

## 시작하기

> 🚧 **Work in Progress**: 핵심 인프라 구현 중입니다.

현재 프로젝트는 초기 설정 단계입니다. 다음 기능들이 곧 추가될 예정입니다:
- pkg/ 패키지 (logger, database, httpclient)
- 예제 API 엔드포인트
- Docker Compose 설정
- Makefile 명령어

## 개발 가이드

- **아키텍처**: `.claude/guides/architecture.md`
- **코딩 스타일**: `.claude/guides/coding-standards.md`
- **워크플로우**: `.claude/guides/workflows.md`

## 라이선스

MIT
