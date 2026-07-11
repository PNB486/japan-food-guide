# suntory-tatsujin-guide.html — 2a/2b 레이아웃 적용 지시서

> Claude Design 시안 **2a(데스크톱 매거진 카드)** + **2b(모바일 타일→상세)** 를 원본에 이식하는 작업.
> 폰트(Zen Kaku Gothic New + Noto Sans KR)와 도시 언더라인 네비(`.city-nav`)는 **이미 적용된 상태**이므로 유지.

---

## 0. 절대 건드리지 말 것 (Hard Constraints)

다음은 **원문 그대로 보존**한다. diff에 이 라인들이 뜨면 실패로 간주:

- `const DATA = [...]` — 매장 350곳 배열 (전체)
- **필터/정렬/검색 로직**: `render()` 내부의 `DATA.filter(...)` 조건부 전체, `items.sort(...)` 분기, `buildChips()` 전체
- **일본어→한국어 번역**: `translateAccess`, `formatBudget`, `translateClosed`, `translateTier`, `deriveBucket`, `tierBucket` 및 `STATION_MAP` / `ACCESS_MAP` / `FALLBACK_MAP` 상수
- `michelinBadge`, `practicalRow`, `scoreClass`, `parseScoreValue`, `parseReviewValue`, `mapUrl`, `tabelogSearchUrl`, `hasMastersDream` 등 헬퍼
- 이미 적용된 폰트 `<link>`, body/제목 `font-family`, `.city-nav` CSS 블록

> **원칙**: 데이터·필터·번역은 손대지 않는다. 변경은 **표시(presentation) 레이어** — CSS + `render()`의 카드 배치 분기 + 신규 렌더 함수 + 모바일 토글 JS — 로 한정한다.

---

## 1. 공통 디자인 토큰 (시안 2a/2b 기준)

기존 CSS 변수(`--bg`, `--surface`, `--accent` 등)는 그대로 쓰되, 시안 실측값과 대응 관계는 아래와 같다. **색상은 기존 골드 유지**가 원칙이므로 기존 변수를 우선 사용하고, 시안 HEX는 참고용:

| 용도 | 시안 실측 | 매핑 |
|---|---|---|
| 카드 배경 surface | `#1e1a15` | `var(--surface-2)` 계열 유지 |
| 포인트 골드 | `#c9a86a` | `var(--accent)` 유지 |
| 본문 텍스트 | `#c7bda9` | `var(--text-dim)` 유지 |
| 라벨 dim | `#6f6657` | `var(--text-faint)` 유지 |
| 헤어라인 | `rgba(201,168,106,.12)` | `var(--line)` 유지 |
| 제목 폰트 | — | `"Zen Kaku Gothic New"` (적용됨) |

---

## 2. 데스크톱 — 도시별 첫 매장 = 대형 매거진 카드 (시안 2a)

### 2-1. 목표
각 도시 섹션에서 **첫 번째 매장만** 좌우 2단 대형 카드로 렌더. **나머지 매장은 기존 3열 그리드 그대로**.

### 2-2. 구현 방식 (render 분기)
`render()` 안에서 도시별 카드 목록을 만드는 부분:

```js
// 현재
<div class="grid">${items.map(card).join('')}</div>
```
를 아래처럼 분기:
```js
// 변경 후 (items 정렬은 기존 로직 그대로 통과한 결과)
const [lead, ...rest] = items;
const leadHtml = lead ? magazineCard(lead) : '';
const restHtml = rest.map(card).join('');
// ...
<div class="city-lead">${leadHtml}</div>
<div class="grid">${restHtml}</div>
```

> `card()` 함수와 `items` 정렬/필터는 **수정 금지**. 첫 요소만 뽑아 다른 렌더 함수에 넘기는 것뿐.

### 2-3. `magazineCard(item)` 신규 함수 (card 옆에 추가)

기존 `card()`가 참조하는 헬퍼(`photoBlock` 생성 로직, `translateAccess`, `formatBudget`, `translateClosed`, `michelinBadge`, `translateTier`, `practicalRow` 등)를 **동일하게 재사용**한다. 마크업 구조만 매거진형으로:

