# Go 코딩 스타일 가이드

이 문서는 CLAUDE.md의 코딩 규칙을 상세히 설명합니다.

---

## 네이밍 규칙

### 파일명
- **규칙**: `snake_case`
- **패턴**: `{entity}_{layer}.go`
```markdown
✅ Good
- user_handler.go
- user_service.go
- user_repository.go
- payment_processor.go
❌ Bad
- UserHandler.go
- userHandler.go
- user.go (모호함)
```

### 패키지명
- **규칙**: 소문자, 단수형, 짧게
- **금지**: 밑줄, 대문자, 복수형