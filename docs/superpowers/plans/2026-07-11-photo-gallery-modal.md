# 사진 갤러리 모달 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 가게 카드에서 확대아이콘 클릭 시 대표사진(`photo`) + 추가사진(`extraPhotos`)을 좌우로 넘겨볼 수 있는 라이트박스 모달을 추가한다.

**Architecture:** 단일 HTML 파일(`suntory-tatsujin-guide.html`) 안에 CSS/모달 마크업/JS를 추가하는 방식. 데이터는 각 가게 객체에 옵션 필드 `extraPhotos: string[]`만 추가하면 되고, 코드는 이번 작업으로 완성 후 재수정 불필요. 기존 `#results` 이벤트위임 패턴을 그대로 재사용한다.

**Tech Stack:** Vanilla JS/CSS, 빌드도구 없음, 테스트 프레임워크 없음(정적 HTML 1파일) — 검증은 브라우저에서 직접 확인.

## Global Constraints
- 기존 `photo` 필드/마킹(🚩) 기능 절대 변경 금지 — 그대로 유지.
- `card()`/`magazineCard()`/`render()`의 필터·정렬 로직 무변경.
- 모바일(≤620px) 탭펼침(`initTileToggle`, `#results` 위임)과 이벤트 충돌 금지 — 확대버튼 클릭 시 반드시 `stopPropagation()`.
- 사진 1장짜리 가게는 화살표/점 없이 확대뷰만 뜨는 게 기존 커밋(`fc64562`)과 동일한 동작이어야 함.
- 새 코드는 `#results`에 이벤트위임 1개 리스너로 처리(카드 개수만큼 리스너 만들지 않기).

---

### Task 1: 확대버튼 + 모달 마크업/CSS 추가

**Files:**
- Modify: `suntory-tatsujin-guide.html` (CSS `</style>` 앞, 함수 영역 `card()`/`magazineCard()` 앞, body 상단 마크업)

**Interfaces:**
- Produces: `galleryPhotos(item)` → `string[]` (photo + extraPhotos 합친 배열, falsy 제거)
- Produces: `zoomButton(item)` → HTML string (사진 없으면 빈 문자열)
- Produces: DOM `#photoModal` (모달 컨테이너), `.photo-modal-open` 상태클래스

- [ ] **Step 1: `card()` 함수 앞에 헬퍼 2개 추가**

`function card(item) {` 바로 위에 삽입:

```js
function galleryPhotos(item) {
  return [item.photo, ...(item.extraPhotos || [])].filter(Boolean);
}
function zoomButton(item) {
  const photos = galleryPhotos(item);
  if (!photos.length) return '';
  return `<button class="photo-zoom" type="button" title="사진 크게 보기" data-photos='${JSON.stringify(photos).replace(/'/g, '&#39;')}' data-name="${item.name}">🔍</button>`;
}
```

- [ ] **Step 2: `card()`의 `.thumb` 안에 버튼 삽입**

`suntory-tatsujin-guide.html` 안 `card()` 함수에서:

```js
    <div class="thumb">
      ${photoBlock}
      <button class="photo-flag${flagOn}" type="button" title="사진 별로 — 교체 필요 표시">🚩</button>
```

를 아래로 교체 (flag 버튼 유지, zoom 버튼만 추가):

```js
    <div class="thumb">
      ${photoBlock}
      ${zoomButton(item)}
      <button class="photo-flag${flagOn}" type="button" title="사진 별로 — 교체 필요 표시">🚩</button>
```

- [ ] **Step 3: `magazineCard()`의 `.mag-photo` 안에도 버튼 삽입**

```js
    <div class="mag-photo">
      ${photoBlock}
      <div class="mag-gradient"></div>
```

를 아래로 교체:

```js
    <div class="mag-photo">
      ${photoBlock}
      ${zoomButton(item)}
      <div class="mag-gradient"></div>
```

- [ ] **Step 4: 모달 마크업 추가**

`<div id="flagExportBar">...</div>` 바로 다음(즉 `<div class="wrap">` 앞)에 삽입:

```html
<div id="photoModal" class="photo-modal">
  <div class="photo-modal-inner">
    <button class="photo-modal-close" type="button" title="닫기">✕</button>
    <button class="photo-modal-prev" type="button" title="이전 사진">‹</button>
    <img class="photo-modal-img" alt="">
    <button class="photo-modal-next" type="button" title="다음 사진">›</button>
    <div class="photo-modal-dots"></div>
  </div>
</div>
```

- [ ] **Step 5: CSS 추가**

`#flagExportBar button { ... }` 블록 다음, `</style>` 앞에 삽입:

