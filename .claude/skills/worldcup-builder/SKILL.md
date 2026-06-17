---
name: worldcup-builder
description: 2026 북중미 월드컵의 현황·하이라이트·일정을 모아 보여주는 FIFA풍 인터랙티브 웹페이지를 만드는 오케스트레이터. data-curator(데이터)→fifa-frontend-builder(구현)→design-director(미감)→qa-verifier(검증) 팀을 조율한다. "월드컵 페이지/사이트 만들어", "월드컵 현황 웹페이지", "경기 결과·하이라이트 페이지", 그리고 후속으로 "다시 실행", "재실행", "결과 업데이트", "일정 갱신", "하이라이트 추가", "디자인 수정", "조별 페이지만 다시", "이전 결과 기반 개선", "페이지 보완" 등 월드컵 페이지의 생성·갱신·수정 요청 시 반드시 이 스킬을 사용. 단순 사실 질문은 직접 응답 가능.
---

# worldcup-builder — 월드컵 페이지 오케스트레이터

2026 FIFA 북중미 월드컵의 현재 결과·예정 경기·하이라이트를 한 페이지에서 보여주는 **FIFA풍 인터랙티브 단일 HTML(`index.html`)**을 팀으로 만든다.

## 실행 모드: 하이브리드
- **Phase A (데이터 수집)**: 서브 에이전트 — `data-curator`가 웹에서 독립 수집.
- **Phase B (구현·검수·검증)**: 에이전트 팀 — `fifa-frontend-builder` + `design-director` + `qa-verifier`가 SendMessage로 조율하며 생성-검증 루프.
- 모든 Agent 호출은 `model: "opus"`.

## Phase 0: 컨텍스트 확인 (재실행 판별)
시작 시 기존 산출물을 확인해 모드를 정한다:
- `index.html` + `_workspace/` **미존재** → **초기 실행** (Phase A부터 전체)
- 존재 + 사용자가 **결과/일정/하이라이트 갱신** 요청 → **부분 재실행**: `data-curator`만 호출해 JSON 갱신 → `fifa-frontend-builder`가 데이터 블록만 교체 → `qa-verifier` 재검증
- 존재 + 사용자가 **디자인/인터랙션 수정** 요청 → **부분 재실행**: `design-director` 지시 → `fifa-frontend-builder` 수정 → `qa-verifier`
- 존재 + 사용자가 **새 입력/전면 재구성** 요청 → 기존 `_workspace/`를 `_workspace_prev/`로 이동 후 초기 실행

## Phase A: 데이터 수집 (서브 에이전트)
`data-curator`를 `model:"opus"`로 호출. `worldcup-data` 스킬의 스키마로 2026 월드컵 데이터를 수집·정제하여 `_workspace/01_data-curator_worldcup.json`에 저장하게 한다. 핵심: 치러진 경기는 실제 스코어·하이라이트, 예정 경기는 일정만(status=scheduled), 불확실 항목은 `_dataNotes`에 명시.

수집 완료 후 JSON의 조 수·경기 수·`_dataNotes`를 사용자에게 한 줄 요약 보고한다.

## Phase B: 구현 + 검수 + 검증 (에이전트 팀)
`TeamCreate`로 3인 팀을 구성하고 `TaskCreate`로 작업을 할당, 팀원은 `SendMessage`로 조율한다.

1. **fifa-frontend-builder**: `_workspace/01_data-curator_worldcup.json`을 `index.html` 상단 `const WORLD_CUP_DATA`로 임베드하고, `fifa-ui-theme` 스킬에 따라 데이터 주도 인터랙티브 페이지를 구현. status 3종(finished/live/scheduled) 분기, 필터 탭, 조별 순위표, 하이라이트 모달, 카운트다운 포함.
2. **design-director**: 초안을 FIFA 미감 기준으로 검수, 우선순위 매긴 수정 지시를 builder에 전달(SendMessage). AI 슬롭·위계·모션 점검.
3. **qa-verifier**: 데이터↔렌더 정합성·인터랙션·반응형을 점진적으로 검증, 결함을 builder에 전달. 통과 근거(카운트·스크린샷) 수집.

builder는 지시·결함을 반영해 수정, design-director와 qa-verifier가 재검수/재검증하는 생성-검증 루프를 통과·합의에 이를 때까지 반복한다.

## 데이터 전달 프로토콜
- **파일 기반(산출물)**: `_workspace/01_data-curator_worldcup.json` → `index.html`. 중간물은 `_workspace/` 보존.
- **태스크 기반(조율)**: `TaskCreate`/`TaskUpdate`로 진행·의존 관리.
- **메시지 기반(실시간)**: design-director↔builder↔qa-verifier 피드백 교환.
- **파일명 컨벤션**: `{phase}_{agent}_{artifact}.{ext}`.

## 에러 핸들링
- 단계 실패 시 1회 재시도, 재실패면 해당 산출 없이 진행하고 결과 보고에 누락 명시.
- 데이터 출처 상충은 삭제하지 말고 `_dataNotes`에 출처 병기.
- 데이터가 크게 부족하면(예: 조 편성 미확정) `worldcup-data` 스키마의 placeholder로 골격을 만들고, 화면에 "데이터 갱신 예정" 상태를 노출한다.

## 팀 크기
중규모(3~5명 협업): 구현 1 + 검수 1 + 검증 1 = 3인 집중 팀. 과도한 인원보다 집중을 우선.

## 완료 기준
- `index.html` 단일 파일이 브라우저에서 열리고 JS 콘솔 에러 없음.
- 데이터의 모든 경기/조가 화면에 렌더되고 status별 시각 분기 동작.
- 필터·순위표·하이라이트·카운트다운 인터랙션 동작.
- design-director 미감 합의 + qa-verifier 통과 근거 확보.

## 테스트 시나리오
- **정상 흐름**: 초기 실행 → data-curator가 JSON 생성 → 팀이 index.html 구현·검수·검증 → 통과. 사용자가 브라우저로 열어 탭 전환·하이라이트 확인.
- **에러 흐름(데이터 불확실)**: 조 편성/결과가 일부 미확정 → data-curator가 `_dataNotes`에 명시하고 placeholder로 채움 → builder가 "갱신 예정" 상태로 렌더 → qa-verifier가 placeholder도 깨지지 않는지 검증.
- **후속 흐름(결과 갱신)**: "조별리그 최신 결과 반영해줘" → Phase 0이 부분 재실행으로 분기 → data-curator가 scheduled→finished 경기 갱신·순위 재계산 → builder가 데이터 블록 교체 → qa-verifier 재검증.
