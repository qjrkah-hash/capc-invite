# 인수인계서 — 고등부 전도축제 탑승권 초대장 → 천장지트 통합

- 작성일: 2026-10-25
- 대상: 천장지트(React + Firebase) 앱에 초대장 기능 이식
- 원본: `qjrkah-hash/capc-invite` 저장소의 `index.html` (단일 파일, 외부 라이브러리 없음)
- 현재 배포: https://qjrkah-hash.github.io/capc-invite/ (GitHub Pages, main 브랜치 루트)

---

## 1. 기능 요약

| 단계 | 내용 |
|---|---|
| ① 입력 | To.(초대받는 친구) / From.(초대하는 나), 각 최대 8자. 두 칸이 다 차야 버튼 활성 |
| ② 생성 | Canvas(1200×600)에 탑승권 그림 + 이름 → PNG Blob 생성 → 결과 화면에 -4° 기울여 표시 |
| ③ 저장 | 기기·브라우저별 분기 (4장 참고) |
| ④ 사전접수 | 결과 화면 + 탑승권 이미지 안에 QR(와우축제 사전접수) + 짧은 설명 |
| ⑤ 다시하기 | 입력 초기화 후 입력 화면으로 |
| 공통 | 인앱브라우저 안내 박스 + "외부 브라우저로 열기" 버튼 |

---

## 2. 행사 데이터 (CONFIG.event / CONFIG.signup)

출처: 고등부 주보(천.장.고 NEWS) 2번 공지, 인스타 @capc_hs 프로필, 와우축제 사전접수 QR.

| 키 | 값 |
|---|---|
| 행사명 | LIVING HOPE 전도축제 (주제 말씀 벧전 1:3) |
| DAY 1 | 10.31 (토) 오후 2–5시 · 본당 · 비전센터 · 주차장 |
| DAY 2 | 11.01 (주일) 오전 11시 · 비전센터 3F 글로리홀 (고등부 예배 + 운동교제) |
| 루트 표기 | EVERYWHERE(어디서든 출발) → 천장고등부(천안장로교회 고등부) |
| 하단 띠 | LIVING HOPE 전도축제 · 찬양 브라이언 킴, 체험부스, 푸드코트 |
| 편명 | LH 1101 (임의) |
| 사전접수 링크 | `https://m.site.naver.com/2iYtX` (QR에서 추출, 네이버 단축 URL) |
| 사전접수 기한 | 10.30(금) 오후 6시까지, 이후 당일 현장 접수 |
| 푸터 | 천안장로교회 고등부 / @capc_hs |

> 행사 문구는 전부 CONFIG 한 곳에 모여 있음. React 이식 시 `inviteConfig.js` 같은 상수 파일로 분리 권장.

---

## 3. 탑승권 이미지 사양 (Canvas 1200×600)

### 색상·폰트
| 토큰 | 값 | 용도 |
|---|---|---|
| navy | `#10223f` | 본문·헤더 밴드 |
| orange | `#ff5a1f` | 강조, From 이름, 스텁 헤더 |
| sky | `#e8eef8` | 하단 띠, 페이지 배경 |
| grey | `#6b7890` | 라벨 |
| muted | `#b3bccb` | 절취선, 비활성 |
| 폰트 | Black Han Sans(제목·이름), Noto Sans KR 500/700/900(본문), IBM Plex Mono 500/700(코드·라벨) | Google Fonts |

### 좌표표 (x, y = 텍스트 baseline)
| 요소 | 위치 / 크기 |
|---|---|
| 카드 | 0,0 1200×600, radius 28, 흰 바탕 |
| 헤더 밴드 | y 0–104 navy / 스텁(x≥860) orange |
| BOARDING PASS | (60,68) Black Han Sans 40 흰색 |
| 교회명 | (820,64) 우측정렬 24/700 |
| 탑승권(스텁 헤더) | (1030,68) 중앙 38 |
| PASSENGER 라벨 | (60,196) Mono 20 |
| **To 이름** | (60,318) Black Han Sans 78, 폭 420 넘으면 자동 축소(최소 20) |
| INVITED BY 라벨 | (60,410) |
| **From 이름** | (60,470) Black Han Sans 40 orange |
| 루트 | EVERYWHERE (480,266) Mono / 천장고등부 우측정렬 (838,266) Black Han Sans, 각 최대폭 135px 자동 축소, 비행기 (660,254) 32px |
| DAY 1·2 | x=480, y=360+i×84 → 태그(Mono 17 orange) / 일시 +32(25/900) / 장소 +58(17/700 grey) |
| WELCOME 도장 | 중심 (640,150), -0.12rad, 184×68 박스 |
| 하단 띠 | y 530–600 sky, 텍스트 (60,574) 23/900 — 장소는 DAY별 하단줄, 프로그램은 하단 띠 |
| 절취선 | x=860 점선(14/12), 상하 노치 r=26 (destination-out 으로 투명 처리) |
| 스텁(중심 x=1030) | FLIGHT 170 / 편명 210(Mono 32) / DATE 268 / 날짜 312(Black Han Sans 32) / QR 150px @ y334 / "와우축제 사전접수" 512 / "QR 찍고 미리 등록하기" 538 / "~10.30(금) 오후 6시" 562 |

