# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 프로젝트 개요

**TONOTE** — 향수 입문자를 위한 향수 추천 SPA (UCD 수업 8조 과제)  
GitHub Pages 배포: `https://bbodok1928-byte.github.io/TONOTE/`  
Git 레포: `git@github.com:bbodok1928-byte/TONOTE.git`

---

## 파일 구조

```
/Users/baeyeonju/Desktop/8조/
├── TONOTE_final.html   ← 유일한 소스 파일. 모든 수정은 여기에만 한다.
├── index.html          ← GitHub Pages 엔트리. TONOTE_final.html 의 복사본. 자동 동기화.
├── TONOTE_feature_spec.md  ← 기능 명세서 (F1~F6)
├── Persona1.md / Persona2.md  ← 유저 페르소나 문서
└── (구버전 프로토타입들 — 무시해도 됨)
```

> **단일 파일 SPA**: CSS·JS·HTML 전부 `TONOTE_final.html` 한 파일 안에 있다.

---

## 배포 워크플로

```bash
# 1. 소스를 index.html에 동기화
cp /Users/baeyeonju/Desktop/8조/TONOTE_final.html /Users/baeyeonju/Desktop/8조/index.html

# 2. 커밋 & 푸시
git add TONOTE_final.html index.html
git commit -m "..."
git push origin main

# 3. 태그 force-update (항상 'final' 태그를 최신으로 유지)
git tag -f final
git push origin final --force
```

GitHub Pages 반영까지 약 30~60초 소요.

---

## 앱 레이아웃 구조 (CSS 핵심)

```
.app { height:100svh; max-width:420px; overflow:hidden }
  #app-shell { position:absolute; inset:0; display:flex; flex-direction:column }
    .screens { flex:1; position:relative; overflow:hidden }
      .screen { position:absolute; inset:0; overflow-y:auto; padding-bottom:var(--nav) }
        /* .screen 의 padding-bottom:60px 는 bnav 공간 확보용 */
        /* #screen-ai 는 예외: style="overflow:hidden;padding-bottom:0" 인라인 적용 */
    .bnav { height:60px(var(--nav)); flex-shrink:0 }
```

**⚠️ 레이아웃 주의사항**

- `.screens`는 `flex:1`이므로 `.bnav`(60px) 높이가 이미 제외된 상태다.  
- 따라서 `.screen` 안에서 `padding-bottom:60px`를 주면 그 안에서 또 60px가 생긴다. 일반 스크롤 화면은 이게 의도적이지만(스크롤 끝에 bnav가 덮지 않도록), AI 채팅 화면처럼 고정 레이아웃에서는 `padding-bottom:0` 오버라이드가 필요하다.
- `#screen-ai`에 `style="overflow:hidden;padding-bottom:0"` 인라인 스타일이 적용되어 있다.

---

## 핵심 JS 데이터 구조

### PERFUMES 배열 (현재 85개, IDs 1~85)

```javascript
{
  id: 1,
  brand: 'LE LABO',
  name: 'Santal 33',
  emoji: '🌿',
  img: 'https://images.unsplash.com/...?w=400',  // Unsplash 이미지 URL
  bg: 'linear-gradient(135deg,#E8F0E6,#D4E4D0)', // 카드 배경 그라디언트
  poem: '비 온 뒤 숲 속 오솔길, 젖은 나무의 조용한 숨결',  // 감성 한 줄
  story: '긴 설명 텍스트...',  // 상세 스토리 (줄바꿈 \n 사용)
  intensity: 4,         // 향 강도 1~5
  moods: ['고요한 숲', '비 오는 날', '늦은 밤'],  // 감성 태그 (3개 이상)
  tags: ['고요한', '우디', '가을·겨울'],          // 부가 태그
  cat: '우디',           // 카테고리: '플로럴'|'우디'|'프루티'|'시트러스'|'머스크'|'아쿠아틱'|'스파이시'|'구르망'
  season: ['가을', '겨울'],
  longevity: 5,          // 지속력 1~5
  sillage: 3,            // 확산력 1~5
  price: 295000,         // 정가 (원)
  samplePrice: 4500,     // 샘플 가격 (원)
  sampleAvail: true,
  rating: 4.8,
  reviews: 1243,
  top: ['카다멈', '바이올렛'],
  mid: ['아이리스', '아미리스'],
  base: ['샌달우드', '레더', '머스크'],
  desc: '짧은 영문 설명...',
  similar: [2, 6, 8]     // 유사 향수 id (다른 PERFUMES 항목의 id)
}
```

**카테고리 목록** (탐색 탭 칩과 일치해야 함):  
`전체` / `플로럴` / `우디` / `프루티` / `시트러스` / `머스크` / `아쿠아틱` / `스파이시` / `구르망`

### STORES 배열 (현재 ~260개, IDs 0~260)

