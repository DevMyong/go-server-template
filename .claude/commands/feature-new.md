---
name: feature
description: 새 기능 추가 전체 플로우 (plan→tdd→verify)
---

# 새 기능 추가 전체 플로우

1. `/plan` 실행
    - 기능 요구사항 분석
    - 설계 방안 수립

2. 계획 승인 후 `/tdd` 실행
    - Red-Green-Refactor 사이클
    - 모든 테스트 통과까지

3. 구현 완료 후 `/verify` 실행
    - 코드 품질 검증
    - 프로덕션 준비도 확인

4. 모든 단계 통과 시
    - Git commit 메시지 제안
    - PR 설명 생성

**각 단계는 이전 단계 성공 시에만 진행**