```css
.photo-zoom {
  position: absolute;
  bottom: 6px;
  right: 6px;
  z-index: 5;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: none;
  background: rgba(0,0,0,.55);
  color: #fff;
  font-size: 13px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
.photo-modal {
  display: none;
  position: fixed;
  inset: 0;
  z-index: 10000;
  background: rgba(0,0,0,.85);
  align-items: center;
  justify-content: center;
}
.photo-modal.open {
  display: flex;
}
.photo-modal-inner {
  position: relative;
  max-width: 92vw;
  max-height: 88vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.photo-modal-img {
  max-width: 92vw;
  max-height: 80vh;
  object-fit: contain;
  border-radius: 8px;
}
.photo-modal-close {
  position: absolute;
  top: -40px;
  right: 0;
  background: none;
  border: none;
  color: #fff;
  font-size: 22px;
  cursor: pointer;
}
.photo-modal-prev, .photo-modal-next {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0,0,0,.5);
  border: none;
  color: #fff;
  font-size: 28px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  cursor: pointer;
  display: none;
}
.photo-modal-prev { left: -8px; }
.photo-modal-next { right: -8px; }
.photo-modal.multi .photo-modal-prev,
.photo-modal.multi .photo-modal-next {
  display: block;
}
.photo-modal-dots {
  display: none;
  gap: 6px;
  margin-top: 12px;
}
.photo-modal.multi .photo-modal-dots {
  display: flex;
}
.photo-modal-dots span {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: rgba(255,255,255,.4);
}
.photo-modal-dots span.active {
  background: #fff;
}
```

- [ ] **Step 6: 브라우저에서 확대아이콘 노출 확인**

`suntory-tatsujin-guide.html`을 브라우저로 열고, 카드 썸네일 우측하단에 🔍 아이콘이 뜨는지 확인. 클릭해도 아직 아무 반응 없는 게 정상(모달 로직은 Task 2에서 추가).

- [ ] **Step 7: 커밋**

```bash
git add suntory-tatsujin-guide.html
git commit -m "feat: 사진 확대 버튼 + 모달 마크업/CSS 추가 (동작 로직은 다음 커밋)"
```

---

### Task 2: 모달 열기/닫기/화살표/점 JS 구현

**Files:**
- Modify: `suntory-tatsujin-guide.html` (`initFlagMode();` 호출부 근처에 `initPhotoModal()` 함수/호출 추가)

**Interfaces:**
- Consumes: `#photoModal`, `.photo-modal-img`, `.photo-modal-dots`, `.photo-modal-prev/next/close` (Task 1에서 생성)
- Consumes: `.photo-zoom` 버튼의 `data-photos`(JSON 문자열), `data-name` 속성
- Produces: `initPhotoModal()` — 전역에서 한 번 호출하는 초기화 함수

- [ ] **Step 1: `initFlagMode()` 함수 뒤에 `initPhotoModal()` 추가**

```js
// 사진 갤러리 모달 — 확대아이콘 클릭 시 photo+extraPhotos 넘겨보기
function initPhotoModal() {
  const modal = document.getElementById('photoModal');
  const img = modal.querySelector('.photo-modal-img');
  const dotsWrap = modal.querySelector('.photo-modal-dots');
  let photos = [];
  let index = 0;

  function renderModal() {
    img.src = photos[index];
    img.alt = '';
    modal.classList.toggle('multi', photos.length > 1);
    dotsWrap.innerHTML = photos.map((_, i) =>
      `<span class="${i === index ? 'active' : ''}"></span>`
    ).join('');
  }

  function open(list, name) {
    if (!list.length) return;
    photos = list;
    index = 0;
    img.alt = name || '';
    renderModal();
    modal.classList.add('open');
    document.body.classList.add('splash-active');
  }

  function close() {
    modal.classList.remove('open');
    document.body.classList.remove('splash-active');
  }

  function prev() {
    index = (index - 1 + photos.length) % photos.length;
    renderModal();
  }

  function next() {
    index = (index + 1) % photos.length;
    renderModal();
  }

  document.getElementById('results').addEventListener('click', e => {
    const zoomBtn = e.target.closest('.photo-zoom');
    if (!zoomBtn) return;
    e.preventDefault();
    e.stopPropagation();
    const list = JSON.parse(zoomBtn.dataset.photos.replace(/&#39;/g, "'"));
    open(list, zoomBtn.dataset.name);
  });

  modal.querySelector('.photo-modal-close').addEventListener('click', close);
  modal.querySelector('.photo-modal-prev').addEventListener('click', prev);
  modal.querySelector('.photo-modal-next').addEventListener('click', next);
  modal.addEventListener('click', e => { if (e.target === modal) close(); });
  document.addEventListener('keydown', e => {
    if (!modal.classList.contains('open')) return;
    if (e.key === 'Escape') close();
    if (e.key === 'ArrowLeft') prev();
    if (e.key === 'ArrowRight') next();
  });
}
```