- 별도 디자인 이미지를 받으면 `CONFIG.cardImage`에 경로 지정 → 배경으로 그리고 `CONFIG.slots`(to/from 좌표·크기·색) 위치에 이름만 얹는 구조. 이 경우 임시 탑승권 그리기는 건너뜀.
- QR은 `segno`로 생성한 PNG를 base64로 내장(`QR_SRC`). 같은 도메인 이미지라 canvas가 오염되지 않아 `toBlob` 가능.

---

## 4. 저장 로직 (가장 중요 — 실기기 이슈 반영)

```
저장하기 클릭
├─ 안드로이드 + 인앱브라우저(카톡·인스타 등)
│    → 토스트 "외부 브라우저로 이동해요…" 후 openExternal(extUrl())
│      (extUrl = 현재 주소 + ?to=이름&from=이름&v=캐시무력화값)
├─ 아이폰 (navigator.canShare({files}) 가능)
│    → navigator.share({ files:[PNG] })  → 공유창에서 '이미지 저장'
└─ 그 외 (안드로이드 크롬·삼성인터넷·PC)
     → <a download="전도축제 초대장_이름.png"> 로 직접 다운로드
```

| 이슈 | 원인 | 처리 |
|---|---|---|
| 갤럭시 카톡에서 "저장 안 됨" | 안드로이드 카톡 웹뷰가 blob 다운로드를 차단 | 외부 브라우저로 이름을 쿼리로 넘겨 이동 → 도착 시 자동 재생성 |
| 파일명 | — | 전 기기 `전도축제 초대장_이름.png`. ※ 리눅스 테스트 환경에서 한글 파일명이 `download`로 나오는 건 OS 로캘(LANG 미설정) 문제 — `LANG=C.UTF-8`로 테스트할 것 |
| 이름 일부 글자만 다른 폰트 | Google Fonts가 한글을 subset 단위로 분할 로드 → 캔버스엔 미로드 글자가 fallback | `document.fonts.load(font, 실제그릴텍스트)`로 사용 글자 전부 선로드 후 그리기 |
| 수정해도 예전 화면 | GitHub Pages `cache-control: max-age=600` | 공유 링크에 `?v=숫자` 붙여 배포 |

### 외부 브라우저 열기 (openExternal)
| 환경 | 방법 |
|---|---|
| 카카오톡 (iOS·안드로이드) | `kakaotalk://web/openExternal?url=` + encodeURIComponent(url) |
| 안드로이드 기타 인앱 | `intent://<host/path>#Intent;scheme=https;package=com.android.chrome;S.browser_fallback_url=<url>;end` |
| iOS 기타 인앱 | 주소 클립보드 복사 + "Safari에 붙여넣기" 안내 |

인앱 판별 UA 정규식: `/KAKAOTALK|Instagram|FBAN|FBAV|Line\/|NAVER|DaumApps|everytimeApp|band/i`

### 진입 시 자동 생성
주소에 `?to=&from=` 이 있으면 입력칸 채우고 `make()` 자동 실행 → 외부 브라우저로 넘어온 사용자가 저장만 다시 누르면 됨.

---

## 5. 천장지트 통합 방안

### 옵션 A — 링크/웹뷰로 연결 (반나절, 권장 1차)
- 앱 홈 배너 또는 이벤트 탭에 버튼 → `https://qjrkah-hash.github.io/capc-invite/?v=N` 외부 링크
- 장점: 코드 수정 없음, 저장 분기 그대로 검증된 상태 유지
- iframe 임베드는 **비추천**: iframe 안에서는 다운로드·share 권한 제한으로 저장이 막힐 수 있음

### 옵션 B — React 컴포넌트로 이식 (1~2일)
권장 파일 구조:
```
src/features/invite/
  inviteConfig.js      // CONFIG.event / signup / slots / colors (그대로 이동)
  qr.js                // export const QR_SRC = 'data:image/png;base64,...'
  drawTicket.js        // 순수 함수: (ctx, {to, from, config, qrImg, seed}) → void
  loadFonts.js         // document.fonts.load(font, text) 일괄 선로드
  saveImage.js         // 4장 저장 분기 (blob, toName, fromName)
  inAppBrowser.js      // IN_APP, IS_ANDROID, extUrl(), openExternal()
  InvitePage.jsx       // 입력 → 결과 화면 상태 관리 (step: 'form' | 'result')
  InvitePage.css
```

