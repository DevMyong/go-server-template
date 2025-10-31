# 브랜치 관리 전략

## 기본 전략: GitHub Flow

단순하고 명확한 GitHub Flow를 사용합니다.

```
main (protected)
  ├── feat/logger-package
  ├── feat/mysql-connection
  └── fix/logger-initialization
```

## 브랜치 네이밍 규칙

### 패턴
`<type>/<description>`

### 타입
- `feat/` - 새 기능 추가
- `fix/` - 버그 수정
- `refactor/` - 리팩토링 (동작 변경 없음)
- `chore/` - 빌드, 설정, 의존성 등
- `docs/` - 문서 작업
- `test/` - 테스트 추가/수정

### 설명 (description)
- 소문자, kebab-case
- 명확하고 간결하게
- 2-4 단어 권장

### 예시
```
feat/logger-package
feat/mysql-connection
feat/user-authentication
fix/redis-timeout-issue
fix/null-pointer-handler
refactor/service-layer
chore/add-makefile
chore/update-dependencies
docs/api-documentation
test/repository-layer
```

## 워크플로우

### 1. Issue 생성
```
Title: [Infrastructure] Implement logger package
Labels: infrastructure, pkg
```

### 2. 브랜치 생성 및 작업
```bash
# main에서 최신 상태 확인
git checkout main
git pull

# 새 브랜치 생성
git checkout -b feat/logger-package

# 작업 후 커밋
git add .
git commit -m "feat: implement zap logger initialization"
git commit -m "test: add logger package tests"
```

### 3. Push 및 PR 생성
```bash
# 원격에 push
git push -u origin feat/logger-package

# PR 생성 (gh CLI)
gh pr create \
  --title "[Infrastructure] Add logger package" \
  --body "Closes #1

## Changes
- Implement zap logger wrapper
- Add dev/prod mode configuration
- Add unit tests

## Test Plan
- [x] Unit tests pass
- [x] Manual testing in dev mode"
```

### 4. 리뷰 및 머지
```bash
# PR 머지 후 (GitHub UI 또는 CLI)
gh pr merge 1 --squash

# 로컬 정리
git checkout main
git pull
git branch -d feat/logger-package
```

## 커밋 컨벤션

### 포맷
```
<type>: <subject>

[optional body]
```

### 타입
- `feat:` - 새 기능
- `fix:` - 버그 수정
- `refactor:` - 리팩토링
- `test:` - 테스트
- `docs:` - 문서
- `chore:` - 기타 (빌드, 설정 등)

### 예시
```bash
feat: implement zap logger initialization
fix: handle nil pointer in user service
refactor: extract validation logic to separate function
test: add table-driven tests for repository
docs: update API documentation
chore: add Makefile with common commands
```

## PR 규칙

### 제목 형식
`[Category] Brief description`

**예시**:
- `[Infrastructure] Add logger package`
- `[Feature] Implement user authentication`
- `[Fix] Resolve Redis connection timeout`
- `[Refactor] Improve error handling in service layer`

### 본문 필수 항목
1. **Closes #N** - 관련 이슈
2. **Changes** - 변경 사항 요약
3. **Test Plan** - 테스트 체크리스트

### PR 체크리스트
- [ ] 테스트 통과 (`make test`)
- [ ] 코딩 스타일 준수
- [ ] 관련 문서 업데이트
- [ ] 커밋 메시지 컨벤션 준수

## main 브랜치 보호 (권장)

GitHub 저장소 설정:
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. 옵션:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass (CI 추가 후)
   - ✅ Do not allow bypassing the above settings

## 브랜치 정리

### 머지된 브랜치 삭제
```bash
# 로컬 브랜치 삭제
git branch -d feat/logger-package

# 원격 브랜치 삭제 (필요시)
git push origin --delete feat/logger-package

# 머지된 모든 로컬 브랜치 삭제
git branch --merged main | grep -v "main" | xargs git branch -d
```

### 작업 중 브랜치 동기화
```bash
# main의 최신 변경사항 가져오기
git checkout main
git pull

# 작업 브랜치로 돌아가서 rebase
git checkout feat/my-feature
git rebase main

# 충돌 해결 후
git push -f origin feat/my-feature
```

## 실전 예시: 1단계 pkg/ 구현

```
Issue #1: Logger 패키지
  └── feat/logger-package → PR #1 → merge

Issue #2: MySQL 연결
  └── feat/mysql-connection → PR #2 → merge

Issue #3: MongoDB 연결
  └── feat/mongodb-connection → PR #3 → merge

Issue #4: Redis 연결
  └── feat/redis-connection → PR #4 → merge

Issue #5: HTTP Client
  └── feat/http-client → PR #5 → merge
```

각 PR은 독립적으로 리뷰/머지 가능하며, main은 항상 안정적인 상태를 유지합니다.
