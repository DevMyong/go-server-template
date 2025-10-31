---
name: test
description: 구현된 코드에 대한 테스트 작성
---

# 테스트 코드 작성

구현된 코드에 대해 테스트를 작성합니다.

## 테스트 규칙

1. **파일명 규칙**
    - `{package}_test.go`
    - 같은 디렉토리에 위치

2. **testify 사용**
    - `testify/assert` 또는 `testify/suite`
    - 모킹 필요 시 `testify/mock`

3. **테이블 드리븐 테스트**
    ```go
       tests := []struct {
           name    string
           input   X
           want    Y
           wantErr bool
       }{
       // 테스트 케이스들
       }
    ```

4. **커버리지**
    - 성공 케이스
    - 실패 케이스 (에러 케이스)
    - Edge 케이스

5. **테스트 실행**
    - 작성 후 go test -v 실행
    - 결과 출력

모든 테스트가 통과해야 다음 단계 진행 가능