이식 포인트:
1. **폰트**: `public/index.html`에 Google Fonts `<link>` 추가 (Black Han Sans, Noto Sans KR 500;700;900, IBM Plex Mono 500;700).
2. **라우트**: `/invite` 추가. 외부 브라우저 복귀용 쿼리는 `useSearchParams()`로 읽어 `to/from` 있으면 `useEffect`에서 자동 생성.
3. **From 자동 채움**: 로그인 사용자 이름으로 From 기본값 세팅. UID는 반드시 `getMyUid()`(localStorage `cj_uid`) 사용 — `currentUser.uid` 직접 참조 금지(기존 UID 변경 이슈).
4. **비로그인 접근**: 초대받은 친구도 링크로 들어올 수 있으므로 `/invite`는 인증 가드에서 제외.
5. **외부 브라우저 URL**: `extUrl()`은 반드시 Firebase Hosting 절대주소(`https://<project>.web.app/invite?...`) 기준. SPA 직접 진입을 위해 `firebase.json` rewrites(`** → /index.html`) 확인.
6. **(선택) 발급 기록**: Firestore `inviteCards` 컬렉션에 `{ fromUid: getMyUid(), fromName, toName, createdAt: serverTimestamp() }` 저장 → 누가 몇 명 초대했는지 집계/시상 가능. 현재 catch-all 규칙(`allow write: if isLoggedIn()`)으로 쓰기 가능. `fromUid + createdAt` 정렬 쿼리를 쓰면 복합 인덱스 수동 생성 필요.
7. **Blob URL 정리**: 재생성 시 `URL.revokeObjectURL` 호출 (원본 구현 동일).

`drawTicket` 최소 골격:
```js
export async function renderTicket({ to, from, config, qrSrc }) {
  await loadFonts(collectText(to, from, config));      // 4장 폰트 이슈
  const cv = document.createElement('canvas');
  cv.width = 1200; cv.height = 600;
  const ctx = cv.getContext('2d');
  const qrImg = await loadImg(qrSrc).catch(() => null);
  drawTicket(ctx, { to, from, config, qrImg, seed: hash(to + from) });
  return new Promise((res, rej) => cv.toBlob(b => (b ? res(b) : rej()), 'image/png'));
}
```
나머지 그리기 코드는 원본 `drawPlaceholder()` + 이름 슬롯 그리기(`fitText`/`txt`)를 그대로 옮기면 됨.

---

## 6. QA 체크리스트

| # | 기기 / 환경 | 확인 항목 | 상태 |
|---|---|---|---|
| 1 | 아이폰 카톡 | 저장 → 공유창 → 사진 저장 | ✅ 사용자 확인 |
| 2 | 갤럭시 카톡 | 저장 → 크롬 자동 이동 → 이름 유지·자동 생성 → 저장 | ⚠️ 에뮬레이션만 통과, **실기기 미확인** |
| 3 | 갤럭시 크롬 / 삼성인터넷 | 직접 다운로드 → 갤러리 표시 | ⚠️ 에뮬레이션만 통과 |
| 4 | 인스타 인앱 (iOS/안드로이드) | 외부 브라우저 열기 | 미확인 |
| 5 | 탑승권 이미지 QR | 스캔 시 `m.site.naver.com/2iYtX` | ✅ 디코딩 확인 |
| 6 | 사전접수 링크 | 실제 접수 폼 열림 | **미확인** (작업 환경에서 접근 차단) |
| 7 | 긴 이름 (8자) | To 이름 자동 축소, 겹침 없음 | 확인 필요 |
| 8 | 폰트 | 모든 글자 동일 폰트로 렌더 | ✅ 수정 후 확인 |

---

## 7. 미결 사항

- 축제 공식 명칭: QR 라벨은 "와우축제", 주보는 "전도축제" → 현재 둘 다 사용. 통일 여부 결정 필요.
- 별도 디자인 초대장 이미지 사용 시 `cardImage` + `slots` 좌표 재설정.
- 행사 종료(11/1) 후 배너 내리기 / 페이지 비공개 처리.

## 8. 변경 이력

| 일시 | 내용 |
|---|---|
| 10/25 10:19 | 탑승권 컨셉으로 신규 제작 |
| 10/25 10:24 | 주보·인스타 기준 행사 정보 반영, @capc_hs |
| 10/25 10:28 | 캔버스 한글 폰트 subset 누락 수정 |
| 10/25 10:34 | GitHub Pages 배포 |
| 10/25 10:38 | 10/31 DAY 1 일정 추가 |
| 10/25 10:55 | 사전접수 QR + 설명 (이미지·결과화면) |
| 10/25 11:05 | 갤럭시 카톡 저장 이슈 수정 |
| 10/25 17:00 | 검토 반영: EVERYWHERE→천장고등부, 하단 띠(체험부스·푸드코트, 벧전 삭제), DAY 하단 장소 통일, 파일명 '전도축제 초대장' |
