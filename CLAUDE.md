# worldcup

## 하네스: 2026 북중미 월드컵 인터랙티브 페이지

**목표:** 2026 FIFA 북중미 월드컵의 현재 결과·예정 경기·하이라이트를 모아 보여주는 FIFA풍 인터랙티브 단일 HTML(`index.html`)을 데이터 주도로 생성·갱신한다.

**트리거:** 월드컵 페이지의 생성·갱신·수정(결과/일정/하이라이트 반영, 디자인 수정, 재실행 포함) 요청 시 `worldcup-builder` 스킬을 사용하라. 단순 사실 질문은 직접 응답 가능.

**산출물:** `index.html`(최종), `_workspace/01_data-curator_worldcup.json`(데이터 원본·갱신 대상).

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-06-17 | 초기 구성 (data-curator·fifa-frontend-builder·design-director·qa-verifier + worldcup-data·fifa-ui-theme·worldcup-builder) | 전체 | - |
| 2026-06-17 | 초기 실행: 실데이터 수집 후 index.html 생성·QA 통과 | index.html, _workspace/ | 사용자 최초 요청 |
| 2026-06-17 | 에이전트 frontmatter `tools: All tools` 제거(필드 생략=전체 도구 상속) | agents/*.md | 스폰된 서브에이전트가 파일 도구 미부여로 빌드 불가했던 드리프트 수정 |
| 2026-06-17 | v2 반영: 팀명 한글화, 모든 시각 KST 변환, 다음경기 동시다발 표시, 가짜 LIVE 뱃지 제거, 종료경기 유튜브 하이라이트 링크, 토너먼트 로드맵 섹션 추가 | index.html, skills/fifa-ui-theme, skills/worldcup-data, _workspace | 사용자 피드백 |
