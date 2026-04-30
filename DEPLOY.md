# SkyVault KE-OZ 합병 시뮬레이터 — 배포 가이드

단일 HTML 파일(`index.html`)로 동작합니다. 외부 빌드 도구 불필요.

## 1. 빠른 배포 (5분 안에 라이브)

### 옵션 A: Cloudflare Pages (권장 — 무료 + 한국 CDN 양호)

1. GitHub에 새 저장소 생성, `index.html`을 루트에 넣고 push.
2. https://dash.cloudflare.com → Workers & Pages → Create → Pages → Connect to Git
3. 저장소 선택, Build command 비움, Output directory: `/`
4. Deploy. `<프로젝트>.pages.dev` 도메인 자동 발급.
5. 커스텀 도메인(`merger.skyvault.app` 등) 연결: Pages → Custom domains.

### 옵션 B: Vercel

1. `vercel` CLI 설치 → `vercel` 명령으로 deploy.
2. 또는 GitHub 연동 후 자동 배포. 빌드 명령 비움.

### 옵션 C: Netlify Drop (가장 빠름, 1분)

1. https://app.netlify.com/drop 접속
2. `index.html`을 드래그 앤 드롭 → 즉시 라이브.
3. 단점: 임시 도메인. 정식 운영엔 옵션 A 권장.

## 2. 사전 알림 폼 백엔드 연결

`index.html` 하단 스크립트에서:

```js
const FORM_ENDPOINT = ''; // ← 여기에 endpoint 입력
```

### 옵션 A: Formspree (가장 빠름, 무료 50건/월)

1. https://formspree.io 가입
2. New Form → Form ID 발급 → endpoint URL 복사
3. `FORM_ENDPOINT = 'https://formspree.io/f/xxxxxxxx'` 로 교체

### 옵션 B: Tally.so / Google Forms

iframe 임베드 방식이라 코드 수정 필요. UX 손상 우려로 비추천.

### 옵션 C: 자체 Supabase / Firebase

본 프로젝트가 어차피 `Supabase`를 채택할 예정이므로 (기획안 §6 백엔드), 출시 임박 시점부터는 Supabase Edge Function으로 일원화 권장.

```js
// 예시: Supabase Edge Function endpoint
const FORM_ENDPOINT = 'https://<project>.supabase.co/functions/v1/signup';
```

서버에서 받은 이메일은 `pre_launch_signups` 테이블에 저장.

## 3. 분석 트래킹 연결

`index.html` 하단의 `track()` 함수에 실제 SDK 호출 주입:

### Google Analytics 4

`<head>`에 GA4 스크립트 추가:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXX');
</script>
```

`track()` 함수 안 주석 해제:

```js
window.gtag && window.gtag('event', event, props);
```

### Amplitude (기획안 권장)

```html
<script src="https://cdn.amplitude.com/libs/analytics-browser-2.x.x-min.js.gz"></script>
<script>amplitude.init('YOUR_API_KEY');</script>
```

`track()` 안:

```js
window.amplitude && window.amplitude.track(event, props);
```

추적 이벤트:
- `page_view` — 진입
- `simulator_calculated` — 입력 변경 시
- `preset_clicked` — 프리셋 버튼
- `grade_selected` — 등급 선택
- `signup_completed` / `signup_failed`
- `link_copied`

## 4. OG 이미지 교체

`<meta property="og:image" content="https://skyvault.app/og-image.png">`

권장 사이즈: 1200×630px. SkyVault 로고 + "내 아시아나 마일, 대한항공으로 얼마?" 헤드라인 + 아시아나 로고/대한항공 로고 실루엣 + 화살표.

빠른 생성: Figma/Canva 1200×630 캔버스 + Pretendard Bold 60pt.

## 5. 마케팅 캠페인 채널 (기획안 §14.1)

배포 후:
1. **마일모아** (https://www.milemoa.com) 게시판 — 기획자 자기소개 + 시뮬레이터 공유
2. **플레이윙즈** — 마일러 토론장에 공유
3. **Reddit r/koreatravel** — 영문 마일러 (한국 거주 외국인)
4. **인스타그램 스토리** — 카드뉴스 형식: "내 마일이 합병 후 얼마?"
5. **링크드인 게시물** — 페르소나 A(출장 직장인) 타깃

### 첫 24시간 KPI 목표
- 페이지 뷰 1,000+
- 시뮬레이터 사용 (이벤트) 500+
- 사전 알림 신청 100+

## 6. 데이터 갱신 절차 (분기별)

KE 차감표 변경 또는 합병 공식 발표 시:

1. `index.html` 최상단 스크립트의 `REGIONS` 배열 수정.
2. 푸터 면책 문구의 "2025년 10월 기준" 날짜 갱신.
3. Cloudflare Pages는 git push 즉시 재배포.

공식 합병 비율 발표 시:
- `RATIOS` 배열을 실제 비율 단일 시나리오로 교체
- 헤드라인을 "공식 비율 적용 시뮬레이터"로 변경
- 사전 알림 신청자 전원에게 푸시/이메일 발송

## 7. 다음 단계 (Sprint 0 진입)

이 시뮬레이터로 사전 신청자 베이스가 확보되면, 본 앱(React Native) 개발 Sprint 0 진입:

- React Native 프로젝트 초기화
- Pretendard 폰트 패키지 (`@react-native-fontfamily/pretendard`)
- 사전 신청 이메일 → 출시 시점 푸시 발송용 데이터로 활용

기획안 §11 Sprint 0 항목 참조.