**레이아웃 스펙 (시안 2a 실측)**
- 컨테이너: `display:grid; grid-template-columns:1.55fr 1fr; border-radius:12px; overflow:hidden; margin-bottom:26px; border:1px solid var(--line); background:var(--surface-2);`
- **왼쪽 사진 영역**: `position:relative; min-height:400px`
  - `img`: `position:absolute; inset:0; width:100%; height:100%; object-fit:cover`
  - 하단 그라데이션: `position:absolute; inset:auto 0 0 0; height:120px; background:linear-gradient(transparent, rgba(14,12,9,.6))`
  - 좌상단 뱃지(장르/대표): `left:20px; top:18px; font-size:10px; letter-spacing:.22em; background:rgba(14,12,9,.55); padding:6px 12px; border-radius:99px; backdrop-filter:blur(4px)`
  - 좌하단 평점: `left:20px; bottom:16px` — 점수 `Zen Kaku Gothic New; font-size:34px; color:var(--accent)` + 리뷰/저장 `font-size:11px; color:rgba(255,255,255,.75)`
- **오른쪽 정보 영역**: `padding:32px 34px; display:flex; flex-direction:column`
  - 매장명: `Zen Kaku Gothic New; font-size:26px; font-weight:600; line-height:1.35` (일본어명, `item.name`)
  - 한글명: `font-size:13px; color:var(--text-dim); margin-top:6px` (`item.nameKr`)
  - 등급 배지(있을 때만): 골드 점 + `font-size:12px; color:var(--accent)` (`translateTier(item.tier)`)
  - 헤어라인: `height:1px; background:var(--line); margin:20px 0 16px`
  - 정보 그리드: `grid-template-columns:40px 1fr; row-gap:8px; font-size:12.5px` — 주소/교통/예산 (교통은 `translateAccess`, 예산은 `formatBudget` 통과값)
  - 대표 메뉴: 라벨 `대표 메뉴`(`font-size:9.5px; letter-spacing:.2em; color:var(--text-faint)`) + 3개를 **인라인 1줄**형 `<메뉴 일본어> <span>— 한글</span>` (일본어 `font-size:13px; color:var(--text)`, 한글 `font-size:11px; color:#8a8071`)
  - 하단 pill 줄: `margin-top:auto; padding-top:18px` — `practicalRow(item)` 재사용하되, 매거진에선 텍스트 점(·) 구분형으로 표시(선택). pill 그대로 써도 무방.
  - 미슐랭 있으면 매장명 옆에 `michelinBadge(item.michelin)` 붙임.

### 2-4. CSS 클래스로 분리 (인라인 style 금지)
시안은 인라인 style이지만, 원본 스타일 규칙에 맞춰 `.mag-card`, `.mag-photo`, `.mag-body` 등 **클래스로 작성**하고 `<style>`에 추가한다. 기존 `.card`/`.grid` 규칙은 수정하지 않는다.

### 2-5. 반응형
`@media (max-width:820px)`에서 `.mag-card { grid-template-columns:1fr }` 로 사진 위·정보 아래 1열 전환. 사진 `min-height:240px`로 축소.

---

## 3. 모바일 — 사진 타일 그리드 → 탭 → 상세 (시안 2b, ≤620px)

### 3-1. 목표
모바일 폭에서는 카드 대신 **2열 사진 타일**(사진+장르+평점+매장명)로 요약 표시. 타일 탭 시 해당 매장 **상세가 펼쳐짐**(같은 데이터). 뒤로가기(‹ 목록으로)로 복귀.

### 3-2. 구현 원칙 (표시 레이어 토글, 데이터 무관)
- 렌더는 기존 `render()` 결과(=데스크톱 카드/매거진 DOM)를 그대로 두고, **CSS로 모바일에서 요약 타일처럼 축약**하는 방식을 우선 검토.
  - 단, 시안 2b는 "타일에는 사진+평점만, 상세는 탭 후"라 정보 표시량이 크게 달라 CSS-only로는 한계. → **경량 JS 토글**을 추가한다.
- **권장 구현**: 각 `.card`(및 매거진 카드)에 이미 들어있는 정보를 사용하되, 모바일에서는
  1. 카드를 `.card--collapsed` 상태로: 사진 썸네일 + 매장명 + 평점만 보이고 나머지(`.meta`, `.menu`, `.badges`, `.practical-row`)는 `display:none`.
  2. 카드 탭(click) 시 `.card--expanded` 토글 → 숨겼던 영역 표시. 한 번에 하나만 펼치려면 형제 카드의 expanded 제거.
