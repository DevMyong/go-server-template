---
name: go-service
description: Go 마이크로서비스 전체 구조 생성 (템플릿)
---

# Go 서비스 생성

재사용 가능한 Go 서비스를 처음부터 생성합니다.

## 실행 단계

### Phase 1: 프로젝트 구조
1. 디렉토리 구조 생성 (Clean Architecture)
2. go.mod 초기화
3. 각 디렉토리에 README.md

### Phase 2: 핵심 인프라
1. `/plan` - 인프라 계획 수립
2. `/implement` - 순서대로 구현
    - pkg/logger (uber/zap)
    - internal/config (viper)
    - pkg/database (MySQL, MongoDB, Redis)

### Phase 3: 미들웨어
1. `/plan` - 미들웨어 설계
2. `/implement`
    - logging
    - recovery
    - cors
    - request-id

### Phase 4: 기본 핸들러
1. Health check 엔드포인트
2. 라우터 설정
3. main.go 통합

### Phase 5: Docker & Make
1. Dockerfile (multi-stage)
2. docker-compose.yml
3. Makefile

### Phase 6: 문서화
1. README.md 작성
2. .env.example
3. CLAUDE.md 업데이트

**각 Phase 완료 후 확인 요청**