```javascript
{
  id: 0,
  name: '올리브영 홍대점',
  brand: 'OLIVE YOUNG',  // 브랜드 필터 기준
  addr: '서울 마포구 양화로 188',
  city: '서울',
  emoji: '🌿',
  bg: '#EAF2E8',
  pressure: 1,           // 직원 개입도: 1(낮음)|2(보통)|3(높음)
  status: 'open',        // 'open'|'closed'
  lat: 37.5572,
  lng: 126.9219
}
```

**브랜드 필터 카테고리**:  
`OLIVE YOUNG` / `CHICOR` / `백화점향수관` (LOTTE·SINSEGAE·GALLERIA·DIPTYQUE·HYUNDAI 포함) / `향수편집샵`

---

## 핵심 JS 함수

| 함수 | 역할 |
|------|------|
| `matchPerfumes(input)` | KW 딕셔너리 기반 키워드 점수 계산 → 향수 추천 |
| `renderExplore()` | `activeCat` + 검색어로 PERFUMES 필터링 후 탐색 탭 렌더 |
| `locateUser()` | Geolocation API → 실패 시 **부산 센텀시티역(35.1689, 129.1321)** 무음 폴백 (alert/toast 없음) |
| `updateStoreListByProximity(lat,lng)` | STORES를 거리순 정렬 후 renderSpaces 호출 |
| `renderSpaces(sortedList, distMap)` | 매장 카드 렌더 — 브랜드 로고 이미지 우선, 없으면 emoji 폴백 |
| `_getStoreLogo(s)` | 매장의 brand/name으로 Clearbit 로고 URL 반환 (없으면 null) |
| `_haversineDist(lat1,lng1,lat2,lng2)` | 하버사인 공식으로 두 좌표 간 거리(m) 반환 |
| `_formatDist(m)` | 거리 포맷 — 1000m 미만 "230m", 이상 "1.2km" |
| `pickPoetic(topMood,topTag,rawInput)` | 원시 입력 정규식으로 컨텍스트 우선 분기 후 POETIC 반환 |
| `_syncAccountBanner()` | 설정 모달 오픈 시 실제 유저 프로필 데이터 동기화 |
| `openNotice()` / `closeFaq()` / `toggleFaq(idx)` | 공지사항·FAQ 바텀시트 모달 제어 |
| `openDictionary()` / `closeDictionary()` | 향기 용어 사전 바텀시트 모달 제어 |
| `toggleNotice(idx)` | 공지사항 아코디언 토글 |

---

## AI 매칭 딕셔너리

```javascript
// KW: 유저 입력 키워드 → mood/tag 값으로 매핑
const KW = {
  '비': '비 오는 날', '숲': '고요한 숲', '달콤': '달콤한',
  '침대': '늦은 밤',  // ← 수면 컨텍스트: 카페오후 아님!
  '이불': '늦은 밤', '잠': '늦은 밤', '수면': '늦은 밤',
  '서점': '카페 오후', '책방': '카페 오후',  // ← 카페 컨텍스트 명시
  ...
};

// POETIC: mood/tag 값 → 감성 응답 문자열 배열 (랜덤 pick)
const POETIC = { '비 오는 날': [...], '카페 오후': [...], ... };
```

`pickPoetic(topMood, topTag, rawInput)` — **rawInput 기반 컨텍스트 우선 분기**:
1. `/침대|이불|잠|수면|베개|안락|자기 전/` 매칭 → 수면 전용 카피 고정 출력
2. `/카페|커피|서점|책방|오후|독서/` 매칭 → 카페오후 카피 고정 출력
3. 위 패턴 없으면 topMood/topTag → POETIC 정상 랜덤 출력

---

## 화면(Screen) 목록

| ID | 탭 | 설명 |
|----|-----|------|
| `#screen-home` | 홈 | `.ai-card` 배너(margin-top:20px) + 무드칩 + 향수 카드 |
| `#screen-explore` | 탐색 | 카테고리 칩 필터 + 향수 그리드 (85개) |
| `#screen-spaces` | 공간 | Leaflet 지도 + 매장 목록 (overlay-stores 모달) |
| `#screen-ai` | AI | 채팅 인터페이스 (`overflow:hidden; padding-bottom:0`) |
| `#screen-my` | 마이 | 프로필 + 메뉴 + 고객지원(공지/FAQ/향기용어사전) + 푸터 |

**홈 화면 구조**: 히어로 h1 제거됨. `.ai-card` 배너가 상단 첫 컴포넌트.  
배너 카피 고정: `"당신의 문장이 향기로 흩어지는 시간"` (로테이션 없음)

---

## 자주 발생하는 CSS 버그 패턴

1. **`overflow-x:auto` 설정 시 `overflow-y`가 자동으로 hidden이 됨 (CSS 스펙)**  
   → 가로 스크롤만 원하면 반드시 `overflow-y:visible`도 명시해야 함  
   → `.cat-chips`에 이미 적용됨