주의: `close()`/`open()`에서 `splash-active` 클래스를 재사용해 스크롤 잠금 처리함 — 스플래시가 이미 끝난 뒤(splash 엘리먼트는 DOM에서 제거된 상태)라 클래스 재사용은 순수하게 `overflow:hidden` 용도로만 동작함 (`suntory-tatsujin-guide.html`의 `body.splash-active { overflow: hidden; }` 규칙, Task 1 이전부터 존재).

- [ ] **Step 2: 호출부에 `initPhotoModal();` 추가**

```js
initTileToggle();
initSplash();
initFlagMode();
initPhotoModal();
```

- [ ] **Step 3: 브라우저에서 단일사진 동작 확인**

브라우저로 열고 아무 카드나 🔍 클릭 → 모달이 뜨고 사진 1장만 크게 보이는지, 화살표/점이 안 보이는지 확인. X 버튼 / 오버레이 클릭(사진 바깥 어두운 영역) / ESC 전부 닫히는지 확인.

- [ ] **Step 4: 브라우저 콘솔에서 다중사진 동작 확인 (DATA 수정 없이)**

브라우저 개발자도구 콘솔에서 실행:

```js
DATA[0].extraPhotos = [DATA[0].photo, DATA[0].photo.replace('640x640', '320x320')];
render();
```

DATA[0] 카드의 🔍 클릭 → 화살표(‹ ›)와 점 인디케이터가 보이는지, 클릭/좌우 화살표키로 사진이 바뀌는지, 마지막 사진에서 다음 누르면 첫 사진으로 순환하는지 확인. 확인 후 새로고침하면 원상복구(런타임 수정이라 파일엔 저장 안 됨).

- [ ] **Step 5: 커밋**

```bash
git add suntory-tatsujin-guide.html
git commit -m "feat: 사진 모달 열기/닫기/화살표/점 인터랙션 구현"
```

---

### Task 3: 모바일 스와이프 + 탭펼침 충돌 방지 확인

**Files:**
- Modify: `suntory-tatsujin-guide.html` (`initPhotoModal()` 함수 내부에 스와이프 리스너 추가)

**Interfaces:**
- Consumes: Task 2의 `prev()`, `next()`, `modal` 엘리먼트
- Consumes: `initTileToggle()`(기존 함수, `suntory-tatsujin-guide.html`에 이미 존재) — 모바일 탭펼침 로직, 무변경

- [ ] **Step 1: `initPhotoModal()` 안, `document.addEventListener('keydown', ...)` 블록 다음에 스와이프 추가**

```js
  let touchStartX = null;
  modal.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
  }, { passive: true });
  modal.addEventListener('touchend', e => {
    if (touchStartX === null) return;
    const dx = e.changedTouches[0].clientX - touchStartX;
    if (Math.abs(dx) > 40 && photos.length > 1) {
      dx > 0 ? prev() : next();
    }
    touchStartX = null;
  }, { passive: true });
```

- [ ] **Step 2: 모바일 폭에서 탭펼침과 충돌 없는지 확인**

브라우저 개발자도구로 뷰포트를 620px 이하(예: 390px)로 줄인 뒤:
1. Task 2 Step 4의 콘솔 명령으로 `DATA[0].extraPhotos` 세팅 + `render()` 재실행.
2. 카드를 탭 → 카드가 펼쳐지는지 확인(기존 동작, 안 깨졌는지).
3. 같은 카드의 🔍 아이콘을 탭 → 카드 펼침은 발동하지 않고 모달만 뜨는지 확인 (Task 1의 `e.stopPropagation()` 덕분).
4. 모달 안에서 사진을 좌우로 스와이프 → 사진이 넘어가는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add suntory-tatsujin-guide.html
git commit -m "feat: 사진 모달 모바일 스와이프 지원"
```

---

### Task 4: 데이터 추가 워크플로 확정 (문서화만, 코드 변경 없음)

**Files:**
- Modify: 없음 (검증 + 확인용 태스크)

- [ ] **Step 1: 실제 가게 하나에 `extraPhotos` 2장 넣고 최종 확인**

`suntory-tatsujin-guide.html`의 `DATA` 배열에서 아무 가게 객체 하나(예: 도쿄 첫 번째 항목)에 실제 이미지 URL 2개로 `extraPhotos: ['url1','url2']`를 추가 → 저장 → 새로고침 → 🔍 클릭해서 화살표/점/스와이프 전부 정상 동작하는지 최종 확인.

- [ ] **Step 2: 확인 끝나면 테스트용 `extraPhotos` 되돌리기 (진짜 사진 준비 전이면)**

```bash
git diff suntory-tatsujin-guide.html
git checkout -- suntory-tatsujin-guide.html
```

(진짜 추가사진 URL이 이미 있어서 그대로 커밋하고 싶다면 이 스텝은 건너뛰고 바로 커밋)

- [ ] **Step 3: 커밋 (실사진을 넣은 경우만)**

```bash
git add suntory-tatsujin-guide.html
git commit -m "content: <가게명> extraPhotos 추가"
```