- JS는 **이벤트 위임** 하나로 처리 (신규 리스너 1개, 기존 로직과 독립):
```js
// render() 이후 한 번만 바인딩. 데이터/필터와 무관.
document.getElementById('results').addEventListener('click', (e) => {
  if (window.innerWidth > 620) return;              // 모바일에서만
  const card = e.target.closest('.card, .mag-card');
  if (!card) return;
  if (e.target.closest('a')) return;                // 링크 클릭은 통과
  card.classList.toggle('is-open');
});
```
> `render()`가 다시 호출되면 DOM이 새로 그려지므로, 리스너는 `results` 컨테이너(부모)에 **위임**으로 걸어 한 번만 등록되게 한다. (필터 변경 시 재바인딩 불필요)

### 3-3. 모바일 타일 CSS 스펙 (시안 2b 실측)
`@media (max-width:620px)` 안에서:
- 그리드: `.grid { grid-template-columns:1fr 1fr; gap:10px }` (기존 1열 → 2열 타일)
- 타일(`.card`): `border-radius:10px`
  - 사진(`.thumb`): `height:118px`
  - 장르 뱃지: 좌상단 `left:9px; top:8px; font-size:8.5px; letter-spacing:.14em; background:rgba(14,12,9,.5); padding:3px 7px; border-radius:99px`
  - 평점: 사진 우하단 `right:9px; bottom:8px; Zen Kaku Gothic New; font-size:14px; color:var(--accent)` (평점은 `.badges` 안 다베로그 값 재활용 or 별도 노출)
  - 본문 요약: `padding:10px 11px 12px` — 매장명 `font-size:13px` 1줄 ellipsis + 한글명 `font-size:10px; color:#8a8071` 1줄 ellipsis
  - `.is-open` 아닐 때: `.meta, .menu, .badges, .practical-row { display:none }`
- 상세 펼침(`.card.is-open`):
  - `grid-column:1 / -1` (2열 폭 전체 차지)
  - 숨겼던 `.meta/.menu/.badges/.practical-row` 다시 `display` 복원
  - 사진 `height:200px`로 확대
  - 부드러운 전환 위해 `.card { transition:... }` (단 `prefers-reduced-motion`에선 none — 기존 규칙 존중)

> 매거진 카드(`.mag-card`)도 모바일에선 일반 타일과 동일 취급(2열 타일)하거나, 도시 첫 매장은 `grid-column:1/-1` 큰 타일로 둬도 됨. 택1하여 명시.

---

## 4. 작업 순서 & 검증 (단계별)

각 단계 후 브라우저(Live Server)에서 확인하고 다음으로 진행:

**단계 1 — 데스크톱 매거진 카드**
- 검증 A: 각 도시 섹션 첫 매장이 좌우 2단 대형 카드, 나머지는 3열 그리드
- 검증 B: 매거진 카드의 주소/교통/예산/메뉴가 **번역 함수 통과값**으로 정상 출력 (일본어 원문 아님)
- 검증 C: 미슐랭/등급 배지가 해당 매장에만 표시

**단계 2 — 모바일 타일→상세**
- 검증 D: 390px 폭에서 2열 사진 타일, 사진+매장명+평점만 보임
- 검증 E: 타일 탭 → 상세 펼침(주소·메뉴·pill 표시), 다시 탭/뒤로 시 접힘 — **실제 클릭 테스트**
- 검증 F: 링크(주소·홈페이지) 탭은 펼침 토글과 충돌 없이 이동

**단계 3 — 회귀 검증 (전 기능)**
- 검증 G: 도시/장르/등급/미슐랭/한국인픽/점수/리뷰 필터 + 검색 + 정렬 전부 기존대로 작동
- 검증 H: 다크/라이트 토글 정상, 매너 모달 정상
- 검증 I: `git diff`에 §0 보존 대상(DATA·번역 함수·필터 조건) 변경 라인 **0**

**단계 4 — 정리**
- 내가 이번에 만든 미사용 CSS/함수만 정리. 인접 코드 임의 리팩터 금지.

---

## 5. 참고 파일
- 시안 원본 마크업: `Japan Guide Redesign.dc.html`
  - 데스크톱 매거진 카드: 468~499행
  - 데스크톱 나머지 그리드 카드: 504~ 행
  - 모바일 타일: 650~665행 / 모바일 상세: 669~703행
- 이 파일들의 인라인 style 수치를 그대로 클래스로 옮기면 시안과 픽셀 단위로 일치.

---

## 6. 커밋 메시지 예시
```
feat(layout): 도시별 첫 매장 매거진 카드 + 모바일 타일→상세 (시안 2a/2b)

- render()에 도시 리더 분기 추가, magazineCard() 신규
- 모바일 2열 타일 + 탭 펼침 토글(이벤트 위임 1개)
- DATA/필터/번역 함수 무변경 (표시 레이어 한정)
```
