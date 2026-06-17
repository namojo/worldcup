---
name: worldcup-data
description: 2026 북중미 월드컵 데이터 수집과 표준 JSON 스키마. 조 편성·일정·결과·하이라이트·순위·득점왕을 모으거나, 기존 데이터를 최신 결과로 갱신할 때 사용. "월드컵 데이터", "경기 결과 업데이트", "일정 갱신", "하이라이트 추가", "조 편성 채우기" 등 데이터 작업 시 반드시 이 스킬을 사용. 화면 디자인이 아니라 데이터 구조·수집이 핵심일 때.
---

# worldcup-data — 월드컵 데이터 수집 & 스키마 표준

2026 FIFA 북중미 월드컵 데이터를 수집·정제하여 프론트엔드가 그대로 임베드할 수 있는 단일 JSON으로 만든다. data-curator 에이전트의 작업 지침이자, 화면과 데이터가 맞물리게 하는 **계약(contract)**이다.

## 왜 스키마가 계약인가
프론트엔드는 이 JSON을 `const WORLD_CUP_DATA`로 그대로 임베드하고 키 이름으로 화면을 그린다. 키가 어긋나면 곧 빈 화면·NaN·깨진 카드다. 따라서 아래 스키마를 **엄격히** 따른다. 필드를 못 채우면 임의값 대신 `null`을 쓰고 `_dataNotes`에 사유를 남긴다.

## 수집 절차
1. **대회 개요** — 개최국, 기간, 포맷(48개국·12개 조·각 조 상위 2팀+성적 우수 3위 8팀이 32강 진출), 개최 도시·경기장.
2. **조 편성** — 12개 조(A~L) 각 4팀. 팀명·국가코드(FIFA 3자, 예 USA/MEX/CAN/BRA)·국기 이모지.
3. **일정·결과** — 조별리그부터 결승까지 경기. 치러진 경기는 실제 스코어·하이라이트, 예정 경기는 일정만(스코어 `null`, status `scheduled`).
4. **순위표** — 각 조 standings(경기수·승·무·패·득·실·승점). 가능하면 결과로부터 직접 계산해 일관성 확보.
5. **하이라이트·스토리라인** — 경기별 핵심 장면 불릿 + 대회 전체 스토리라인.
6. **득점왕** — topScorers.
7. **교차검증** — fifa.com / espn.com / wikipedia 등 2곳 이상 대조. 시점은 "오늘" 기준(대회 진행 중일 수 있음).

## 표준 JSON 스키마
```json
{
  "lastUpdated": "2026-06-17",
  "tournament": {
    "name": "FIFA World Cup 2026",
    "hosts": ["United States", "Canada", "Mexico"],
    "dates": "2026-06-11 ~ 2026-07-19",
    "format": "48개국, 12개 조, 32강 토너먼트",
    "teamCount": 48,
    "venues": [{"city": "...", "country": "...", "stadium": "..."}]
  },
  "groups": [
    {
      "name": "A",
      "teams": [{"name": "Mexico", "code": "MEX", "flag": "🇲🇽"}],
      "standings": [
        {"team": "Mexico", "code": "MEX", "played": 1, "won": 1, "drawn": 0, "lost": 0, "gf": 2, "ga": 0, "points": 3}
      ]
    }
  ],
  "matches": [
    {
      "id": 1, "date": "2026-06-11", "time": "20:00", "stage": "Group A",
      "home": "Mexico", "homeCode": "MEX", "homeFlag": "🇲🇽",
      "away": "...", "awayCode": "...", "awayFlag": "...",
      "homeScore": 2, "awayScore": 0,
      "venue": "Estadio Azteca, Mexico City",
      "status": "finished",
      "highlights": ["짧고 사실적인 핵심 장면 불릿"]
    }
  ],
  "storylines": ["대회 전체 관전 포인트/화제"],
  "topScorers": [{"player": "...", "team": "...", "goals": 3}],
  "_dataNotes": "불확실하거나 미확정인 항목, 출처 상충 사항을 여기에 명시"
}
```

## 필드 규칙
- `status`: `finished` | `live` | `scheduled` 셋 중 하나. 화면 상태 분기의 기준이므로 정확해야 한다.
- `stage`: `Group A`~`Group L`, `Round of 32`, `Round of 16`, `Quarter-final`, `Semi-final`, `Third place`, `Final`.
- 예정 경기: `homeScore`/`awayScore`는 `null`, `highlights`는 `[]`. 진출팀 미정이면 `home`/`away`에 `"조 1위"` 같은 placeholder 허용(코드/국기는 `null`).
- `flag`: 이모지 국기 권장(인터넷 없이 렌더). 없으면 `null`.

## 추가 필드 (v2)
- **시간대:** `time`은 **현지 시각 + 표기(ET 등)** 그대로 저장한다(예: `"18:00 ET"`). 화면은 프론트엔드가 KST로 변환하므로 데이터에서 미리 변환하지 않는다. `date`도 현지 기준.
- **`youtube`(선택):** 경기별 공식 하이라이트 영상 URL을 확인했으면 매치 객체에 `"youtube":"https://..."`로 넣는다. 없으면 생략(프론트가 검색 딥링크로 대체).
- **`knockoutRounds`(배열):** 토너먼트 로드맵용. 라운드별 일정/경기수를 미리 채워 둔다. 대진 확정 전이라도 골격을 제공한다.
```json
"knockoutRounds": [
  {"round":"32강","stage":"Round of 32","dates":"6/28 ~ 7/3","matchCount":16},
  {"round":"16강","stage":"Round of 16","dates":"7/4 ~ 7/7","matchCount":8},
  {"round":"8강","stage":"Quarter-final","dates":"7/9 ~ 7/11","matchCount":4},
  {"round":"4강","stage":"Semi-final","dates":"7/14 ~ 7/15","matchCount":2},
  {"round":"3·4위전","stage":"Third place","dates":"7/18","matchCount":1},
  {"round":"결승","stage":"Final","dates":"7/19 · MetLife Stadium","matchCount":1}
]
```
녹아웃 경기가 확정되면 `matches[]`에 `stage`를 위 영문 단계명으로 맞춰 추가하면 로드맵 진행률과 경기 카드가 자동 갱신된다.

## 갱신 모드 (재실행)
기존 JSON이 있으면 통째로 새로 만들지 말고, `status: scheduled`였다가 종료된 경기의 스코어·하이라이트를 채우고 해당 조 standings를 재계산한다. `lastUpdated`를 갱신한다.

## 산출 위치
`_workspace/01_data-curator_worldcup.json` (오케스트레이터 컨벤션). 최종 화면에는 이 JSON이 `index.html`에 임베드된다.