2. **`.screens`(flex:1) + `.bnav`(fixed height) 구조에서 padding-bottom 이중 계산**  
   → `.screen`의 기본 `padding-bottom:var(--nav)`는 스크롤 가능 화면에서 필요하지만, 고정 레이아웃 화면(`#screen-ai`)에서는 이중 공간을 만듦  
   → `#screen-ai`에 인라인 `style="overflow:hidden;padding-bottom:0"` 유지 필수

3. **`min-height` 없이 flex 칩 컨테이너에 `overflow-y:visible` 적용 시 높이가 0이 될 수 있음**  
   → `.cat-chips`에 `min-height:52px` 적용됨

---

## 2026-06-05 세션 구현 완료 내역

### 시향공간 (Spaces)
- **브랜드 로고**: `_getStoreLogo(s)` → Clearbit Logo API, `onerror` 시 emoji 폴백
- **위치 오류**: `locateUser()` 실패 시 `alert/showToast` 완전 제거, 부산 센텀시티역(35.1689, 129.1321) 무음 폴백
- **배지 정렬**: 거리 배지 + 영업상태 배지 같은 flex row에 `align-items:center`
- **배너 문구**: "직원 개입도 낮은 매장만 엄선..." 삭제

### 마이 탭
- **프로필 프리셋**: 5개 이모지 칩(🌿🌹🤍🗿🧴), localStorage 저장, 커뮤니티 아바타 동기화
- **고객지원 칼정렬**: `my-support-item` = `my-menu-item`과 동일 padding/font/gap/icon 크기
- **하단 여백**: `.screen { padding-bottom:20px }`, `.sec { padding-bottom:16px }` 슬림화
- **고객지원 섹션 헤더**: `--s2` 배경 풀-위드 블록 스타일
- **배너 문구 삭제**: "직원 개입도 낮은..." 제거, banner-top padding 보정
- **공지사항 모달** (`#overlay-notice`): 바텀시트 + 아코디언 토글 + Maru Buri 폰트
- **FAQ 모달** (`#overlay-faq`): 아코디언 Q&A 2건
- **향기 용어 사전** (`#overlay-perfume-dictionary`): 9종 (시트러스/우디/플로럴/머스크/그린/오리엔탈/파우더리/스파이시/아쿠아)

### 설정 모달
- **계정 카드**: "김지은" 하드코딩 → `_syncAccountBanner()`로 실시간 동기화 (nickname, avatar, sub)
- **"로컬 체험판" 뱃지**: HTML + CSS 완전 삭제
- **취향 재진단 버튼**: `closeSettings()` → `setTimeout(openScentTest, 220)`
- **선택 초기화 리스트**: bordered 버튼 → `.data-reset-item` 미니멀 라인, 소프트 로즈브라운(`#B85C54`)
- **야간 푸시 토글**: 알림 설정 탭 하단 🌙 추가
- **탭 오타 수정**: `switchSettingsTab('preference')` → `'notification'`

### 홈 화면
- **히어로 h1 삭제**: "오늘의 감정이 향이 됩니다." 완전 제거
- **배너 상단 여백**: `.ai-card { margin-top:20px }` 추가
- **배너 카피 고정**: "당신의 문장이 향기로 흩어지는 시간" (로테이션 제거)

### AI 탭
- **맥락 분기**: `pickPoetic(topMood, topTag, rawInput)` — 침대/수면 → 전용 카피, 카페/서점 → 커피 카피
- **KW 재매핑**: `'침대'→'늦은 밤'`, `'서점'→'카페 오후'`

### 스플래시
- **로고 컬러**: `#FFFFFF` → `#4A3736` (다크 로즈브라운), font-weight 400→600

### 타이포그래피 시스템
- **Google Fonts**: `Maru Buri` (400/600) `<link>` 주입
- **제목/감성 텍스트**: `'Maru Buri','ChosunIlbo_Myungjo',serif`
- **본문/UI**: `'Pretendard','Noto Sans KR',-apple-system,sans-serif`
- 적용 위치: AI 말풍선(Maru Buri), AI 칩/입력창(Pretendard), 공지사항 타이틀(Maru Buri), 공지 본문(Pretendard), 향기용어사전 제목(Maru Buri), 설명(Pretendard), 홈 배너 카피(Maru Buri), 섹션 타이틀(Pretendard 700)

---

## 기능 명세 요약 (TONOTE_feature_spec.md)

| 기능 | 우선순위 | 상태 |
|------|---------|------|
| F1 향수 취향 진단 테스트 | P0 | MVP 핵심 |
| F2 감성 태그 기반 향수 카드 | P0 | 구현됨 (85개 향수) |
| F3 향수 비교 모드 | P0 | 미구현 |
| F4 셀프 시향 키트 신청 | P1 | 미구현 |
| F5 오프라인 시향 공간 가이드 | P1 | 구현됨 (260개 매장) |
| F6 취향 기반 향수 피드 | P2 | MVP 외 보류 |
