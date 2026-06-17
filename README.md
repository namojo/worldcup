# ⚽ FIFA 월드컵 2026 북중미 LIVE 센터

2026 FIFA 북중미 월드컵(미국·캐나다·멕시코)의 현황·결과·일정·하이라이트를 한 페이지에서 보는 FIFA풍 인터랙티브 웹페이지.

**🔗 라이브:** https://namojo.github.io/worldcup/

## 특징
- 경기 결과·예정 경기·조별 순위·득점왕·토너먼트 로드맵
- status별 카드 분기(종료/예정/LIVE), 필터, 하이라이트 토글, 실시간 카운트다운
- 모든 시각 **한국시간(KST)**, 팀명 **한글** 표기
- 종료 경기 **유튜브 하이라이트** 링크
- 데이터 주도: `index.html` 상단 `WORLD_CUP_DATA` 블록만 고치면 화면 자동 갱신

## 자동 갱신
매일 18:00 KST에 클라우드 루틴(`/schedule`)이 최신 결과를 반영해 `index.html`을 갱신하고 push → GitHub Pages로 자동 배포.

## 구조
- `index.html` — 단일 파일 페이지(최종 산출물)
- `_workspace/01_data-curator_worldcup.json` — 데이터 원본
- `.claude/` — 페이지를 생성·갱신하는 에이전트 하네스(에이전트 4 + 스킬 3)
- `CLAUDE.md` — 하네스 트리거 포인터 + 변경 이력

## 수동 갱신
Claude Code에서 `"월드컵 결과 업데이트해줘"` → `worldcup-builder` 하네스가 데이터 조사 후 페이지 갱신.
